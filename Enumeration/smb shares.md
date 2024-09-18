We can enumerate any shares that the domain controller may be giving out.
```
smbclient -L <target ip> -U <domain name>/<username we have>%<password>
#OR
smbclient -L <target ip>
```
**To list the permissions we have over the shares:**
```
smbmap -u <username> -p <password> -H <target ip>
#OR
crackmapexec smb <target ip> -u <username> -p <password> --shares  #we may did not submit any value to '-p' 
```
To make an smb prompt:
```
smbclient \\\\<target ip>\\<sharename we have access to> -U <username we have>
```
