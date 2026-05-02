### CPTS / HTB Penetration Tester Path <br>
### Password Attacks - Intro to Hashcat <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Introduction to Hashcat



Hashcat — fastest open-source GPU password cracker, supports hundreds of hash types and multiple attack modes.

General syntax:

```diff
+ $ hashcat -a 0 -m 0 <hashes> [wordlist, rule, mask, ...]
```

<br>

Flags:

| Flag | Meaning |
|---|---|
| `-a` | attack mode (0=dict, 3=mask, 1=combinator, etc.) |
| `-m` | hash type ID (0=MD5, 100=SHA1, 1000=NTLM, 1800=sha512crypt, 22100=BitLocker) |

List all hash mode IDs:

```diff
+ $ hashcat --help
```

<br>

Identify a hash with `hashID`:

```diff
+ $ hashid -m '$1$FNr44XZC$wQxY6HHLrgrGX0e1195k.1'
```

<br>

Dictionary attack (`-a 0`) against MD5:

```diff
+ $ hashcat -a 0 -m 0 e3e3ec5831ad5e7288241960e5d4fdb8 /usr/share/wordlists/rockyou.txt
```

<br>

Dictionary + rules — common rule sets in `/usr/share/hashcat/rules/`:

```diff
+ $ ls -l /usr/share/hashcat/rules
```

<br>

Apply a rule file:

```diff
+ $ hashcat -a 0 -m 0 <hash> rockyou.txt -r /usr/share/hashcat/rules/best64.rule
```

<br>

Mask attack (`-a 3`) — explicit charset by position:

| Symbol | Charset |
|---|---|
| `?l` | abcdefghijklmnopqrstuvwxyz |
| `?u` | ABCDEFGHIJKLMNOPQRSTUVWXYZ |
| `?d` | 0123456789 |
| `?h` | 0-9a-f |
| `?H` | 0-9A-F |
| `?s` | special chars |
| `?a` | ?l?u?d?s |
| `?b` | 0x00-0xff |

Custom charsets via `-1`/`-2`/`-3`/`-4`, then reference as `?1`/`?2`/`?3`/`?4`.

Example mask — uppercase + 4 lowercase + digit + symbol:

```diff
+ $ hashcat -a 3 -m 0 1e293d6912d074c0fd15844d803400dd '?u?l?l?l?l?d?s'
```

<br>

---

<br>

### Exercise

---

### Question 1:
Use a dictionary attack to crack the first password hash. (Hash: e3e3ec5831ad5e7288241960e5d4fdb8)

```diff
+ $ hashcat -m 0 -a 0 e3e3ec5831ad5e7288241960e5d4fdb8 /mnt/SSD_DATA/SecLists/Passwords/Leaked-Databases/rockyou.txt -d 1,2
```

	Dictionary cache built:
	* Filename..: rockyou.txt
	* Passwords.: 14344391
	e3e3ec5831ad5e7288241960e5d4fdb8:crazy!

&#x1F6A9; found **cra--edit--zy!**.

---

### Question 2:
Use a dictionary attack with rules to crack the second password hash. (Hash: 1b0556a75770563578569ae21392630c)

```diff
+ $ hashcat -m 0 -a 0 1b0556a75770563578569ae21392630c /mnt/SSD_DATA/SecLists/Passwords/Leaked-Databases/rockyou.txt -r /usr/share/doc/hashcat-doc/rules/best64.rule -d 1,2
```

	Status...........: Cracked
	1b0556a75770563578569ae21392630c:c0wb0ys1

&#x1F6A9; found **c0wb--edit--0ys1**.

---

### Question 3:
Use a mask attack to crack the third password hash. (Hash: 1e293d6912d074c0fd15844d803400dd)

```diff
+ $ hashcat -a 3 -m 0 1e293d6912d074c0fd15844d803400dd '?u?l?l?l?l?d?s' -d 1,2
```

	Status...........: Cracked
	Guess.Mask.......: ?u?l?l?l?l?d?s [7]
	1e293d6912d074c0fd15844d803400dd:Mouse5!

&#x1F6A9; found **Mou--edit--se5!**.
