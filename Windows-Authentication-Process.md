### CPTS / HTB Penetration Tester Path <br>
### Password Attacks - Windows Authentication Process <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Windows Authentication Process



Local interactive logon flow on Windows:

`WinLogon` → `LogonUI` (collects creds via credential providers / COM DLLs) → `LSASS` → authentication packages → SAM (workgroup) or AD (`ntds.dit`).

| Component | Role |
|---|---|
| `WinLogon` | Trusted process — handles login UI, password change, lock/unlock |
| `LogonUI` | GUI shell for credential prompts |
| `LSASS` (`%SystemRoot%\System32\Lsass.exe`) | Enforces local security policy, validates creds, writes Event Log |
| `LSA` | Maintains policies + SID translation |
| `SAM` | Local user DB (`%SystemRoot%\system32\config\SAM`, mounted at `HKLM\SAM`, requires SYSTEM to read) |
| `NTDS.dit` | AD database on DCs (`%SystemRoot%\ntds`) — synced across all writable DCs |

LSASS authentication packages (DLLs):

| DLL | Purpose |
|---|---|
| `Lsasrv.dll` | LSA Server — Negotiate function picks NTLM or Kerberos |
| `Msv1_0.dll` | Local logon, no custom auth |
| `Samsrv.dll` | SAM — local accounts + policy |
| `Kerberos.dll` | Kerberos |
| `Netlogon.dll` | Network logon service |
| `Ntdsa.dll` | NTDS access (registry creation) |

`SYSKEY` (`syskey.exe`) — Windows NT 4.0+ feature partially encrypts SAM file with a system-generated key to resist offline cracking.

`Credential Manager` — built-in (Server 2008 R2 / Win 7+). Stores creds per-user under:

`PS C:\Users\[Username]\AppData\Local\Microsoft\[Vault/Credentials]\`

`NTDS.dit` stores user accounts (incl. password hashes), groups, computer accounts, GPOs. Synced across all DCs except RODCs.

Domain-joined logons go to a DC; SAM still reachable for local accounts via `WS01\user` or `.\user`.
