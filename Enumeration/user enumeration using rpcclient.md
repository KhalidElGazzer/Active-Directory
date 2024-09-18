To enumerate domain users from an Active Directory (AD) environment using ```rpcclient```, the ```enumdomusers``` command is a common method.

1- Connect to the AD server:

First, establish a connection to the Active Directory box using ```rpcclient```. You will need either valid credentials or attempt an *anonymous login*, depending on the AD server configuration.

```
rpcclient -U "" -N <IP_address_of_AD>
```
    -U "": Indicates an empty username, typically for an anonymous login.
    -N: No password prompt.
    <IP_address_of_AD>: The IP address of the target Active Directory server.

If you have valid credentials, you can use:
```
rpcclient -U 'username%password' <IP_address_of_AD>
```
2- Run ```enumdomusers```:

Once connected to the AD server, use the enumdomusers command to enumerate domain users:
```
rpcclient $> enumdomusers
```
This command will output a list of user accounts along with their relative identifiers (RIDs).
<br>
The output will look like this:
```
user:[username] rid:[RID]
```
For example:
```
user:[Administrator] rid:[500]
user:[john.doe] rid:[1001]
```
> The rid (Relative Identifier) is the part of a Security Identifier (SID) unique to each user within a domain.

**Retrieve Detailed Information for Each User (Optional):**

To get more details about a specific user, you can use the queryuser command followed by the user's RID:

```
rpcclient $> queryuser <RID>
```
This method is useful for anonymously or authenticated enumeration of domain users if the AD server does not have restrictive permissions in place.
