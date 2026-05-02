### CPTS / HTB Penetration Tester Path <br>
### Password Attacks - Writing Custom Wordlists and Rules <br>
<mark>hook it up with a &#x2B50; if this helps.</mark> <br>
🐦: @<a href="https://x.com/st8less">**st8less**</a>

<br>
<br>

---

### Writing Custom Wordlists and Rules



Most password policies enforce min-length + uppercase/lowercase/digit/symbol — but users still pick predictable patterns: company name, season + year, pet/family names, hobbies. Combine OSINT with rule-based mutations to target individuals.

Common end-user mutations to layer in via rules:

| Mutation | Example |
|---|---|
| Capitalize first | `Password` |
| Append digits | `Password123` |
| Append year | `Password2022` |
| Append month | `Password02` |
| Append `!` | `Password2022!` |
| Leet substitution | `P@ssw0rd2022!` |

Hashcat rule syntax (subset):

| Function | Description |
|---|---|
| `:` | no-op |
| `l` / `u` / `c` | lowercase / uppercase / capitalize |
| `sXY` | replace all X with Y |
| `$!` | append `!` to end |

Example rule file:

```diff
+ :
+ c
+ so0
+ c so0
+ sa@
+ c sa@
+ c sa@ so0
+ $!
+ $! c
+ $! so0
+ $! sa@
+ $! c so0
+ $! c sa@
+ $! so0 sa@
+ $! c so0 sa@
```

<br>

Generate mutated wordlist (no cracking, just stdout):

```diff
+ $ hashcat --force password.list -r custom.rule --stdout | sort -u > mut_password.list
```

<br>

Scrape company website for words with [CeWL](https://github.com/digininja/CeWL):

```diff
+ $ cewl https://www.inlanefreight.com -d 4 -m 6 --lowercase -w inlane.wordlist
+ $ wc -l inlane.wordlist
```

<br>

CeWL flags: `-d` depth, `-m` min length, `--lowercase`, `-w` output.

<br>

---

<br>

### Exercise

We compromised the password hash for `Mark White`'s work email. OSINT gathered:

- DOB: `August 5, 1998`
- Employer: `Nexura, Ltd.` (12-char min, mixed case + symbol + digit)
- Lives in `San Francisco, CA, USA`
- Cat `Bella`, wife `Maria`, son `Alex`, fan of `baseball`

Hash: `97268a8ae45ac7d15c3cea4ce6ea550b`

---

### Question 1:
What is Mark's password?

#### Built `mark.list` — ~100 candidate strings derived from the OSINT (combinations of `Mark`, `White`, `1998`, `Nexura`, `Bella`, `Maria`, `Alex`, `Baseball`, `SF`, `CA`, `0805`, plus separators/symbols).

Save `custom.rule` from the section reading. Generate the mutated list:

```diff
+ $ hashcat --force -r custom.rule mark.list --stdout | sort -u > mutations.list
```

<br>

Crack:

```diff
+ $ hashcat -a 0 -m 0 97268a8ae45ac7d15c3cea4ce6ea550b mutations.list -d 1,2
```

	* Filename..: mutations.list
	* Passwords.: 896
	97268a8ae45ac7d15c3cea4ce6ea550b:Baseball1998!

&#x1F6A9; found **Bas--edit--eball1998!**.
