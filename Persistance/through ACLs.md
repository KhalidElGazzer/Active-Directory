> What is AdminSDHolder?

It is a special container object located in the System **container** within the domain. This container holds the security descriptor (permissions) template used to protect specific privileged accounts and groups, such as Domain Admins, Enterprise Admins, and others, from unauthorized modifications.
> How Does AdminSDHolder Work?

SDProp Process: A process known as Security Descriptor Propagator (**SDProp**) runs periodically on the PDC Emulator (Primary Domain Controller) in each domain. This process ensures that the permissions on all protected accounts and groups match the template set on the AdminSDHolder object. By default, the SDProp process runs every 60 minutes, During each cycle, **SDProp** compares the current security descriptor of each protected object with the AdminSDHolder template and updates any that have been altered, ensuring that they revert to the secure, defined state.

**Persisting with AdminSDHolder:**

Before we start we need to inject our creds to open the **MMC** cuz we need the GUI:
```
runas /netonly /user:<ourUserName>@<DomainName> cmd.exe
```
then:
```
mmc.exe
```
Once you have an MMC window, add the Users and Groups Snap-in (File->Add Snap-In->Active Directory Users and Computers). Make sure to enable Advanced Features (View->Advanced Features). We can find the AdminSDHolder group under Domain->System, Then navigate to the Security of the group (Right-click->Properties->Security).
finally, add our account and grant Full Control:

    Click Add.
    Search for your username and click Check Names.
    Click OK.
    Click Allow on Full Control.
    Click Apply.
    Click OK.
**SDProp**

Now we just need to wait 60 minutes, and our user will have full control over all Protected Groups. This is because the Security Descriptor Propagator (SDProp) service executes automatically every 60 minutes and will propagate this change to all Protected Groups. However, since we do not like to wait, let's kick off the process manually using Powershell using **Invoke-ADSDPropagation** script:
```
PS C:\Tools> Import-Module .\Invoke-ADSDPropagation.ps1 
PS C:\Tools> Invoke-ADSDPropagation
```

Once done, give it a minute and then review the security permissions of a Protected Group such as the Domain Admins group.








