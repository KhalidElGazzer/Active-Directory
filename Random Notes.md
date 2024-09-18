** when enumerate smb shares and we have read access to ```IPC$``` share, We will be able to list the domain users as **anonymous** using different tools:

1- ```lookupsid.py anonymous@<target ip>```<br>
2- ```crackmapexec smb <target ip> -u 'guest' -p '' --rid-brute```
----------------------------------------------------------------------------------------------------------------------

