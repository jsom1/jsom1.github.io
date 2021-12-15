---
title: "Forge"
author: "Me"
date: "December 13, 2021"
output: html_document
---

# Forge

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: Medium (?/10)</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Linux</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_forge_desc.png){: height="300px" width = 320px"}
</div>

**Ports/services exploited:** ?\\
**Tools:** wfuzz, ffuf\\
**Techniques:** subdomain enumeration, SSRF (server side request forgery)\\
**Keywords:** ?

**In a nutshell**: ?

## 1. Services enumeration
{:style="color:DarkRed; font-size: 170%;"}

As usual, we start by using *nmap* to discover the services running on the target. We use the flags *-sV* to determine service/version info, *-sC* to run default scripts (equivalent to *--script=default*), and *-O* to enable OS detection.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_forge_nmap.png)
</div>

We see FTP on port 21, but its state is *filtered*. That means that a firewall, filter, or other network obstacle is blocking the port (the packets are being filtered). Therefore, *nmap* cannot tell whether it's open or closed. Sometimes it's a little bit more verbose, displaying for example an ICMP error message such as "*destionation unreachable: communication administratively prohibited*". It's not the case here however.\\
Next, we see SSH on port 21. This service is almost always running on HtB machines, but it's rarely the right attack vector. It's rather used once we discover credentials. From memory, I don't think this version is vulnerable.\\
Finally, there's a Apache web server on port 80, and this is where we will start this box.
## 2. Gaining a foothold
{:style="color:DarkRed; font-size: 170%;"}

I don't do it systemically, but let's add the IP adress and hostname to our */etc/hosts* file:

````
sudo echo "10.10.11.111 forge.htb" >> /etc/hosts
`````

Then, we can browse to *http://forge.htb* to discover what's hosted on the web server. The default page consists of a gallery of images and doesn't seem to have anything useful. In the upper right corner of the page, there's a *Upload an image* button. When clicking on it, we're redirected to */upload* and are presented with the following options:

<div class="img_container">
![site]({{https://jsom1.github.io/}}/_images/htb_forge_site.png)
</div>

We can either upload a local file, or upload one from an url (in this case there's a blank space in which we can input an url). At this point, there are many things we can do: test for directory traversal vulnerabilities, try to upload a local file, and so on... While thinking about the possibilites, let's use *dirb* and *gobuster* to bruteforce directories. I'm using two different tools here to reduce the risk of missing something. Also, we see references to javascript when inspecting the page (right click on it -> Inspect element. In the Debugger tab, we see a script *main.js* in */static/js*. Therefore, we can ask *gobuster* to look for *.js* extensions specifically with the *-x* flag:

````
sudo dirb http://forge.htb
sudo gobuster dir -u http://forge.htb -w /usr/share/wordlists/dirb/big.txt
sudo gobuster dir -u http://forge.htb -w /usr/share/wordlists/dirb/big.txt -x js
`````

Note that by default, *dirb* uses the *common.txt* wordlist. So even though we used */dirb/big.txt* wordlist with gobuster, it's not exactly the same.\\
In this case, the result of the 3 scans are the same. The discovered directories are: */server-status* (code 403), */static* (code 301), */upload* (code 200) and */uploads* (code 301).

If we browse to */static*, we see 3 subdirectories: *css*, *images* (contains the images displayed on the default page) and *js* (contais the *main.js* script we saw by inspecting the page). Nothing really stands out here.

Next, let's try the functionnalities to upload a local file and one from an url, starting with the url. In the blank space, we input the following: *http://forge.htb/../../../../../* (if we don't start with *http* or *https*, we get an error message: "Invalid protocol! Supported protocols: http, https"). This results in another error message, "URL contains a blacklisted address!". It appears we can't use that IP...\\
Let's now try to upload a local file from our machine. In this example, I uploaded *linpeas.sh*:

