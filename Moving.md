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
There are two protocols when establishing a WMI session **DCOM** **Wsman**.<br>h
