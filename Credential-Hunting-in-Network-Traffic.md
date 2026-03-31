### CPTS / HTB Penetration Tester Path <br>
### Password Attacks: Credential Hunting in Network Traffic <br>
<mark>hook it up with a follow if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Question 1:
The packet capture contains cleartext credit card information. What is the number that was transmitted?

In your filter bar in Wireshark, simply search for `http contains "card"`. Look for the POST request, and expand the 'html form url encoded...' section in the OSI layer inspection window.

	Frame 10465: 676 bytes on wire (5408 bits), 676 bytes captured (5408 bits) on interface eth1, id 0
	Ethernet II, Src: ASUSTekCOMPU_1c:2c:77 (f0:2f:74:1c:2c:77), Dst: VMware_7c:28:ec (00:0c:29:7c:28:ec)
	Internet Protocol Version 4, Src: 192.168.31.243, Dst: 192.168.31.238
	Transmission Control Protocol, Src Port: 55705, Dst Port: 80, Seq: 1, Ack: 1, Len: 622
	Hypertext Transfer Protocol
	HTML Form URL Encoded: application/x-www-form-urlencoded
	    Form item: "card_name" = "Joshua M Benito"
	    Form item: "card_number" = "5156 8829 4478 9834"
	    Form item: "exp_date" = "12/30"
	    Form item: "cvv" = "928"
	    Form item: "product_id" = "SHRT553"
	    Form item: "quantity" = "3"
	    Form item: "price" = "49.99"

🚩 found **5156 8829 4478 9834**.

---

### Question 2:
What is the SNMPv2 community string that was used?

Clone the PCredz repo:
```diff
+ $ git clone https://github.com/lgandx/PCredz.git
```

Download & unzip the pcap:
```diff
+ $ wget https://academy.hackthebox.com/storage/modules/147/credential-hunting-in-network-traffic.zip
+ $ unzip credential-hunting-in-network-traffic.zip
```

Install dependencies:
```diff
+ $ sudo apt install python3-pip && sudo apt-get install libpcap-dev && pip3 install Cython && pip3 install python-libpcap
```

Now run it:
```diff
+ $ python3 ./Pcredz -f demo.pcapng
```

	Pcredz 2.0.3
	
	Author: Laurent Gaffie <lgaffie@secorizon.com>
	
	This script will extract NTLM (HTTP,LDAP,SMB,MSSQL,RPC, etc), Kerberos,
	FTP, HTTP Basic and credit card data from a given pcap file or from a live interface.
	
	CC number scanning activated
	
	Unknown format, trying TCPDump format
	
	protocol: udp 192.168.31.211:59022 > 192.168.31.238:161
	Found SNMPv2 Community string: s3cr3tSNMPC0mmun1ty
	
	protocol: tcp 192.168.31.243:55707 > 192.168.31.211:21
	FTP User: leah
	FTP Pass: qwerty123
	
	
	demo.pcapng parsed in: 1.98 seconds (File size 15.5 Mo).

This gave us our next two answers.

🚩 found **s3cr3tSNMPC--edit--0mmun1ty**.

---

### Question 3:
What is the password of the user who logged into FTP?

🚩 found **qwerty123**.

---

### Question 4:
What file did the user download over FTP?

Filter Wireshark by `FTP` and look at the response message directly above the '226 Transfer Complete' message.

🚩 found **creds.txt**.
