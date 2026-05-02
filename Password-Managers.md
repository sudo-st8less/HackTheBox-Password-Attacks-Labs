### CPTS / HTB Penetration Tester Path <br>
### Password Attacks - Password Managers <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Password Managers



Avg user has ~100 unique creds — drives reuse + weak passwords. A password manager stores everything encrypted under a single master password / passphrase.

#### Common products

| Manager | Notes |
|---|---|
| `KeePass` / `KeePassXC` | Local `.kdbx` DB, AES-256, optional keyfile + Yubikey |
| `Bitwarden` | OSS, self-hostable (Vaultwarden), web/CLI/mobile |
| `1Password` | Commercial, secret-key + master password |
| `LastPass` | Commercial; multiple historical breaches |
| `Pass` | Linux CLI, GPG-backed |
| `pwsafe` (Password Safe) | Local `.psafe3` DB |

#### Threat model — what an attacker steals if they get the DB

- The encrypted vault file (`.kdbx`, `.psafe3`, etc.) — useless without the master password
- Master password from memory if the manager is unlocked (mimikatz / lazagne / process dumps)
- Browser auto-fill on a clipboard — captured by clipper malware

#### Crackable formats — `*2john` extractors

| Vault | JtR/hashcat |
|---|---|
| KeePass `.kdbx` | `keepass2john` → JtR `KeePass` / hashcat `-m 13400` |
| Password Safe `.psafe3` | `pwsafe2john` → JtR `pwsafe` |
| 1Password `.opvault` | `1password2john` |
| LastPass exported vault | `lastpass2john` |

Example — extract + crack a `.psafe3`:

```diff
+ $ pwsafe2john vault.psafe3 > vault.hash
+ $ john --wordlist=rockyou.txt vault.hash
```

<br>

KeePass:

```diff
+ $ keepass2john Database.kdbx > kp.hash
+ $ hashcat -m 13400 kp.hash /usr/share/wordlists/rockyou.txt
```

<br>

#### Defender / user best practice

- Use a long passphrase as the master password
- Enable keyfile + Yubikey on KeePass-style managers
- Lock the vault when idle; clear clipboard after copy
- Never store the vault on the same drive as its keyfile
- Disable autofill on untrusted sites
- Audit vault for breached entries (HIBP integrations on Bitwarden / 1Password)
