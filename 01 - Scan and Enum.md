# 01 - Scan and Enumeration

Index
- [Port Scan](#Port-Scan)
- [Web Scan](#Web-Scan)
- [Service Enumeration](#Service-Enumeration)
	- [FTP](#FTP)
	- [SMB](#SMB)
	- [SNMP](#SNMP)

## Port Scan
``` bash
nc -nvv -w 1 -z 192.168.1.1 1-1000 > nc.txt 2>&1		## TCP port via netcat
nc -nvv -u -w 1 -z 192.168.1.1 1-1000 > nc.txt 2>&1		## UDP port via netcat
sudo nmap -sT -p- 192.168.1.1							## TCP scan, all port
sudo nmap -sS -p- 192.168.1.1							## SYN scan, all port
sudo nmap -sC -p- 192.168.1.1							## script default, all port
sudo nmap -sT -p- --script vuln 192.168.1.1				## vuln script, all port
sudo nmap -sT -p- --script all 192.168.1.1				## all script, all port
sudo nmap -sS -sC -sV -oN nmap.txt 192.168.1.1			## SYN scan, script default, file output
sudo nmap -sU --top-ports=100 192.168.1.1				## UDP scan

### WARNING ### VERY SLOW ### UDP SCAN ###
sudo nmap -sU 192.168.1.1
sudo nmap -sU -sS 192.168.1.1
```

## Web Scan
 - check all sub-directory
 - check for file extension: txt, pdf, sql, sh, ps1, bak, kdbx, zip, tar, git
 - analyze pdf meta-data
``` bash
nikto -h http://192.168.1.1:8080                                                                    ## NIKTO scan, webapp enumeration

gobuster dir -u http://192.168.1.1 -w /usr/share/wordlists/dirb/common.txt                    		## directory enum
gobuster dir -u http://192.168.1.1 -w /usr/share/wordlists/dirb/big.txt	                    		## directory enum
gobuster dir -u http://192.168.1.1 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt	## directory enum (large)
gobuster dir -u http://192.168.1.1 -w /usr/share/wordlists/dirb/big.txt -x txt                 		## txt file
gobuster dir -u http://192.168.1.1 -w /usr/share/wordlists/dirb/big.txt -x pdf                 		## pdf file
gobuster dir -u http://192.168.1.1 -w /usr/share/wordlists/dirb/big.txt --exclude-length 1917  		## length exclusion

feroxbuster --url http://192.168.1.1/ --depth 2 --wordlist /usr/share/wordlists/dirb/common.txt		## faster and useful dir enum
```

CMS scan: 
``` bash
wpscan --url http://192.168.1.1													## standard wordpress scan
wpscan --url http://192.168.1.1 --enumerate p --plugins-detection aggressive	## vulnerable plugin
joomscan -u http://192.168.1.1/path-to-cms										## standard joomla scan
joomscan -u http://192.168.1.1/path-to-cms -ec									## components enum
```

Git repo:
``` bash
git-dumper http://192.168.1.1/.git/ $destination_dir	## repository dump if /.git/ dir available
git log													## commit logs
git show $id											## commit info and message
```

Windows Service Enum:
``` bash
sudo nbtscan -r 192.168.1.0/24												## scan for win services (ex: SAMBA)
crackmapexec smb 192.168.1.1 -d domain.local -u 'guest' -p '' --rid-brute	## RID brute forcing for user enumeration
enum4linux 192.168.1.1
enum4linux -a -u '$username' -p '$password' 192.168.1.1
```

## Service Enumeration
### FTP
Try anonymous FTP acccess.
``` bash
$ ftp 192.168.1.1                       ## target host
Name (192.168.1.1:user): anonymous      ## insert "anonymous"
Password: ....                          ## blank 
230 Login successful.
ftp>
```

### SMB
Try anonymous SMB acccess.
``` bash
smbclient -L //192.168.1.1/             ## target host
Password for [WORKGROUP\user]:          ## blank

	Sharename       Type      Comment
	---------       ----      -------
	C$              Disk      Default share
	Share           Disk      
[...]
smbclient //192.168.1.1/Share           ## target host
Password for [WORKGROUP\user]:          ## blank
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Thu Oct 13 19:19:09 2022
  ..                                DHS        0  Mon Nov  4 12:38:09 2024
  Users                               D        0  Thu Oct 13 19:19:08 2022      ## directory listing
```

### SNMP
``` bash
onesixtyone -c community.txt -i target.txt								## community discovery
snmpwalk -c public -v1 192.168.1.1										## get snmp content
snmpwalk -c public -v1 192.168.1.1 NET-SNMP-EXTEND-MIB::nsExtendObjects	## get snmp EXTEND MIB
```
