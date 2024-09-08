> What is Local Security Authority Subsystem Service?

Is a Windows **process** that handles the operating system security policy and enforces it on a system. It verifies logged in accounts and ensures passwords, hashes, and Kerberos tickets. Windows system stores credentials in the LSASS process to enable users to access network resources, such as file shares, SharePoint sites, and other network services, without entering credentials every time a user connects,Thus, the LSASS process is a juicy target for red teamers because it stores sensitive information about user accounts.<br>

Luckily for us, if we have **administrator privileges**, we can dump the process memory of **LSASS**. Windows system allows us to create a dump file, a snapshot of a given process.<br>

This could be done either with the Desktop access (GUI) or the command prompt.

**First: the GUI**

To dump any running Windows process using the GUI, open the Task Manager, and from the Details tab, find the required process, right-click on it, and select "Create dump file".<br>
Once the dumping process is finished, a pop-up message will show containing the path of the dumped file. Now copy the file and transfer it to our machine to extract NTLM hashes offline using **mimikatz**. 

**Second: the command prompt**
```
procdump.exe -accepteula -ma lsass.exe /path/to/mimikatz/folder
```
> note

If we try these two methods we will get an error indicating that the "access is denied" cuz Microsoft implemented an LSA protection, to keep LSASS from being accessed to extract credentials from memory.<br>

Luckily for us, we will use **mimikatz** to disable this protection.
<br>
If we try to dump the creds from the **lsass** process using mimikatz we will get an error like this:
```       
mimikatz # sekurlsa::logonpasswords
ERROR kuhl_m_sekurlsa_acquireLSA ; Handle on memory (0x00000005)
```
The command returns a 0x00000005 error code message (Access Denied).<br>

**Disable LSASS protection**

Mimikatz provides a **mimidrv.sys** driver that works on kernel level to disable the LSA protection. We can import it to Mimikatz by executing "**!+**" as follows:
```
mimikatz # !+
[*] 'mimidrv' service not present
[+] 'mimidrv' service successfully registered
[+] 'mimidrv' service ACL to everyone
[+] 'mimidrv' service started
```
Once the driver is loaded, we can disable the LSA protection by executing the following Mimikatz command:
```
mimikatz # !processprotect /process:lsass.exe /remove
Process : lsass.exe
PID 528 -> 00/00 [0-0-0]
```       
Finally, we will be enable to dump the creds using **LSASS**.
> note

Mimikatz can operate in two modes for extracting credentials from the LSASS process:

1- Direct Access to LSASS (Live Mode):

Mimikatz can interact directly with the LSASS process in memory if you run it with sufficient privileges on the target system.
```
privilege::debug
sekurlsa::logonpasswords
```
2- Using a Dump File (Offline Mode):

If you have already created a dump of the LSASS process memory using tools like procdump, you can provide this dump file to Mimikatz for analysis.
```
sekurlsa::logonpasswords /dumpfile:C:\path\to\lsass_dump
```   
































