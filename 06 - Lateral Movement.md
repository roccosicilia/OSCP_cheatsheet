# 06 - Lateral Movement

Index
- [Windows New-PSSession](#Windows-New-PSSession)

## Windows New-PSSession
``` powershell
$username = 'username';
$password = 'password';
$secureString = ConvertTo-SecureString $password -AsPlaintext -Force;
$credential = New-Object System.Management.Automation.PSCredential $username, $secureString;
New-PSSession -ComputerName 192.168.1.1 -Credential $credential                                 ## create a session
Enter-PSSession 1                                                                               ## login in session 1
```

## Evil-winrm
Linux tool to access a remote system via WinRM and obtain a shell.
``` bash
evil-winrm -i 192.168.163.249 -u 'username' -p 'password'
```
