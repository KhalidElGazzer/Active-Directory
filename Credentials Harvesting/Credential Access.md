**Credential Access Overview**

Credential access involves attackers obtaining user credentials from compromised systems to impersonate users, facilitating lateral movement and access to other resources. Attackers prefer acquiring legitimate credentials over exploiting vulnerabilities.

**Common Storage Locations for Credentials:**

> Clear-text Files:

Credentials can be found in command histories, configuration files, backup files, shared files, and registry entries. For example, PowerShell commands are saved in ```C:\Users\USER\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt```.<br>
We can also search for the "password" keyword in the Registry:
```
c:\Users\user> reg query HKLM /f password /t REG_SZ /s
#OR
C:\Users\user> reg query HKCU /f password /t REG_SZ /s
```

> Database Files: 

Applications may store credentials in local database files, which are valuable targets for attackers.

> Password Managers:

These applications store and manage login information. Misconfigurations or vulnerabilities can expose stored credentials.

> Memory Dumps: 

Sensitive information, such as clear-text credentials and cached passwords, can be extracted from memory, though this requires administrator-level access.

> Active Directory:

AD stores critical data, including user credentials. Misconfigurations, such as storing passwords in user descriptions:
```
Get-ADUser -Filter * -Properties * | select Name,SamAccountName,Description
```

Or leaking keys in Group Policy SYSVOL and NTDS make AD a prime target.
**Why NTDS.dit is a Target for Attackers:**

    The NTDS.dit file contains the hashed passwords of all users in the AD environment.
    If attackers gain access to this file, they can extract these password hashes and attempt to crack them offline, potentially gaining access to user accounts, including privileged accounts like domain administrators.

> Network Sniffing:

Attackers can use network sniffing techniques, like Man-in-the-Middle attacks, to capture authentication information such as NTLM hashes.






















