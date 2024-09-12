**Enumerating Users**

Enumerating users allows you to know which user accounts are on the target domain and which accounts could potentially be used to access the network.
<br>
This will brute force user accounts from a domain controller using a supplied wordlist
```
./kerbrute userenum --dc <DC name> -d <domain name> ~/wordlists/active_directory_users.txt
```
