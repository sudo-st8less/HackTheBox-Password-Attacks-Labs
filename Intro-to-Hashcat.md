### HTB Pentester Path <br>
### Password Attacks - Intro to Hashcat <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Question 1:
Use a dictionary attack to crack the first password hash. (Hash: e3e3ec5831ad5e7288241960e5d4fdb8)
```diff
+ $ hashcat -m 0 -a 0 e3e3ec5831ad5e7288241960e5d4fdb8 /mnt/SSD_DATA/SecLists/Passwords/Leaked-Databases/rockyou.txt -d 1,2
```

	hashcat (v6.2.6) starting
	
	CUDA API (CUDA 13.0)
	====================
	* Device #1: NVIDIA GeForce RTX 4070, 10105/11850 MB, 46MCU
	* Device #2: NVIDIA GeForce GTX 1080 Ti, 11022/11165 MB, 28MCU
	
	OpenCL API (OpenCL 3.0 CUDA 13.0.84) - Platform #1 [NVIDIA Corporation]
	=======================================================================
	* Device #3: NVIDIA GeForce RTX 4070, skipped
	* Device #4: NVIDIA GeForce GTX 1080 Ti, skipped
	
	Watchdog: Temperature abort trigger set to 90c
	
	Host memory required for this attack: 1299 MB
	
	Dictionary cache built:
	* Filename..: /mnt/SSD_DATA/SecLists/Passwords/Leaked-Databases/rockyou.txt
	* Passwords.: 14344391
	* Bytes.....: 139921497
	* Keyspace..: 14344384
	* Runtime...: 1 sec
	
	e3e3ec5831ad5e7288241960e5d4fdb8:crazy!

🚩 found **cra--edit--zy!**.

---

### Question 2:
Use a dictionary attack with rules to crack the second password hash. (Hash: 1b0556a75770563578569ae21392630c)
```diff
+ $ hashcat -m 0 -a 0 1b0556a75770563578569ae21392630c /mnt/SSD_DATA/SecLists/Passwords/Leaked-Databases/rockyou.txt -r /usr/share/doc/hashcat-doc/rules/best64.rule -d 1,2
```

	hashcat (v6.2.6) starting
	
	CUDA API (CUDA 13.0)
	====================
	* Device #1: NVIDIA GeForce RTX 4070, 10254/11850 MB, 46MCU
	* Device #2: NVIDIA GeForce GTX 1080 Ti, 11022/11165 MB, 28MCU
	
	OpenCL API (OpenCL 3.0 CUDA 13.0.84) - Platform #1 [NVIDIA Corporation]
	=======================================================================
	* Device #3: NVIDIA GeForce RTX 4070, skipped
	* Device #4: NVIDIA GeForce GTX 1080 Ti, skipped
	
	Minimum password length supported by kernel: 0
	Maximum password length supported by kernel: 256
	
	Hashes: 1 digests; 1 unique digests, 1 unique salts
	Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
	Rules: 77
	
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
	* Keyspace..: 1104517568
	* Runtime...: 0 secs
	
	1b0556a75770563578569ae21392630c:c0wb0ys1
	
	Session..........: hashcat
	Status...........: Cracked
	Hash.Mode........: 0 (MD5)
	Hash.Target......: 1b0556a75770563578569ae21392630c
	Time.Started.....: Sat Oct 11 02:59:03 2025 (0 secs)
	Time.Estimated...: Sat Oct 11 02:59:03 2025 (0 secs)
	Kernel.Feature...: Pure Kernel
	Guess.Base.......: File (/mnt/SSD_DATA/SecLists/Passwords/Leaked-Databases/rockyou.txt)
	Guess.Mod........: Rules (/usr/share/doc/hashcat-doc/rules/best64.rule)
	Guess.Queue......: 1/1 (100.00%)
	Speed.#1.........:        0 H/s (0.00ms) @ Accel:128 Loops:77 Thr:64 Vec:1
	Speed.#2.........:  1049.4 MH/s (9.34ms) @ Accel:128 Loops:77 Thr:64 Vec:1
	Speed.#*.........:  1049.4 MH/s
	Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
	Progress.........: 17661952/1104517568 (1.60%)
	Rejected.........: 0/17661952 (0.00%)
	Restore.Point....: 0/14344384 (0.00%)
	Restore.Sub.#1...: Salt:0 Amplifier:0-0 Iteration:0-77
	Restore.Sub.#2...: Salt:0 Amplifier:0-77 Iteration:0-77
	Candidate.Engine.: Device Generator
	Candidates.#1....: [Copying]
	Candidates.#2....: 123456 -> 161616
	Hardware.Mon.#1..: Temp: 54c Fan:  0% Util:  0% Core:2520MHz Mem:10251MHz Bus:16
	Hardware.Mon.#2..: Temp: 22c Fan:  0% Util: 71% Core:1721MHz Mem:5005MHz Bus:4
	
	Started: Sat Oct 11 02:59:00 2025
	Stopped: Sat Oct 11 02:59:03 2025

