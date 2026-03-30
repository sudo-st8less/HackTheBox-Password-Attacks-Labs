### CPTS / HTB Penetration Tester Path <br>
### Password Attacks: Cracking Protected Files <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
Only use -device, -fork flags if you have a gpu array + opencl or cuda
<br>

---

### Question 1:
Download the attached ZIP archive (cracking-protected-files.zip), and crack the file within. What is the password?

Search for your office john script:
```diff
+ $ locate *2john*
```

	/home/st8less/userbin/john/doc/README.7z2john.md
	/home/st8less/userbin/john/doc/pcap2john.readme
	/home/st8less/userbin/john/run/1password2john.py
	/home/st8less/userbin/john/run/7z2john.pl
	/home/st8less/userbin/john/run/DPAPImk2john.py
	/home/st8less/userbin/john/run/adxcsouf2john.py
	/home/st8less/userbin/john/run/aem2john.py
	/home/st8less/userbin/john/run/aix2john.pl
    
    SNIP

	/home/st8less/userbin/john/src/rar2john.c
	/home/st8less/userbin/john/src/rar2john.h
	/home/st8less/userbin/john/src/rar2john.o
	/home/st8less/userbin/john/src/uaf2john.c
	/home/st8less/userbin/john/src/uaf2john.o
	/home/st8less/userbin/john/src/vncpcap2john.c
	/home/st8less/userbin/john/src/vncpcap2john.o
	/home/st8less/userbin/john/src/wpapcap2john.c
	/home/st8less/userbin/john/src/wpapcap2john.h
	/home/st8less/userbin/john/src/wpapcap2john.o
	/home/st8less/userbin/john/src/zip2john.c
	/home/st8less/userbin/john/src/zip2john.o

Extract hash from encrypted .xlsx:
```diff
+ $ office2john.py /home/st8less/Desktop/htb/Confidential.xlsx > xl.hash
```

Peanut Butter and Crack:
```diff
+ $ john --format=office-opencl --devices=1,2 --fork=2 --wordlist=/mnt/SSD_DATA/SecLists/Passwords/Leaked-Databases/rockyou.txt /home/st8less/Desktop/htb/xl.hash
```

	Using default input encoding: UTF-8
	Loaded 1 password hash (office-opencl, MS Office [SHA1/SHA512 AES OpenCL])
	Cost 1 (MS Office version) is 2013 for all loaded hashes
	Cost 2 (iteration count) is 100000 for all loaded hashes
	Node numbers 1-2 of 2 (fork)
	Device 2: NVIDIA GeForce GTX 1080 Ti
	Device 1: NVIDIA GeForce RTX 4070
	Note: Passwords longer than 41 [worst case UTF-8] to 47 [ASCII] rejected
	1: LWS=64 GWS=94208 (1472 blocks)
	Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
	2: LWS=256 GWS=14336 (56 blocks)
	beethoven        (Confidential.xlsx)
	1 1g 0:00:00:04 DONE (2025-10-12 02:56) 0.2445g/s 23033p/s 23033c/s 23033C/s Dev#1:78°C 123456..beckyg
	Waiting for 1 child to terminate
	2 0g 0:00:00:04 DONE (2025-10-12 02:56) 0g/s 14124p/s 14124c/s 14124C/s Dev#2:33°C 022179..260328
	Use the "--show" option to display all of the cracked passwords reliably
	Session completed

🚩 found **beethoven**.
