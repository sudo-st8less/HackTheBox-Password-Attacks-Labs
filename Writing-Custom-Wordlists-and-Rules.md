### CPTS / HTB Penetration Tester Path <br>
### Password Attacks: Writing Custom Wordlists and Rules <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>
<br>
<br>

---

For this sections exercise, imagine that we compromised the password hash of a `work email` belonging to `Mark White`. After performing a bit of OSINT, we have gathered the following information about Mark:

- He was born on `August 5, 1998`
- He works at `Nexura, Ltd.`
    - The company's password policy requires passwords to be at least 12 characters long, to contain at least one uppercase letter, at least one lowercase letter, at least one symbol and at least one number
- He lives in `San Francisco, CA, USA`
- He has a pet cat named `Bella`
- He has a wife named `Maria`
- He has a son named `Alex`
- He is a big fan of `baseball`

The password hash is: `97268a8ae45ac7d15c3cea4ce6ea550b`. Use the techniques covered in this section to generate a custom wordlist and ruleset targeting Mark specifically, and crack the password.

---

### Question 1:
What is Mark's password?

First create a custom wordlist, calling it mark.list. Then I prompted an LLM to make me a list of 100 possible passwords given the criteria:

	MarkWhite1998!
	Nexura2024!Mark
	Bella@Home98
	Baseball#1998
	Maria&Mark2024
	SanFran#Nexura1
	WhiteFamily@24
	BellaCat!0805
	Nexura!Mark98
	BaseballFan#98
	SFMark@1998
	MariaAlex#05
	Nexura_White24
	Bella&Maria98
	Mark@Nexura2024
	Nexura!Baseball98
	MWhite0805!
	Bella#Baseball1
	SanFran!Mark98
	Alex&Maria0805
	White@Bella2024
	MarkInSF#1998
	Mark1998Maria!
	Maria0805Alex!
	Bella0805White!
	SanFrancisco98!
	Nexura#White24
	BaseballDad@24
	Nexura!1998
	baseball1998!
	MariaMark!98
	SFBaseball@1998
	CatBella#0805
	Mark!08Aug98
	Nexura!Bella24
	WhiteMark#CA98
	Baseball@SF1998
	Maria&Alex#1998
	Bella08051998!
	MWhite#Nexura24
	Baseball_White24
	Mark&Bella#24
	NexuraTeam!98
	SF!White0805
	MarkLovesMaria@
	MariaAlexBella!
	NexuraBella0805!
	BaseballDad98!
	Bella@Nexura98
	AlexMark@2005!
	WhiteFamily#CA
	MarkInCA#98!
	NexuraSF@24!
	Base@Nexura98!
	Maria_Bella#24
	SFMark#2024!
	CatDadMark@08
	0805#Baseball!
	Nexura#MarkW98
	Baseball_Mark@
	White@Nexura08
	SF_California#24
	Bella&Alex@05
	NexuraBaseball1998!
	NexuraMariaAlex!
	Nexura!LTD1998
	MarkSF!1998
	CAWhite@24!
	Baseball@Bella24
	MariaNexura#98
	M@rk1998!
	M@rkWh!te1998
	WhiteFam0805!
	Baseball#White98
	BellaMark2024!
	MWhite@CA24
	Nexura@SF_1998
	BaseballFan@CA
	Mark&Bella@24
	White@Baseball!
	NexuraTeam#98!
	Maria&Mark#SF
	1998Bella#Mark
	BaseballDad#98
	White@Nexura2024
	MarkWhite@CA98!
	Baseball_Family24
	Nexura08!MarkW
	MWhite#0805CA
	Bella@Base98!
	MariaNexura@24
	Nexura08051998!
	M@ria0805!
	SanFranMark!98
	NexuraMark1998!
	Mark&Bella0805
	Baseball#Fan1998
	White_Nexura@CA
	MWhite@Nexura#08
	Nexura_SF#2024
	Maria&Alex_Base#
	CatBella@White!
	BaseballDad_CA98!
	NexuraFamily#24
	SanFraBaseball@
	MarkWhite#LTD24
	Bella!SF@Nexura
	Baseball#Nexura!
	Mark&Family2024!
	WhiteMark!Bella98
	Bella_0805_Mark!
	BaseballCA#White
	Nexura_Mark_CA98!
	SFBaseballDad@98

Saved that in mark.list.

