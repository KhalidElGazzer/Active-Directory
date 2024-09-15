We can enumerate any shares that the domain controller may be giving out.
```
 smbclient -L <target ip> -U <domain name>/<username we have>%<password>
```
To list the permissions we have over the shares:
```
smbmap -u <username> -p <password> -H <target ip>
```
To make an smb prompt:
```
smbclient \\\\<target ip>\\<sharename we have access to> -U <username we have>
```
