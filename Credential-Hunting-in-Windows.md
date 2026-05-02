### CPTS / HTB Penetration Tester Path <br>
### Password Attacks - Credential Hunting in Windows <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Credential Hunting in Windows



Once on a Windows host, search files / browsers / app stores / shares for plaintext or weakly-protected creds. Tailor search to host role (IT admin vs. dev vs. user).

#### Common search keywords

`Passwords`, `Passphrases`, `Keys`, `Username`, `User account`, `Creds`, `Users`, `Passkeys`, `configuration`, `dbcredential`, `dbpassword`, `pwd`, `Login`, `Credentials`.

#### Built-in tools

Windows Search (GUI) — searches OS settings + filesystem.

`findstr` for pattern hunting:

```diff
+ C:\> findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml *.git *.ps1 *.yml
```

<br>

#### LaZagne — third-party multi-app credential dumper

Standalone binary releases on GitHub. Modules:

| Module | Targets |
|---|---|
| `browsers` | Chrome, Firefox, Edge, Opera (35 supported) |
| `chats` | Skype, etc. |
| `mails` | Outlook, Thunderbird |
| `memory` | KeePass, LSASS |
| `sysadmin` | OpenVPN, WinSCP |
| `windows` | LSA secrets, Credential Manager |
| `wifi` | WiFi profiles |

Run all modules:

```diff
+ C:\Users\bob\Desktop> start LaZagne.exe all
```

<br>

Verbose mode (`-vv`) shows attempted apps even when nothing's found.

#### High-yield hunting locations

- Group Policy / scripts in `SYSVOL` share
- IT shares — config / scripts
- `web.config` on dev / IT shares
- `unattend.xml`
- AD user/computer description fields
- KeePass DBs (if master password is guessable)
- Files named `pass.txt`, `passwords.docx`, `passwords.xlsx` on user systems / SharePoint

Browser-specific decryption tools: [firefox_decrypt](https://github.com/unode/firefox_decrypt), [decrypt-chrome-passwords](https://github.com/ohyicong/decrypt-chrome-passwords).

<br>

---

<br>

### Exercise

IP: 10.129.202.99 — RDP as `bob` / `HTB_@cademy_stdnt!`.

---

### Question 1:
What password does Bob use to connect to the Switches via SSH? (Format: Case-Sensitive)

RDP with shared folder for transferring loot:

```diff
+ $ xfreerdp /v:10.129.202.99 /u:bob /p:HTB_@cademy_stdnt! /drive:shared,/home/htb-ac-830862/Desktop/sharon
```

<br>

Browsing bob's history files surfaced a notepad with creds + scripts. To be thorough, drop LaZagne onto C:\ and run it (whitelist Defender first if needed):

```diff
+ C:\>start LaZagne.exe all -vv
```

<br>

Output included WinSCP creds, hashdump, and bob's switch + DC + ansible creds in his docs:

	Switches via SSH    admin           WellConnected123
	DC via RDP          bwilliamson     P@55w0rd!
	WinSCP -> ubuntu@10.129.202.64:FSadmin123

&#x1F6A9; found **WellCon--edit--nected123**.

---

### Question 2:
What is the GitLab access code Bob uses? (Format: Case-Sensitive)

#### Found in bob's notes/history during the same hunt.

&#x1F6A9; found **3z1ePfGbjW--edit--PsTfCsZfjy**.

---

### Question 3:
What credentials does Bob use with WinSCP to connect to the file server? (Format: username:password, Case-Sensitive)

#### LaZagne's WinSCP module pulled the cred:

	URL: 10.129.202.64
	Login: ubuntu
	Password: FSadmin123

&#x1F6A9; found **ubuntu:FS--edit--admin123**.

---

### Question 4:
What is the default password of every newly created Inlanefreight Domain user account? (Format: Case-Sensitive)

#### Found inside an `Import-Csv | New-ADUser` PowerShell automation script in bob's `WorkStuff`:

	-AccountPassword (ConvertTo-SecureString "Inlanefreightisgreat2022" -AsPlainText -Force)

&#x1F6A9; found **Inlanefreight--edit--isgreat2022**.

---

### Question 5:
What are the credentials to access the Edge-Router? (Format: username:password, Case-Sensitive)

#### Found in bob's Ansible playbook (work-in-progress) to configure router interfaces:

	user: "{ edgeadmin }"
	passwd: "{ Edge@dmin123!}"

&#x1F6A9; found **edgeadmin:Edge--edit--@dmin123!**.
