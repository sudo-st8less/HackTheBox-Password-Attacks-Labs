### CPTS / HTB Penetration Tester Path <br>
### Password Attacks: Cracking Protected Archives <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Question 1:
Run the above target then navigate to http://ip:port/download, then extract the downloaded file. Inside, you will find a password-protected VHD file. Crack the password for the VHD and submit the recovered password as your answer.

Extract hashes to a backup file:
```diff
+ $ bitlocker2john -i /home/htb-ac-830862/Desktop/Private.vhd > /home/htb-ac-830862/Desktop/backup.hashs
```

	Signature found at 0x10003
	Version: 8 
	Invalid version, looking for a signature with valid version...
	
	Signature found at 0x2200000
	Version: 2 (Windows 7 or later)
	
	VMK entry found at 0x22000bb
	
	VMK encrypted with User Password found at 22000dc
	VMK encrypted with AES-CCM
	
	VMK entry found at 0x220019b
	
	VMK encrypted with Recovery Password found at 0x22001bc
	Searching AES-CCM from 0x22001d8
	Trying offset 0x220026b....
	VMK encrypted with AES-CCM!!
	
	Signature found at 0x2956000
	Version: 2 (Windows 7 or later)
	
	VMK entry found at 0x29560bb
	
	VMK entry found at 0x295619b
	
	Signature found at 0x30ab000
	Version: 2 (Windows 7 or later)
	
	VMK entry found at 0x30ab0bb
	
	VMK entry found at 0x30ab19b

Put the user hash string in its own bkup.hash file.

Now crack that file with hashcat:
```diff
+ $ hashcat -a 0 -m 22100 -d 1,2 '$bitlocker$0$16$b3c105c7ab7faaf544e84d712810da65$1048576$12$b020fe18bbb1db0103000000$60$e9c6b548788aeff190e517b0d85ada5daad7a0a3f40c4467307011ac17f79f8c99768419903025fd7072ee78b15a729afcf54b8c2e3af05bb18d4ba0' /mnt/SSD_DATA/SecLists/Passwords/Leaked-Databases/rockyou.txt
```

	hashcat (v6.2.6) starting
	
	CUDA API (CUDA 13.0)
	====================
	* Device #1: NVIDIA GeForce RTX 4070, 10231/11850 MB, 46MCU
	* Device #2: NVIDIA GeForce GTX 1080 Ti, 11022/11165 MB, 28MCU
	
	OpenCL API (OpenCL 3.0 CUDA 13.0.84) - Platform #1 [NVIDIA Corporation]
	=======================================================================
	* Device #3: NVIDIA GeForce RTX 4070, skipped
	* Device #4: NVIDIA GeForce GTX 1080 Ti, skipped
	
	Minimum password length supported by kernel: 4
	Maximum password length supported by kernel: 256
	
	Hashes: 1 digests; 1 unique digests, 1 unique salts
	Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
	Rules: 1
	
	Optimizers applied:
	* Single-Hash
	* Single-Salt
	* Slow-Hash-SIMD-LOOP
	
	Watchdog: Temperature abort trigger set to 90c
	
	Host memory required for this attack: 562 MB
	
	Dictionary cache built:
	* Filename..: /mnt/SSD_DATA/SecLists/Passwords/Leaked-Databases/rockyou.txt
	* Passwords.: 14344391
	* Bytes.....: 139921497
	* Keyspace..: 14344384
	* Runtime...: 1 sec
	
	$bitlocker$0$16$b3c105c7ab7faaf544e84d712810da65$1048576$12$b020fe18bbb1db0103000000$60$e9c6b548788aeff190e517b0d85a
	da5daad7a0a3f40c4467307011ac17f79f8c99768419903025fd7072ee78b15a729afcf54b8c2e3af05bb18d4ba0:francisco
	
	Session..........: hashcat
	Status...........: Cracked
	Hash.Mode........: 22100 (BitLocker)
	Hash.Target......: $bitlocker$0$16$b3c105c7ab7faaf544e84d712810da65$10...8d4ba0
	Time.Started.....: Mon Oct 13 01:11:43 2025 (3 secs)
	Time.Estimated...: Mon Oct 13 01:11:46 2025 (0 secs)
	Kernel.Feature...: Pure Kernel
	Guess.Base.......: File (/mnt/SSD_DATA/SecLists/Passwords/Leaked-Databases/rockyou.txt)
	Guess.Queue......: 1/1 (100.00%)
	Speed.#1.........:      357 H/s (11.15ms) @ Accel:32 Loops:4096 Thr:32 Vec:1
	Speed.#2.........:      236 H/s (16.92ms) @ Accel:4 Loops:4096 Thr:256 Vec:1
	Speed.#*.........:      592 H/s
	Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
	Progress.........: 1024/14344384 (0.01%)
	Rejected.........: 0/1024 (0.00%)
	Restore.Point....: 0/14344384 (0.00%)
	Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:1044480-1048576
	Restore.Sub.#2...: Salt:0 Amplifier:0-1 Iteration:692224-696320
	Candidate.Engine.: Device Generator
	Candidates.#1....: 123456 -> bethany
	Candidates.#2....: kucing -> lovers1
	Hardware.Mon.#1..: Temp: 50c Fan:  0% Util:100% Core:2835MHz Mem:10251MHz Bus:16
	Hardware.Mon.#2..: Temp: 27c Fan:  0% Util:100% Core:1987MHz Mem:5005MHz Bus:4
	
	Started: Mon Oct 13 01:11:33 2025
	Stopped: Mon Oct 13 01:11:47 2025

