### CPTS / HTB Penetration Tester Path <br>
### Password Attacks: Linux Authentication Process <br>
<mark>hook it up with a follow if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Question 1:
Download the attached ZIP file (linux-authentication-process.zip), and use single crack mode to find martin's password. What is it?

Download and extract the zip. Now use unshadow to merge the relevant data from both into one file:
```diff
+ $ unshadow /home/st8less/Desktop/htb/passwd /home/st8less/Desktop/htb/shadow > /home/st8less/Desktop/htb/unshadowed.hashes
```

Now I went in and manually cleaned up the file to only contain martin and sarah's data, deleting all the other linux-based sys accounts. 

![](/img/1.png)

<br>

Since the questions are asking us to use different cracking methods, I'll separate them further into files of their own and save them as martin.hash and sarah.hash, so we aren't wasting time and our electric bill.

<br>

![](/img/2.png)

<br>

Now crack in single mode with JTR:
```diff
+ $ john --single martin.hash
```

	Warning: detected hash type "sha512crypt", but the string is also recognized as "sha512crypt-opencl"
	Use the "--format=sha512crypt-opencl" option to force loading these as that type instead
	Using default input encoding: UTF-8
	Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 256/256 AVX2 4x])
	Cost 1 (iteration count) is 5000 for all loaded hashes
	Will run 12 OpenMP threads
	Note: Passwords longer than 26 [worst case UTF-8] to 79 [ASCII] rejected
	Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
	Martin1          (martin)
	1g 0:00:00:00 DONE (2025-10-17 15:59) 50.00g/s 2400p/s 2400c/s 2400C/s martin..martinMendes3
	Use the "--show" option to display all of the cracked passwords reliably
	Session completed

🚩 found **Mar--edit--tin1**.

---

### Question 2:
Use a wordlist attack to find sarah's password. What is it?

Now we're just going to run a dictionary attack against sarah's hash combo. Keep in mind, unless you are using cuda or opencl for gpu parallelization, you won't need the format, device or fork flags:
```diff
+ $ john --format=sha512crypt-opencl --devices=1,2 --fork=2 --wordlist=/mnt/SSD_DATA/SecLists/rockyou.txt /home/st8less/Desktop/htb/sarah.hash
```

	Using default input encoding: UTF-8
	Loaded 1 password hash (sha512crypt-opencl, crypt(3) $6$ [SHA512 OpenCL])
	Cost 1 (iteration count) is 5000 for all loaded hashes
	Node numbers 1-2 of 2 (fork)
	Device 1: NVIDIA GeForce RTX 4070
	Device 2: NVIDIA GeForce GTX 1080 Ti
	Note: Passwords longer than 7 [worst case UTF-8] to 23 [ASCII] rejected
	1: LWS=32 GWS=188416 (5888 blocks)
	Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
	mariposa         (sarah)
	1 1g 0:00:00:00 DONE (2025-10-17 16:11) 2.083g/s 392533p/s 392533c/s 392533C/s Dev#1:68°C 123456..1473
	57
	Waiting for 1 child to terminate
	2: LWS=32 GWS=229376 (7168 blocks)
	2 0g 0:00:00:33 DONE (2025-10-17 16:12) 0g/s 214339p/s 214339c/s 214339C/s Dev#2:40°C 0108058828.a6_12
	3
	Use the "--show" option to display all of the cracked passwords reliably
	Session completed

🚩 found **mariposa**.
