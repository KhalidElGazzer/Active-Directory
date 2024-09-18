** when enumerate smb shares and we have read access to ```IPC$``` share, We will be able to list the domain users as **anonymous** using different tools:

1- ```lookupsid.py anonymous@<target ip>```<br>
2- ```crackmapexec smb <target ip> -u 'guest' -p '' --rid-brute```
----------------------------------------------------------------------------------------------------------------------

** if we do not know the domain name we could use ```crackmapexec``` to dump some cool information:
```
crackmapexec smb <target ip>
                                                                  
SMB         10.10.250.17    445    VULNNET-BC3TCK1  [*] Windows 10.0 Build 17763 x64 (name:VULNNET-BC3TCK1) (domain:vulnnet.local) (signing:True) (SMBv1:False)
```
