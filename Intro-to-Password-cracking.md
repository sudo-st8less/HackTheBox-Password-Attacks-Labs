### CPTS / HTB Penetration Tester Path <br>
### Password Attacks - Intro to Password Cracking <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Introduction to Password Cracking



Stored passwords are typically `hashed` — one-way mathematical transform (MD5, SHA-256, etc.). Cracking = recovering plaintext from a hash.

Generate hashes:

```diff
+ $ echo -n Soccer06! | md5sum
```

	40291c1d19ee11a7df8495c4cccefdfa  -

<br>

```diff
+ $ echo -n Soccer06! | sha256sum
```

	a025dc6fabb09c2b8bfe23b5944635f9b68433ebd9a1a09453dd4fee00766d93  -

<br>

Three common cracking techniques:

| Technique | Description |
|---|---|
| `Rainbow tables` | Pre-computed maps of input → hash; instant lookup |
| `Dictionary attack` | Try every word in a wordlist (statistically likely passwords) |
| `Brute-force` | Try every possible char combination — 100% effective, but very slow |

Salt — random bytes prepended/appended to a password before hashing. Defeats rainbow tables since pre-computed hashes don't account for the salt. Salts are not secret, but force per-password computation.

```diff
+ $ echo -n Th1sIsTh3S@lt_Soccer06! | md5sum
```

	90a10ba83c04e7996bc53373170b5474  -

<br>

Brute-force speed depends on hash algo + hardware. Hashcat on a typical laptop: ~5M/sec MD5 vs ~10K/sec DCC2.

Common wordlists: [rockyou.txt](https://github.com/brannondorsey/naive-hashcat/releases/download/data/rockyou.txt), [SecLists](https://github.com/danielmiessler/SecLists).

Peek at rockyou:

```diff
+ $ head --lines=20 /usr/share/wordlists/rockyou.txt
```

	123456
	12345
	123456789
	password
	iloveyou
	princess
	1234567
	rockyou
	12345678
	abc123
	...

<br>

---

<br>

### Exercise

---

### Question 1:
What is the SHA1 hash for `Academy#2025`?

```diff
+ $ echo -n 'Academy#2025' | sha1sum
```

&#x1F6A9; found **750fe4b402dc--edit--9f91cedf09b652543cd85406be8c**.
