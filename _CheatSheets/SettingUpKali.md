---
title: "Setting up Kali"
author: "Me"
date: "January 26, 2022"
output: html_document
---

# Setting up Kali

Once in a while, we may have to reinstall Kali and start from fresh. This cheatsheets aims to provide a basic working pentesting environment.

# Change hostname

Edit */etc/hosts* and add:

````
127.0.1.1    <hostName>
`````

Edit */etc/hostname* and add:

````
<hostName>
`````

# Change password

````
passwd
`````

# Change the keyboard

It is US by default, we can change it by right-clicking on the Desktop > Applications > Settings > keyboard > layout. 
There, we add a new keyboard layout and put it at the top of the list (or remove the unwanted one).

# Create a directory structure

In the home directory, create a directory for enumeration, exploits, privesc, BOFs, and so on...

# Install useful Firefox addons
On Firefox, open the Application menu > Add-ons and themes. In particular, we want to instal **FoxyProxy (standard)** and **Cookie-editor**. 
Foxyproxy is useful to simply turn on and off a proxy from the browser. Oncce installed, we add a proxy for Burp. Title: _Burp_, Proxy Type: _HTTP_, Proxy IP address: _127.0.0.1_, Port: _8080_.

# Install Burp

````
sudo apt install burpsuite
`````
Before using Burp, it is necessary to install Burp CA certificate in Firefox (or any other browser). Once the proxy listener is active and the browser has been configured to work with Burp (with FoxyProxy for example), go to *http://burpsuite* > click *CA Certificate* in the top right corner > open Firefox' preferences > Privacy and Security > Certificates > View certificates > Authorities > Import. Tick *This certificate can identify web sites*.

# Install seclists' wordlists

````
sudo apt install seclists
`````

# Install Gobuster

````
sudo apt install gobuster
`````

# Install Docker

````
sudo apt install docker.io
`````

# Create a file containing badchars (for buffer overflows)
````
badchars = (
"\x00\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10"
"\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20"
"\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30"
"\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
"\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50"
"\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60"
"\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70"
"\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80"
"\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90"
"\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0"
"\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0"
"\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0"
"\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0"
"\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0"
"\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0"
"\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff")
`````

# Install Wine
Useful to cross-compile exploits
````
sudo apt-get install wine
`````
Add the following lines into _/etc/apt/sources.list_:
````
deb http://old.kali.org/kali sana main non-free contrib
deb http://old.kali.org/kali moto main non-free contrib
````
Finally, install wine32 as well:
````
sudo apt-get install wine32
````


# Download privesc scripts
- Windows-privesc-check (https://github.com/pentestmonkey/windows-privesc-check):
````
sudo wget https://github.com/pentestmonkey/windows-privesc-check/blob/master/windows-privesc-check2.exe
sudo wget https://raw.githubusercontent.com/pentestmonkey/windows-privesc-check/master/windows_privesc_check.py
````

- Linux_privesc_check (http://pentestmonkey.net/tools/audit/unix-privesc-check): it has do be downloaded manually from the website. Then, we unzip it:
````
sudo tar -xf /home/kali/Downloads/unix-privesc-check-1.4.tar.gz
````

- LinPEAS
- WinPEAS










