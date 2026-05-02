### CPTS / HTB Penetration Tester Path <br>
### Password Attacks - Cracking Protected Files <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Cracking Protected Files



Hunt for encrypted office docs / SSH keys, then crack offline using `*2john` extractors → JtR / hashcat.

Hunt common encrypted office extensions:

```diff
+ $ for ext in $(echo ".xls .xls* .xltx .od* .doc .doc* .pdf .pot .pot* .pp*");do echo -e "\nFile extension: " $ext; find / -name *$ext 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done
```

<br>

Hunt SSH private keys system-wide via header pattern:

```diff
+ $ grep -rnE '^\-{5}BEGIN [A-Z0-9]+ PRIVATE KEY\-{5}$' /* 2>/dev/null
```

<br>

Test if a key is passphrase-protected — `ssh-keygen` will prompt:

```diff
+ $ ssh-keygen -yf ~/.ssh/id_rsa
```

<br>

Crack a passphrase-protected SSH key:

```diff
+ $ ssh2john.py SSH.private > ssh.hash
+ $ john --wordlist=rockyou.txt ssh.hash
+ $ john ssh.hash --show
```

<br>

Crack Office docs:

```diff
+ $ office2john.py Protected.docx > protected-docx.hash
+ $ john --wordlist=rockyou.txt protected-docx.hash
+ $ john protected-docx.hash --show
```

<br>

Crack PDFs:

```diff
+ $ pdf2john.py PDF.pdf > pdf.hash
+ $ john --wordlist=rockyou.txt pdf.hash
+ $ john pdf.hash --show
```

<br>

Find every `*2john` helper on the system:

```diff
+ $ locate *2john*
```

<br>

---

<br>

### Exercise

---

### Question 1:
Download the attached ZIP archive (cracking-protected-files.zip), and crack the file within. What is the password?

```diff
+ $ locate *2john*
```

<br>

Extract hash from the encrypted .xlsx:

```diff
+ $ office2john.py /home/st8less/Desktop/htb/Confidential.xlsx > xl.hash
```

<br>

Crack with the OpenCL JtR variant:

```diff
+ $ john --format=office-opencl --devices=1,2 --fork=2 --wordlist=/mnt/SSD_DATA/SecLists/Passwords/Leaked-Databases/rockyou.txt /home/st8less/Desktop/htb/xl.hash
```

	Loaded 1 password hash (office-opencl, MS Office [SHA1/SHA512 AES OpenCL])
	Cost 1 (MS Office version) is 2013
	Cost 2 (iteration count) is 100000
	beethoven        (Confidential.xlsx)

&#x1F6A9; found **bee--edit--thoven**.
