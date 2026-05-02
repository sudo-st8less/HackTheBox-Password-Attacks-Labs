### CPTS / HTB Penetration Tester Path <br>
### Password Attacks - Cracking Protected Archives <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Cracking Protected Archives



Common encrypted archive types: `tar`, `gz`, `rar`, `zip`, `vmdb/vmx`, `cpt`, `truecrypt`, `bitlocker`, `kdbx`, `deb`, `7z`. Each needs a specific `*2john` helper.

Identify a file's true format (helps when extension doesn't match):

```diff
+ $ file GZIP.gzip
```

	GZIP.gzip: openssl enc'd data with salted password

<br>

Crack a ZIP — extract hash, run JtR:

```diff
+ $ zip2john ZIP.zip > zip.hash
+ $ john --wordlist=rockyou.txt zip.hash
+ $ john zip.hash --show
```

<br>

Crack OpenSSL-encrypted GZIP via brute-decrypt loop (avoids false-positives from raw hash cracking):

```diff
+ $ for i in $(cat rockyou.txt);do openssl enc -aes-256-cbc -d -in GZIP.gzip -k $i 2>/dev/null| tar xz;done
```

<br>

Crack BitLocker — extract user-password hash, crack with hashcat mode 22100:

```diff
+ $ bitlocker2john -i Backup.vhd > backup.hashes
+ $ grep "bitlocker\$0" backup.hashes > backup.hash
+ $ hashcat -a 0 -m 22100 backup.hash /usr/share/wordlists/rockyou.txt
```

<br>

Mount BitLocker on Linux/macOS via [dislocker](https://github.com/Aorimn/dislocker):

```diff
+ $ sudo apt-get install dislocker
+ $ sudo mkdir -p /media/bitlocker /media/bitlockermount
+ $ sudo losetup -f -P Backup.vhd
+ $ sudo dislocker /dev/loop0p2 -u"<password>" -- /media/bitlocker
+ $ sudo mount -o loop /media/bitlocker/dislocker-file /media/bitlockermount
```

<br>

Cleanup:

```diff
+ $ sudo umount /media/bitlockermount
+ $ sudo umount /media/bitlocker
```

<br>

---

<br>

### Exercise

---

### Question 1:
Run the above target then navigate to http://ip:port/download, then extract the downloaded file. Inside, you will find a password-protected VHD file. Crack the password for the VHD and submit the recovered password as your answer.

Extract hashes from the VHD:

```diff
+ $ bitlocker2john -i /home/htb-ac-830862/Desktop/Private.vhd > /home/htb-ac-830862/Desktop/backup.hashs
```

	Signature found at 0x2200000
	Version: 2 (Windows 7 or later)
	VMK encrypted with User Password found at 22000dc
	VMK encrypted with AES-CCM
	VMK encrypted with Recovery Password found at 0x22001bc

#### Save the user-password hash (`$bitlocker$0$...`) into its own `bkup.hash`, then crack:

```diff
+ $ hashcat -a 0 -m 22100 -d 1,2 bkup.hash /mnt/SSD_DATA/SecLists/Passwords/Leaked-Databases/rockyou.txt
```

	Status...........: Cracked
	Hash.Mode........: 22100 (BitLocker)
	$bitlocker$0$16$...:francisco

&#x1F6A9; found **fran--edit--cisco**.

---

### Question 2:
Mount the BitLocker-encrypted VHD and enter the contents of flag.txt as your answer.

Make the dirs dislocker expects:

```diff
+ $ sudo mkdir -p /media/bitlocker /media/bitlockermount
```

<br>

Setup VHD as loop device:

```diff
+ $ sudo losetup -fP Private.vhd
+ $ losetup -a
```

	/dev/loop0: []: (/home/htb-ac-830862/Desktop/Private.vhd)

<br>

Map partitions inside the VHD with `kpartx`:

```diff
+ $ sudo apt install kpartx
+ $ sudo kpartx -av /dev/loop0
+ $ ls /dev/mapper
```

<br>

Decrypt with dislocker using the cracked password:

```diff
+ $ sudo dislocker -V /dev/loop0p1 -u"francisco" -- /media/bitlocker
```

<br>

Mount and read the flag:

```diff
+ $ sudo mount -o loop /media/bitlocker/dislocker-file /media/bitlockermount
+ $ cat /media/bitlockermount/flag.txt
```

	43d95aeed3114a53ac66f01265f9b7af

&#x1F6A9; found **43d95aeed31--edit--14a53ac66f01265f9b7af**.
