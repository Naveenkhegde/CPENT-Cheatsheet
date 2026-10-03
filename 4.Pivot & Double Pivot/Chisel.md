Chisel
===
🔙 [MENU README](./Pivot%20&%20Double%20Pivot.md)

> https://github.com/jpillora/chisel/releases

# Local
![](./Chisel%20Local.png)
> Server on the compromised/target machine
```bash
chisel server -p 443

chisel client <chisel_server>:443 <remote_addr>:445
```
Sock
```bash
chisel.exe server -p 12345

./chisel client 192.168.61.129:12345 socks
```

# Remote
![](./Chisel%20Remote.png)
> Server on your attack machine
```bash
chisel server -p 443 --reverse 

chisel client <chisel_server>:443 R:<remote_addr>:445

chisel client <chisel_server>:443 R:<port>:<remote_addr>:445
```
Sock
```bash
./chisel server -p 12345 --reverse

chisel.exe client 192.168.61.128:12345 R:socks
```