<div class="img_container">
![upload]({{https://jsom1.github.io/}}/_images/htb_forge_upload.png)
</div>

We see the file was successfully uploaded. However, if we click on the link, we get a "Not Found" error. This is because we see it was uploaded in */uploads*, and not in */upload*. *Dirb* and *gobuster* found this directory, but indicated it returned the HTTP code 301 (Moved Permanently or redirect).\\
Next, let's try to add the *.jpg* extension to *linpeas.sh* and try again. The file is also successfully uploaded but this time, clicking the link shows another error message: "The image http://forge.htb/uploads/A1uTy4XXSvCfrHXvxeao cannot be displayed because it contains errors". A few seconds later, it's not being found anymore... Maybe the file is uploaded in */uploads* where it is subject to some tests, and if it fails, it gets removed?

As usual, I had to look at the forum at this point because I was stuck. Apparently, we're supposed to enumerate subdomains. I don't remember how we're supposed to do that, and a quick search mentionned that *wfuzz* is a good tool. I tried with the following syntax (which is supposed to work):

````
sudo wfuzz -c -f res.txt -Z -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --sc 200,202,204,301,302,307,4003 http://FUZZ.forge.htb
`````
Where *res.txt* is the file in which the results are stored, and *-sc* specifies the responses we want to keep. Finally, FUZZ in http://FUZZ.forge.htb is the word that will be replaced by the wordlist (seclists wordlists can be obtained with *sudo apt install seclists*) values. This should work, but I get the error "Unhandled exception: cannot add/remove handle - multi_perform() already running". I'm not the only to encounter this error, but there doesn't seem to be a quick and easy fix so I'll rather look at another tool.\\

The other tool that I tried is **ffuf** (sudo apt-get install ffuf) with the following syntax:

````
sudo ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u http://forge.htb/ -H "Host: FUZZ.forge.htb" -t 200 -fl 10
`````

This time the scan is successful and returns one subdomain: *admin.forge.htb*:

<div class="img_container">
![ffuf]({{https://jsom1.github.io/}}/_images/htb_forge_ffuf.png)
</div>

We add this subdomain to our */etc/hosts* (*sudo echo "10.10.11.111 admin.forge.htb" >> /etc/hosts*) file and browse to it:

<div class="img_container">
![subdomain]({{https://jsom1.github.io/}}/_images/htb_forge_subdom.png)
</div>

It seems we can't access it from a different IP address, but maybe we can use the file upload option and provide the subdomain address to access resources on *admin.forge.htb*... In this manner, it is *forge.htb* that would request *admin.forge.htb*.

By Googling "only localhost is allowed", the first string completion propistion is "only localhost is allowed bypass". By looking at this, the first links all mention SSRF (Server Side Request Forgery).\\
I've learned about it some time ago but don't really remember how it works nor what it is exactly, so here's a quick explanation from <https://owasp.org/www-community/attacks/Server_Side_Request_Forgery>:\\
*The target application may have functionality for importing data from a URL, publishing data to a URL or otherwise reading data from a URL that can be tampered with. The attacker modifies the calls to this functionality by supplying a completely different URL or by manipulating how URLs are built (path traversal etc.).\\
When the manipulated request goes to the server, the server-side code picks up the manipulated URL and tries to read data to the manipulated URL. By selecting target URLs the attacker may be able to read data from services that are not directly exposed on the internet. The attacker may also use this functionality to import untrusted data into code that expects to only read data from trusted sources, and as such circumvent input validation.*

On another page, there are a few indications about what we should try to do if we suspect a SSRF vulnerability, such as:

- Accessing to local files by using *file://* in the URL
- Trying to access local IP
- Local IP bypass
- DNS spoofing (domains pointing to 127.0.0.1)
- DNS Rebinding (resolves to an IP and next time to a local IP: http://rbnd.gl0.eu/dnsbin). This is useful to bypass configurations which resolves the given domain and check it against a white-list and then try to access it again (as it has to resolve the domain again a different IP can be served by the DNS).
- Accessing private content (filtered by IP or only accessible locally, like /admin path).





## 3. Horizontal privilege escalation
{:style="color:DarkRed; font-size: 170%;"}



## 3. Vertical privilege escalation
{:style="color:DarkRed; font-size: 170%;"}


<ins>**My thoughts**</ins>

I like this kind of box, where we know where we have to start (here on port 80).


<ins>**Fix the vulnerabilities**</ins>