🚩 found **c0wb--edit--0ys1**.

---

### Question 3:
Use a mask attack to crack the third password hash. (Hash: 1e293d6912d074c0fd15844d803400dd)
```diff
+ $ hashcat -a 3 -m 0 1e293d6912d074c0fd15844d803400dd '?u?l?l?l?l?d?s' -d 1,2
```

	hashcat (v6.2.6) starting
	
	CUDA API (CUDA 13.0)
	====================
	* Device #1: NVIDIA GeForce RTX 4070, 10221/11850 MB, 46MCU
	* Device #2: NVIDIA GeForce GTX 1080 Ti, 11022/11165 MB, 28MCU
	
	OpenCL API (OpenCL 3.0 CUDA 13.0.84) - Platform #1 [NVIDIA Corporation]
	=======================================================================
	* Device #3: NVIDIA GeForce RTX 4070, skipped
	* Device #4: NVIDIA GeForce GTX 1080 Ti, skipped
	
	Minimum password length supported by kernel: 0
	Maximum password length supported by kernel: 256
	
	Hashes: 1 digests; 1 unique digests, 1 unique salts
	Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
	
	Optimizers applied:
	* Zero-Byte
	* Early-Skip
	* Not-Salted
	* Not-Iterated
	* Single-Hash
	* Single-Salt
	* Brute-Force
	* Raw-Hash
	
	ATTENTION! Pure (unoptimized) backend kernels selected.
	Pure kernels can crack longer passwords, but drastically reduce performance.
	If you want to switch to optimized kernels, append -O to your commandline.
	See the above message to find out about the exact limits.
	
	Watchdog: Temperature abort trigger set to 90c
	
	Host memory required for this attack: 2939 MB
	
	The wordlist or mask that you are using is too small.
	This means that hashcat cannot use the full parallel power of your device(s).
	Unless you supply more work, your cracking speed will drop.
	For tips on supplying more work, see: https://hashcat.net/faq/morework
	
	Approaching final keyspace - workload adjusted.
	
	1e293d6912d074c0fd15844d803400dd:Mouse5!
	
	Session..........: hashcat
	Status...........: Cracked
	Hash.Mode........: 0 (MD5)
	Hash.Target......: 1e293d6912d074c0fd15844d803400dd
	Time.Started.....: Sat Oct 11 03:03:15 2025 (0 secs)
	Time.Estimated...: Sat Oct 11 03:03:15 2025 (0 secs)
	Kernel.Feature...: Pure Kernel
	Guess.Mask.......: ?u?l?l?l?l?d?s [7]
	Guess.Queue......: 1/1 (100.00%)
	Speed.#1.........: 19935.0 MH/s (0.75ms) @ Accel:32 Loops:256 Thr:512 Vec:1
	Speed.#2.........:  1326.5 MH/s (0.15ms) @ Accel:1024 Loops:128 Thr:32 Vec:1
	Speed.#*.........: 21261.5 MH/s
	Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
	Progress.........: 454364160/3920854080 (11.59%)
	Rejected.........: 0/454364160 (0.00%)
	Restore.Point....: 0/223080 (0.00%)
	Restore.Sub.#1...: Salt:0 Amplifier:5888-6144 Iteration:0-256
	Restore.Sub.#2...: Salt:0 Amplifier:10752-10880 Iteration:0-128
	Candidate.Engine.: Device Generator
	Candidates.#1....: Ggner1. -> Dikug0,
	Candidates.#2....: Nzcfb2, -> Keplc8#
	Hardware.Mon.#1..: Temp: 60c Fan:  0% Util:  1% Core:2790MHz Mem:10251MHz Bus:16
	Hardware.Mon.#2..: Temp: 27c Fan:  0% Util: 93% Core:1885MHz Mem:5005MHz Bus:4
	
	Started: Sat Oct 11 03:03:08 2025
	Stopped: Sat Oct 11 03:03:17 2025

🚩 found **Mou--edit--se5!**.
