### CPTS / HTB Penetration Tester Path <br>
### Password Attacks - Password Policies <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Password Policies



Modern guidance (NIST SP 800-63B): length >= 8, no enforced periodic rotation, screen against breach lists, support 2FA. Long passphrases beat short complex passwords.

#### Recommended baseline

| Setting | Recommended |
|---|---|
| Minimum length | `12+` (NIST 8 min, longer = better) |
| Complexity | Allow all printable ASCII + Unicode, don't force special-char rules |
| Periodic rotation | Disabled — only rotate on suspected breach |
| Lockout threshold | `5 failed` attempts → 5-min lockout (or progressive) |
| Breach list screening | Block compromised passwords (HaveIBeenPwned API, etc.) |
| 2FA / MFA | Required for admin + remote access |

#### Common enforcement mechanisms

| Platform | Mechanism |
|---|---|
| AD | Default Domain Policy → Password Policy / Account Lockout Policy / Fine-Grained Password Policies (PSOs) |
| Linux | `/etc/security/pwquality.conf` (PAM `pam_pwquality`), `/etc/login.defs`, `pam_unix` history |
| Network devices | local password-policy stanza or AAA back to TACACS+/RADIUS |
| Web apps | bcrypt/argon2 hashing + breach screening at registration |

#### Storage best practice

- Hash with adaptive function: `bcrypt`, `scrypt`, `argon2id`. Avoid bare MD5/SHA family.
- Always salt per password.
- Rotate hashing parameters as compute scales.

#### Defender side checks

- Audit `passwd` / `shadow` / `NTDS.dit` for weak / reused hashes
- Scan environment variables, scripts, GPP for embedded creds
- Enforce LAPS for local admin password rotation
- Disable LM hash storage (`NoLMHash` policy)
- Disable WDIGEST cleartext caching (KB2871997 patch + reg key)
