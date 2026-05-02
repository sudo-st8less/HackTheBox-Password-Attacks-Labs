### CPTS / HTB Penetration Tester Path <br>
### Password Attacks - Linux Authentication Process <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Linux Authentication Process



`PAM` (Pluggable Authentication Modules) — `pam_unix.so` reads/writes `/etc/passwd` + `/etc/shadow`. PAM also wraps LDAP / Kerberos / mount auth.

`/etc/passwd` (world-readable) — 7 colon-separated fields:

| Field | Example |
|---|---|
| Username | `htb-student` |
| Password | `x` (placeholder; hash in shadow) |
| UID | `1000` |
| GID | `1000` |
| GECOS | `,,,` |
| Home | `/home/htb-student` |
| Shell | `/bin/bash` |

If `/etc/passwd` is writable (misconfig), can null the root password field — login w/o prompt.

`/etc/shadow` (root-readable) — 9 fields, hash format: `$<id>$<salt>$<hashed>`

| ID | Algorithm |
|---|---|
| `1` | MD5 |
| `2a` | Blowfish |
| `5` | SHA-256 |
| `6` | SHA-512 |
| `sha1` | SHA1crypt |
| `y` | Yescrypt (modern Debian default) |
| `gy` | Gost-yescrypt |
| `7` | Scrypt |

`!` or `*` in password field = no Unix login (other auth still possible). Empty = no password.

`/etc/security/opasswd` — PAM stores previous password hashes here to enforce no-reuse policy. Useful for spotting weak history (e.g. MD5 entries).

#### Cracking flow — `unshadow` + JtR / hashcat

```diff
+ $ sudo cp /etc/passwd /tmp/passwd.bak
+ $ sudo cp /etc/shadow /tmp/shadow.bak
+ $ unshadow /tmp/passwd.bak /tmp/shadow.bak > /tmp/unshadowed.hashes
```

<br>

Hashcat sha512crypt:

```diff
+ $ hashcat -m 1800 -a 0 /tmp/unshadowed.hashes rockyou.txt -o /tmp/unshadowed.cracked
```

<br>

JtR single-mode is purpose-built for this (uses GECOS/username for candidates).

<br>

---

<br>

### Exercise

---

### Question 1:
Download the attached ZIP file (linux-authentication-process.zip), and use single crack mode to find martin's password. What is it?

Download + extract the zip. Combine passwd + shadow:

```diff
+ $ unshadow /home/st8less/Desktop/htb/passwd /home/st8less/Desktop/htb/shadow > /home/st8less/Desktop/htb/unshadowed.hashes
```

<br>

Trim the file to just martin + sarah (skip system accounts), split each into its own `.hash`. Then single-mode against martin:

```diff
+ $ john --single martin.hash
```

	Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 256/256 AVX2 4x])
	Martin1          (martin)

&#x1F6A9; found **Mar--edit--tin1**.

---

### Question 2:
Use a wordlist attack to find sarah's password. What is it?

```diff
+ $ john --format=sha512crypt-opencl --devices=1,2 --fork=2 --wordlist=/mnt/SSD_DATA/SecLists/rockyou.txt /home/st8less/Desktop/htb/sarah.hash
```

	Loaded 1 password hash (sha512crypt-opencl, crypt(3) $6$ [SHA512 OpenCL])
	mariposa         (sarah)

&#x1F6A9; found **mar--edit--iposa**.
