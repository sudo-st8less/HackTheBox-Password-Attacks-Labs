### HTB Pentester Path <br>
### Password Attacks - Intro to John The Ripper <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Question 1:
Use single-crack mode to crack r0lf's password.
only use the -device & -fork flags with a gpu array + cuda/opencl
```diff
+ $ john --single /home/st8less/Desktop/folder/passwd.txt --devices=1,2 --format=sha512crypt-opencl --fork=2
```

	Using default input encoding: UTF-8
	Loaded 1 password hash (sha512crypt-opencl, crypt(3) $6$ [SHA512 OpenCL])
	Cost 1 (iteration count) is 5000 for all loaded hashes
	Node numbers 1-2 of 2 (fork)
	Device 1: NVIDIA GeForce RTX 4070
	Device 2: NVIDIA GeForce GTX 1080 Ti
	Note: Passwords longer than 7 [worst case UTF-8] to 23 [ASCII] rejected
	1: LWS=256 GWS=11776 (46 blocks)
	Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
	Almost done: Processing the remaining buffered candidate passwords, if any.
	1: Warning: Only 9032 candidates buffered for the current salt, minimum 11776 needed for performance.
	NAITSABES        (r0lf)
	1 1g 0:00:00:00 DONE (2025-10-10 05:19) 20.00g/s 180640p/s 180640c/s 180640C/s Dev#1:56°C r0lf..Srolf19
	00
	Waiting for 1 child to terminate
	2: LWS=128 GWS=14336 (112 blocks)
	2: Warning: Only 9072 candidates buffered for the current salt, minimum 14336 needed for performance.
	2 0g 0:00:00:00 DONE (2025-10-10 05:19) 0g/s 120960p/s 120960c/s 120960C/s Dev#2:28°C R0lf..Srolf1901
	Use the "--show" option to display all of the cracked passwords reliably
	Session completed

🚩 found **NAITS--edit--ABES**.

---

### Question 2:
Use wordlist-mode with rockyou.txt to crack the RIPEMD-128 password.
```diff
+ $ john --format=ripemd-128 --wordlist=/mnt/SSD_DATA/SecLists/Passwords/Leaked-Databases/rockyou.txt /home/st8less/Desktop/folder/ripe.txt
```

	Using default input encoding: UTF-8
	Loaded 1 password hash (ripemd-128, RIPEMD 128 [32/64])
	Warning: no OpenMP support for this hash type, consider --fork=12
	Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
	50cent           (?)
	1g 0:00:00:00 DONE (2025-10-10 05:40) 100.0g/s 32000p/s 32000c/s 32000C/s angelo..101010
	Use the "--show" option to display all of the cracked passwords reliably
	Session completed

🚩 found **50--edit--cent**.
