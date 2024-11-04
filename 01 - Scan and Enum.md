# 01 - Scan and Enumeration

Index
- [Port Scan](#Port Scan)
- [Web Scan](#Web Scan)
- [Service Enumeration](#Service Enumeration)

## Port Scan
``` bash
nc -nvv -w 1 -z 192.168.1.1 1-1000              ## TCP port via netcat
nc -nvv -u -w 1 -z 192.168.1.1 1-1000           ## UDP port via netcat
nmap -sT -p- 192.168.1.1                        ## TCP scan, all port
nmap -sS -p- 192.168.1.1                        ## SYN scan, all port
nmap -sC -p- 192.168.1.1                        ## script default, all port
nmap -sT -p- --script vuln 192.168.1.1          ## vuln script, all port
nmap -sT -p- --script all 192.168.1.1           ## all script, all port
nmap -sS -sC -sV -oN file.txt 192.168.1.1       ## SYN scan, script default, file output
```

## Web Scan
``` bash
nikto -h http://192.168.1.1:8080                                                                    ## NIKTO scan, webapp enumeration

gobuster dir -u http://192.168.1.1:8080 -w /usr/share/wordlists/dirb/big.txt                        ## directory enum
gobuster dir -u http://192.168.1.1:8080 -w /usr/share/wordlists/dirb/common.txt                     ## directory enum
gobuster dir -u http://192.168.1.1:8080 -w /usr/share/wordlists/dirb/bit.txt -x txt                 ## txt file
gobuster dir -u http://192.168.1.1:8080 -w /usr/share/wordlists/dirb/bit.txt -x pdf                 ## txt file
gobuster dir -u http://192.168.1.1:8080 -w /usr/share/wordlists/dirb/bit.txt --exclude-length 1917  ## length exclusion

wpscan --url http://192.168.1.1                                                                     ## standard scan
wpscan --url http://192.168.1.1 --enumerate p --plugins-detection aggressive                        ## vulnerable plugin
```

## Service Enumeration

- Web Server Version
- HTML header
- File robots.txt: sub-directory to scan
- Not stardard service PORT
- Anonymous Access and brute-force:
 - FTP: try anon login
 - SMB: try anon login
 - SSH: try Hydra brute force
 