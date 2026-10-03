IOT
===
🔙 [MENU README](../README.md#note)

> Firmware Analysis / Extraction

# Enviroment
```bash
Use your own Linux VM with the firmware files
```
```bash
# Firmware filesystem decompression / unsquashfs tool
# Parrot
git clone https://github.com/useidel/sasquatch.git
sh build.sh
# kali
git clone --quiet --depth 1 --branch "master" https://github.com/devttys0/sasquatch
cd sasquatch
wget https://github.com/devttys0/sasquatch/pull/47.patch && patch -p1 < 47.patch && sudo ./build.sh

# Binary file analysis tool
binwalk
```

# Find Something
```bash
### Windows 
dir /s <FILE_NAME> 2> nul  #file
findstr /n /i /s <KEYWORD> *.* #content

### Linux 
find / -name <FILE_NAME> -ls 2> /dev/null  #file
find / -name <FILE_NAME> -print 2>/dev/null #file
find / -iname <FILE_NAME> 2> /dev/null  #file
grep -nir <KEYWORD> . #content
```


# Dlink_firmware.bin
Target
```bash
Find the Telnet password
```
```bash
binwalk -t -e Dlink_firmware.bin
cd _Dlink_firmware.bin.extracted
grep -nir telnet . 
# etc/scripts/misc/telnetd.sh
```
```bash
cat etc/scripts/misc/telnetd.sh

#!/bin/sh
image_sign=`cat /etc/config/image_sign`
TELNETD=`rgdb -g /sys/telnetd`
if [ "$TELNETD" = "true" ]; then
        echo "Start telnetd ..." > /dev/console
        if [ -f "/usr/sbin/login" ]; then
                lf=`rgdb -i -g /runtime/layout/lanif`
                telnetd -l "/usr/sbin/login" -u Alphanetworks:$image_sign -i $lf &
        else
                telnetd &
        fi
fi
```
```bash
find / -name image_sign -ls 2> /dev/null

cat /home/kali/Desktop/IOT/_Dlink_firmware.bin.extracted/squashfs-root/etc/config/image_sign

# wrgn23_dlwbr_dir300b
```

# D6000-V1.0.0.41_1.0.1.bin
Target
```bash
Find 6 sets of passwords: 5 plaintext, 1 hash/ciphertext
```
```bash
binwalk -t -e DVRF_v03.bin

# Plaintext
grep -nir passwd .

# Ciphertext / Hash
find . -name passwd -ls 2> /dev/null


cat squashfs-root/usr/etc/passwd
# admin:$1$$iC.dUsGpxNNJGeOm1dFio/:0:0:root:/:/bin/sh
```
Crack
```bash
john squashfs-root/usr/etc/passwd
# 1234
```


# encrypted.bin
```bash
binwalk -t encrypted.bin
# Nothing
```
Entropy
```bash
binwalk -E encrypted.bin 

# When Error
pip install --upgrade matplotlib
rm -r ~/.cache/matplotlib/
sudo apt remove python3-numpy
pip3 install numpy


# Entropy Range : 1~10
# Higher entropy indicates greater uncertainty/randomness and more chaotic file content.
# Lower entropy indicates less uncertainty, more structured/regular file content.

# Typically, an English text article has an entropy around 4-6
```
Dlink_firmware.bin
```bash
# Based on Entropy analysis:
# Generally, the file is either encrypted or compressed.
# However, we can observe that the middle and trailing sections are slightly less chaotic.
# Inspect the randomness of Dlink_firmware.bin in the middle and trailing sections:
hexdump -C -v Dlink_firmware.bin
# We can see this section consists purely of 00 bytes

```
encrypted.bin
```bash
# In other words:
# The middle and trailing sections of encrypted.bin are composed of similar repeating patterns
hexdump -C -v encrypted.bin


# Consider what scenario would produce this result:
# Under what operation does 00 yield an output that matches the key?
# XOR truth table:
  0 1 -> key
0 0 1 -> cipher
1
```
Steam XOR
```bash
# Dlink_firmware.bin: Highest frequency bytes in standard files
hexdump -C -v Dlink_firmware.bin | cut -d ' ' -f3-20 | sort | uniq -c | sort -nr | head -n 20
  2511 00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00
     4 00 00 00 01 00 00 00 00  00 00 3f 91 45 84 68 3b
     4 00 00 00 00 00 3f 91 45  84 68 3b de de a6 0f 23


# encrypted.bin: Highest frequency byte sequences in the file
hexdump -C -v encrypted.bin | cut -d ' ' -f3-20 | sort | uniq -c | sort -nr | head -n 20
    111 88 44 a2 d1 68 b4 5a 2d  88 44 a2 d1 68 b4 5a 2d
     19 88 45 fb 8b 95 83 20 75  d2 44 a2 d0 01 96 84 1b
     16 b6 74 af 5a 6a b4 5a 2d  88 45 fb 8b 95 83 20 75


# Get XOR Key
88 44 a2 d1 68 b4 5a 2d
```
xcat
```bash
> https://github.com/mstrand/xcat.git

chmod +x xcat.py

./xcat.py -x <xor_key> encrypted.bin > decrypted.bin

./xcat.py -x 8844a2d168b45a2d ../encrypted.bin > ../decrypted.bin

# Now we can analyze
binwalk -t decrypted.bin
```

# encrypted.bin - XORTool
CTF Quick XOR Cracking
```bash
# install
python -m pip install xortool
git clone https://github.com/hellman/xortool.git
cd xortool
poetry build
pip install dist/xortool*.whl


# xortool
xortool enctypted.bin
xortool cipher -c " "
xortool encrypted.bin -c 00
# -c: Specify most frequent character (usually space, or 00 for null bytes)
# -m: Specify maximum key length to try


# decrypted
xortool enctypted.bin -l 8 -c 00
binwalk -t -e xortool_out/0.out

# key
cat xortool_out/filename-key.csv
python -c "print(b'\x88D\xa2\xd1h\xb4Z-'.hex())"
```
