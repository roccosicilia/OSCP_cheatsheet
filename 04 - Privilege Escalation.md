# 04 - Privilege Escalation

Index
- [System Enumeration](#System-Enumeration)
- [SeImpersonatePrivilege](#SeImpersonatePrivilege)

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


## SeImpersonatePrivilege
Verify user's privileges:
``` text
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

### Exploit by PrintSpoofer64.exe
...