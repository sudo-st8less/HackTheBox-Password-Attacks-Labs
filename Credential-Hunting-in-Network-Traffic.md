### CPTS / HTB Penetration Tester Path <br>
### Password Attacks - Credential Hunting in Network Traffic <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Credential Hunting in Network Traffic



Plaintext protocols still appear in legacy / misconfigured / dev environments. Cleartext = creds in PCAP.

| Plaintext | Encrypted alt | Use case |
|---|---|---|
| HTTP | HTTPS | Web |
| FTP | FTPS / SFTP | File transfer |
| SNMP | SNMPv3 | Device mgmt |
| POP3 | POP3S | Mail retrieve |
| IMAP | IMAPS | Mail mgmt |
| SMTP | SMTPS | Mail send |
| LDAP | LDAPS | Directory |
| DNS | DoH | Name resolution |
| SMB | SMB 3.0 + TLS | File sharing |
| VNC | VNC + TLS | Remote desktop |

#### Wireshark display filters

| Filter | Purpose |
|---|---|
| `ip.addr == X.X.X.X` | Specific IP |
| `tcp.port == 80` | Specific port |
| `http` / `dns` / `icmp` | Protocol |
| `tcp.flags.syn == 1 && tcp.flags.ack == 0` | SYN packets |
| `http.request.method == "POST"` | POSTed forms |
| `tcp.stream eq 53` | Single TCP stream |
| `eth.addr == 00:11:22:...` | MAC address |
| `http contains "passw"` | String search |

`Edit > Find Packet` for direct string searches inside packet bytes.

#### Pcredz — automated credential extractor

Pulls: CC numbers, POP/SMTP/IMAP/SNMP/FTP/HTTP creds, NTLMv1/v2 hashes (DCE-RPC, SMB1/2, LDAP, MSSQL, HTTP), Kerberos AS-REQ pre-auth etype 23.

```diff
+ $ ./Pcredz -f demo.pcapng -t -v
```

<br>

---

<br>

### Exercise

---

### Question 1:
The packet capture contains cleartext credit card information. What is the number that was transmitted?

In Wireshark filter bar:

```diff
+ http contains "card"
```

<br>

Find the POST → expand `HTML Form URL Encoded` in the protocol tree:

	Form item: "card_name" = "Joshua M Benito"
	Form item: "card_number" = "5156 8829 4478 9834"
	Form item: "exp_date" = "12/30"
	Form item: "cvv" = "928"

&#x1F6A9; found **5156 8829 44--edit--78 9834**.

---

### Question 2:
What is the SNMPv2 community string that was used?

Clone Pcredz, install deps, run against the supplied PCAP:

```diff
+ $ git clone https://github.com/lgandx/PCredz.git
+ $ wget https://academy.hackthebox.com/storage/modules/147/credential-hunting-in-network-traffic.zip
+ $ unzip credential-hunting-in-network-traffic.zip
+ $ sudo apt install python3-pip && sudo apt-get install libpcap-dev && pip3 install Cython && pip3 install python-libpcap
+ $ python3 ./Pcredz -f demo.pcapng
```

	Found SNMPv2 Community string: s3cr3tSNMPC0mmun1ty
	FTP User: leah
	FTP Pass: qwerty123

&#x1F6A9; found **s3cr3tSNMP--edit--C0mmun1ty**.

---

### Question 3:
What is the password of the user who logged into FTP?

#### Same Pcredz output captured the FTP cleartext.

&#x1F6A9; found **qwer--edit--ty123**.

---

### Question 4:
What file did the user download over FTP?

In Wireshark, filter `FTP`. Look at the response message just above `226 Transfer complete`.

&#x1F6A9; found **creds.txt**.