Now create a rule file with the rules given in this chapter:

	:
	c
	so0
	c so0
	sa@
	c sa@
	c sa@ so0
	$!
	$! c
	$! so0
	$! sa@
	$! c so0
	$! c sa@
	$! so0 sa@
	$! c so0 sa@

Now lets create the mutated list:
```diff
+ $ hashcat --force -r custom.rule mark.list --stdout | sort -u > mutations.list
```

And crack it:
```diff
+ $ hashcat -a 0 -m 0 97268a8ae45ac7d15c3cea4ce6ea550b mutations.list -d 1,2
```

	hashcat (v6.2.6) starting
	
	CUDA API (CUDA 13.0)
	====================
	* Device #1: NVIDIA GeForce RTX 4070, 10341/11850 MB, 46MCU
	* Device #2: NVIDIA GeForce GTX 1080 Ti, 11022/11165 MB, 28MCU
	
	OpenCL API (OpenCL 3.0 CUDA 13.0.84) - Platform #1 [NVIDIA Corporation]
	=======================================================================
	* Device #3: NVIDIA GeForce RTX 4070, skipped
	* Device #4: NVIDIA GeForce GTX 1080 Ti, skipped
	
	Minimum password length supported by kernel: 0
	Maximum password length supported by kernel: 256
	
	Hashes: 1 digests; 1 unique digests, 1 unique salts
	Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
	Rules: 1
	
	Optimizers applied:
	* Zero-Byte
	* Early-Skip
	* Not-Salted
	* Not-Iterated
	* Single-Hash
	* Single-Salt
	* Raw-Hash
	
	ATTENTION! Pure (unoptimized) backend kernels selected.
	Pure kernels can crack longer passwords, but drastically reduce performance.
	If you want to switch to optimized kernels, append -O to your commandline.
	See the above message to find out about the exact limits.
	
	Watchdog: Temperature abort trigger set to 90c
	
	Host memory required for this attack: 1299 MB
	
	Dictionary cache built:
	* Filename..: mutations.list
	* Passwords.: 896
	* Bytes.....: 14188
	* Keyspace..: 896
	* Runtime...: 0 secs
	
	The wordlist or mask that you are using is too small.
	This means that hashcat cannot use the full parallel power of your device(s).
	Unless you supply more work, your cracking speed will drop.
	For tips on supplying more work, see: https://hashcat.net/faq/morework
	
	Approaching final keyspace - workload adjusted.
	
	97268a8ae45ac7d15c3cea4ce6ea550b:Baseball1998!
	
	Session..........: hashcat
	Status...........: Cracked
	Hash.Mode........: 0 (MD5)
	Hash.Target......: 97268a8ae45ac7d15c3cea4ce6ea550b
	Time.Started.....: Sat Oct 11 16:20:43 2025 (0 secs)
	Time.Estimated...: Sat Oct 11 16:20:43 2025 (0 secs)
	Kernel.Feature...: Pure Kernel
	Guess.Base.......: File (mutations.list)
	Guess.Queue......: 1/1 (100.00%)
	Speed.#1.........:        0 H/s (0.00ms) @ Accel:2048 Loops:1 Thr:32 Vec:1
	Speed.#2.........:  3660.3 kH/s (0.01ms) @ Accel:2048 Loops:1 Thr:32 Vec:1
	Speed.#*.........:  3660.3 kH/s
	Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
	Progress.........: 896/896 (100.00%)
	Rejected.........: 0/896 (0.00%)
	Restore.Point....: 0/896 (0.00%)
	Restore.Sub.#1...: Salt:0 Amplifier:0-0 Iteration:0-1
	Restore.Sub.#2...: Salt:0 Amplifier:0-1 Iteration:0-1
	Candidate.Engine.: Device Generator
	Candidates.#1....: [Copying]
	Candidates.#2....: 0805#baseball! -> White_Nexur@@CA!
	Hardware.Mon.#1..: Temp: 56c Fan:  0% Util: 11% Core:2520MHz Mem:10501MHz Bus:16
	Hardware.Mon.#2..: Temp: 24c Fan:  0% Util: 61% Core:1721MHz Mem:5005MHz Bus:4
	
	Started: Sat Oct 11 16:20:42 2025
	Stopped: Sat Oct 11 16:20:44

🚩 found **Baseb--edit--all1998!**.
