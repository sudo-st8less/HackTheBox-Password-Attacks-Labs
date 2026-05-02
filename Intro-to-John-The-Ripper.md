### CPTS / HTB Penetration Tester Path <br>
### Password Attacks - Intro to John The Ripper <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Introduction to John The Ripper



`JtR` (john) — open-source pen-test password cracker. Use the `jumbo` build for full hash format and platform support.

Three primary cracking modes:

| Mode | Use case |
|---|---|
| `--single` | Generate candidates from username/GECOS/home dir info — best for Linux creds |
| `--wordlist=<file>` | Dictionary attack against supplied hash file |
| `--incremental` | Markov-chain brute-force, prioritized by training data |

Single crack mode against `passwd`:

```diff
+ $ john --single passwd
```

<br>

Wordlist mode:

```diff
+ $ john --wordlist=<wordlist_file> <hash_file>
```

<br>

Incremental (slow, exhaustive):

```diff
+ $ john --incremental <hash_file>
```

<br>

Inspect built-in incremental modes:

```diff
+ $ grep '# Incremental modes' -A 100 /etc/john/john.conf
```

<br>

Identify unknown hash format with `hashID`:

```diff
+ $ hashid -j 193069ceb0461e1d40d216e32c79c704
```

<br>

JtR includes `*2john` helper scripts to extract hashes from common protected file types:

```diff
+ $ <tool> <file_to_crack> > file.hash
```

<br>

Locate every helper on the box:

```diff
+ $ locate *2john*
```

<br>

Common helpers: `pdf2john`, `ssh2john`, `mscash2john`, `keychain2john`, `rar2john`, `pfx2john`, `truecrypt_volume2john`, `keepass2john`, `vncpcap2john`, `putty2john`, `zip2john`, `hccap2john`, `office2john`, `wpa2john`.

Specify hash format with `--format=<name>` (e.g. `raw-md5`, `nt`, `mscash2`, `sha512crypt`, `ripemd-128`).

<br>

---

<br>

### Exercise

---

### Question 1:
Use single-crack mode to crack r0lf's password.

```diff
+ $ john --single /home/st8less/Desktop/folder/passwd.txt --devices=1,2 --format=sha512crypt-opencl --fork=2
```

	Loaded 1 password hash (sha512crypt-opencl, crypt(3) $6$ [SHA512 OpenCL])
	Cost 1 (iteration count) is 5000 for all loaded hashes
	Node numbers 1-2 of 2 (fork)
	Device 1: NVIDIA GeForce RTX 4070
	Device 2: NVIDIA GeForce GTX 1080 Ti
	NAITSABES        (r0lf)

#### Single mode generated candidates from r0lf's GECOS info (`Rolf Sebastian`).

&#x1F6A9; found **NAITS--edit--ABES**.

---

### Question 2:
Use wordlist-mode with rockyou.txt to crack the RIPEMD-128 password.

```diff
+ $ john --format=ripemd-128 --wordlist=/mnt/SSD_DATA/SecLists/Passwords/Leaked-Databases/rockyou.txt /home/st8less/Desktop/folder/ripe.txt
```

	Loaded 1 password hash (ripemd-128, RIPEMD 128 [32/64])
	50cent           (?)
	1g 0:00:00:00 DONE (2025-10-10 05:40)

&#x1F6A9; found **50cent**.
