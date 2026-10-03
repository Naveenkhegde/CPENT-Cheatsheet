Penetration Test - Scan
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

# IP Scan
Parrot Environment
```bash
ip route

# 172.19.19.0/24 dev eth1 proto kernel scope link src 172.19.19.18
# 192.168.0.0/24 dev eth0 proto kernel scope link src 192.168.0.18
```
Nmap Parameter
```bash
-sT           TCP connect scan (SYN, SYN+ACK, ACK)
              Full connection, easily logged
              In practice, significantly faster than -sS in certain environments
-sS           TCP SYN stealth scan (half-open)
-sP           Ping ICMP scan (check online hosts)
-sV           Probe open ports to determine service/version info
-sU           UDP scan
-sC           Equivalent to --script=default
-sn           Ping scan (host discovery only). Suitable for scanning large subnets to verify which hosts are up
-Pn           Treat all hosts as online (skip ping). Useful when firewalls block ICMP ping
-O            Enable OS detection
-A            Aggressive scan (OS detection, version detection, script scanning, traceroute)
-n            Never do DNS resolution (use IP addresses directly)
-p-           Scan all ports (1-65535)
-p 1-65535    Specify port range. By default, Nmap scans the top 1000 ports
-p T:80       TCP port 80
-P U:53       UDP port 53
--script vuln Scan for known vulnerabilities
-T4           Set timing template. If network is stable, -T4 is recommended
              Highly recommended to improve scan speed
-oG -         Output scan results in Grepable format to stdout
```
Sudo vs UnSudo
```bash
nmap 192.168.0.7 -sn -n --packet-trace
# Observation: CONN uses TCP 80, 443 to probe the host

sudo nmap 192.168.0.7 -sn -n --packet-trace
# Observation: 
# This time SENT and RCVD are used to probe the host
# With root privileges, Nmap calls raw sockets using ARP requests to detect the host
# Firewalls cannot block ARP; ARP is fundamental to local Ethernet communication


# Is adding sudo completely invincible?
sudo nmap scanme.nmap.org -sn -n --packet-trace
# Observation: 
# This time ICMP type=8, ICMP type=13 are used
# CONN uses 80, 443
# In other words, if firewalls block all these probes, the host will not be found


# Is there a reliable scanning approach that works most of the time?
sudo nmap scanme.nmap.org -sn -PS22,80,445,3389 -n --packet-trace
# Windows : TCP 139,445,3389
# Linux   : TCP 22,80
# Typically, if a machine is running normally, these ports are open; check these ports to determine host status
```
IP Scan Script
```bash
# Nmap
nmap 192.168.0.* -sn -oG -
nmap 192.168.0.* -sn -oG - | grep Up | cut -d' ' -f2 > ip_list

# Port
sudo nmap -n -sn -PS22,80,445,3389 192.168.0.1-254 -oG - | grep Up | cut -d' ' -f2 > ip_list

sudo nmap 192.168.0.* -n -sn -PS22,80,445,3389 -oG > ip_list

# Icmp
ping -c 1 192.168.0.7
ping -c 1 192.168.0.7 | grep "bytes from"
for ip in {1..254};do (ping -c 1 192.168.0.$ip | grep "bytes from" &);done
# & : Run in background so ping executes concurrently while the loop iterates to next IP

# The real essence: ARP requests preceding ICMP cannot be blocked on local subnets
cat /proc/net/arp 
cat /proc/net/arp | grep -v 00:00:00:00:00:00 | grep eth0 | cut -d' ' -f1
```

---

# Port Scan
Nmap Default
```bash
nmap 192.168.0.70
nmap 192.168.0.70 --top 1000
# Observation: 
# Nmap scans 1000 ports by default
# Check /usr/share/nmap/nmap-services to see which ones

# See the top 3 priority ports scanned by Nmap
nmap 192.168.0.70 --top 3 #23 80 443
nmap 192.168.0.70 --top 1 #80

# Full port scan
# Omitting before "-" defaults to 1
# Omitting after "-" defaults to 65535
nmap 192.168.0.70 -p -
```
hping3
```bash
# Enhanced ping tool that handles firewall issues
# Because it uses TCP/UDP packets rather than just ICMP to perform the ping action
sudo hping3 192.168.0.7 -n -S -c 3 -p 80
```
Port Scan Script
```bash
# TCP
bash -c 'echo > /dev/tcp/192.168.0.70/22 && echo open'
bash -c 'for port in {1..100}; do (echo > /dev/tcp/192.168.0.70/$port && echo $port open &);done' 2> /dev/null


# UDP
sudo nmap -sU -p137-139 192.168.0.70  # netbios
# Observation:
# An ICMP type=3 port unreachable response typically indicates that the UDP port is closed
# Because UDP is a connectionless protocol without a 3-way handshake like TCP,
# if a port is closed, the target host usually responds with an ICMP port unreachable error message

# Common UDP ports:
53,69,137,138,161,1900,5353  # Note for exam: port 161 (SNMP) is critical
```
RustScan
```bash
# A fast and aggressive TCP port scanner from port 1 to 65535
# https://github.com/RustScan/RustScan/releases


# Install
wget https://github.com/RustScan/RustScan/releases/download/2.0.1/rustscan_2.0.1_amd64.deb
sudo dpkg -i rustscan_2.0.1_amd64.deb

# Command
rustscan -a 192.168.0.7 -u 5000 -t 8000 
rustscan -a 192.168.0.7 -u 5000 -t 8000 --scripts none
rustscan -a 192.168.0.7 -u 5000 -t 8000 --scripts -- -n -Pn -sVC -oG
# -u : The default Linux ulimit (soft limit) is low (usually 1024), preventing RustScan from reaching its full speed
```


---


# Service Scan
Service
```bash
nmap 192.168.0.7 -p47001
# Windows Remote Management (WinRM)
# WinRM is a Microsoft technology used for remote management and execution on Windows OS
# Communicates over HTTP/HTTPS using SOAP (Simple Object Access Protocol)
# and XML (Extensible Markup Language)

# Windows
sudo nmap -n -p445,3389 192.168.0.8,20 -sVC
# Linux
sudo nmap -n -p22,80 192.168.0.24,70 -sVC
```
Service Script Scan
```bash
nmap 192.168.0.7 -p445 -sV -sC
# -sC  :  ls /usr/share/nmap/scripts
nmap 192.168.0.7 -p445 -script vulners
nmap 192.168.0.7 -sV -p445 -script smb-protocols
```