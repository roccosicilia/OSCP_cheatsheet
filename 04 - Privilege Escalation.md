# 04 - Privilege Escalation

Index
- [System Enumeration](#System-Enumeration)
- [SeImpersonatePrivilege](#SeImpersonatePrivilege)

## System Enumeration
Base commands and tools:
``` bash
whoami                              ## domain\username
whoami /groups                      ## my groups
whoami /priv                        ## user's local privileges
Get-LocalUser                       ## local users list
Get-LocalGroup                      ## local groups list
Get-LocalGroupMember $GROUP_NAME    ## members list
Get-ChildItem Env:                  ## environment vars -- tips: informations hidden
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