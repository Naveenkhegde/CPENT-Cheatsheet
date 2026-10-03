SSH port forwarding
===
🔙 [MENU README](./Pivot%20&%20Double%20Pivot.md)


# Local
![](./SSH%20local%20port%20forwarding.png)
```bash
# Open port locally
```
```bash
# Parrot Command
sudo ssh -L *:80:192.168.0.24:80 administrator@192.168.0.70
# administrator / Infinit3
# Not Need Shadow Terminal
sudo ssh -nNTL *:80:192.168.0.24:80 administrator@192.168.0.70

# Connect
Parrot web connect to localhost:80
nc 127.0.0.1 80

# Observe connection status across machines
sudo netstat -antp | grep :80
# :80  => bound to 127.0.0.1
# *:80 => bound to 0.0.0.0

# 192.168.0.70
sudo watch netstat -antp
```

# Remote
![](./SSH%20remote%20port%20forwarding.png)
```bash
# Open port on remote host
# Using Remote Forwarding and an internet-facing machine,
# you can connect back to this internal service from anywhere
```
```bash
# Setting SSH server(70)
# For security reasons, Remote Forwarding by default only binds to localhost on the SSH Server
nano /etc/ssh/sshd_config # GatewayPorts yes
sudo service ssh restart

# Parrot Command
ssh administrator@192.168.0.70 -R *:8008:192.168.0.24:80

# Connect
Parrot web connect to  192.168.0.70:8008

# Observe connection status across machines
sudo netstat -antp | grep :80
```
Windows
```bash
plink.exe -ssh kali@172.27.234.2 -R 445:172.25.170:445
```

# Dynamic(proxychains)
![](./SSH%20dynamic%20port%20forwarding.png)
```bash
# Parrot
whereis proxychains
cat /usr/bin/proxychains
# Proxy Server <===> ProxyChains
# Tracing dynamic linking reveals proxychains works via dynamic API Hooking (LD_PRELOAD)
# Inspecting the source code shows it hooks connect() via LD_PRELOAD
# Therefore, if an application's networking code does not use connect(), proxychains cannot proxy it
sudo nano /etc/proxychains.conf
# sock4 127.0.0.1 9095
# sock5 127.0.0.1 9095

# Any traffic sent to .18:9050 (Host A) will be forwarded out through .70 (Host B)
ssh -D 9050 administrator@192.168.0.70
ssh -nNTCD 9050 administrator@192.168.0.70

# Connect
proxychains curl 192.168.0.24
# |S-chain|-<>-127.0.0.1:9050-<><>-192.168.0.24:80-<><>-OK
proxychains firefox # 192.168.0.24
```


# Local Double Pivot
![](./SSH%20local%20Double%20Pivot.png)
```bash
# -J : Specify a Jump Host to reach the target host
```
```bash
ssh -J administrator@192.168.0.70 administrator@192.168.0.10 -L *:80:192.168.0.24:80
# 192.168.0.70 : administrator / Infinit3
# 192.168.0.10 : administrator / toor


# Parrot web connect localhost:8888

# Observe connection status on 192.168.0.24
# Connections from 192.168.0.10 can be seen
sudo netstat -antp
```