🚩 found **fran--edit--cisco**.

---

### Question 2:
Mount the BitLocker-encrypted VHD and enter the contents of flag.txt as your answer.

Make two directories labeled specifically like this, so the dislocker tool can use them:
```diff
+ $ sudo mkdir -p /media/bitlocker /media/bitlockermount
```

Setup the vhd as a loop device, so your OS can write and read blocks to it like it would an actual hard drive:
```diff
+ $ sudo losetup -fP Private.vhd
```
```diff
+ $ losetup -a
```

	/dev/loop0: []: (/home/htb-ac-830862/Desktop/Private.vhd)

We need to find and use partitions in this vhd, so download a tool called kpartx, and use it on the loop device:
```diff
+ $ sudo apt install kpartx
```
```diff
+ $ sudo kpartx -av /dev/loop0
```

	add map loop0p1 (253:0): 0 124928 linear 7:0 128

Verify part exists:
```diff
+ $ ls /dev/mapper
```

	control  loop0p1

Sweet, now we can use dislocker and the cracked PW to decrypt:
```diff
+ $ sudo dislocker -V /dev/loop0p1 -u"francisco" -- /media/bitlocker
```

Wham Bam Thank You Mam. Now just mount and traverse for the shiny thing:
```diff
+ $ sudo mount -o loop /media/bitlocker/dislocker-file /media/bitlockermount
```
```diff
+ $ lsblk
```

	NAME      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
	loop0       7:0    0   64M  0 loop 
	├─loop0p1 253:0    0   61M  0 part 
	└─loop0p1 259:2    0   61M  0 part 
	loop1       7:1    0   64M  0 loop 
	└─loop1p1 259:1    0   61M  0 part 
	loop2       7:2    0   64M  0 loop 
	└─loop2p1 259:0    0   61M  0 part 
	loop3       7:3    0   61M  0 loop /media/bitlockermount
	vda       254:0    0   40G  0 disk 
	├─vda1    254:1    0   39G  0 part /
	├─vda2    254:2    0    1K  0 part 
	└─vda5    254:5    0  975M  0 part [SWAP]
```diff
+ $ cd /media/bitlockermount
+ $ ls
```

	 flag.txt  'System Volume Information'
```diff
+ $ cat flag.txt
```

	43d95aeed3114a53ac66f01265f9b7af

🚩 found **43d95aeed3114a53ac--edit--66f01265f9b7af**.
