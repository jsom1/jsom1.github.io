---
title: "BoardLight"
author: "Me"
date: "June 10, 2024"
output: html_document
---

# BoardLight

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: Easy</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Linux</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

  
**Ports/services exploited:** 80/http\\
**Tools:** dirb, ffuf, Burp, Nikto\\
**Techniques:** \\
**Keywords:** MIME

**TL;DR**: The target is running an Apache web server which hosts a web site. The only thing we can do on the website is to fill a formular to request a call back.

## 1. Services enumeration
{:style="color:DarkRed; font-size: 170%;"}

I wrote a small shell script that runs a *nmap* scan, and if it finds a web server running, it then runs *dirb* to search for existing directories,
and *ffuf* for subdomains enumeration. The results of the nmap scan are the following (*sudo nmap -sV -sC 10.10.11.11*):

```
Starting Nmap 7.92 ( https://nmap.org ) at 2024-06-10 16:36 CEST
Nmap scan report for 10.10.11.11
Host is up (0.036s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 06:2d:3b:85:10:59:ff:73:66:27:7f:0e:ae:03:ea:f4 (RSA)
|   256 59:03:dc:52:87:3a:35:99:34:44:74:33:78:31:35:fb (ECDSA)
|_  256 ab:13:38:e4:3e:e0:24:b4:69:38:a9:63:82:38:dd:f4 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 38.08 seconds
```
There are 2 services running:

- SSH on port 22, which uses OpenSSH version 8.2p1 on Ubuntu Linux. Recent versions of SSH are usually secure, so we'll leave it aside for now and might come back to it later, when we have credentials or if we don't find anything else.
- HTTP on port 80, which is an Apache web server version 2.4.41. We see "Site doesn't have a title", but that doesn't matter. Let's browse to *10.10.11.11* and see what we find.

I added *10.10.11.11* to my */etc/hosts* file, so I can simply browse to *http://boardlight.htb*. We land on a page and see *"Welcome to BOARDLIGHT. Boardlight is a cybersecurity consulting firm spacializing in providing cutting-edge security solutions to protect your business from cyber threats"*.
The menu buttons (Home, About, What we do, and Contact us) are just anchors to sections on the same page. Most buttons, like the head icon which looks like a login button, the Youtube, Linkedin, Facebook buttons, do not work at all.
If we scroll down the page, there's a "contact us" form:

<div class="img_container">
![web]({{https://jsom1.github.io/}}/_images/htb_board_form.png){: height="400px" width = "500px"}
</div>

The fields of this formular are typical candidates for XSS, SQLi, SSTI, and other kind of vulnerabilities. 
Before trying to detect one of those, let's start a *dirb* scan to search for other existing directories, as well as an *ffuf* scan to enumerate potential subdomains (I realize afterwards I used the IP address in the commands, but the domain name would also work):

```
sudo dirb http://10.10.11.11 -r

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Mon Jun 10 17:41:58 2024
URL_BASE: http://10.10.11.11/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt
OPTION: Not Recursive

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.10.11.11/ ----
==> DIRECTORY: http://10.10.11.11/css/                                                                           
==> DIRECTORY: http://10.10.11.11/images/                                                                        
+ http://10.10.11.11/index.php (CODE:200|SIZE:15949)                                                             
==> DIRECTORY: http://10.10.11.11/js/                                                                            
+ http://10.10.11.11/server-status (CODE:403|SIZE:276)                                                           
                                                                                                                 
-----------------
END_TIME: Mon Jun 10 17:44:41 2024
DOWNLOADED: 4612 - FOUND: 2
```

*Dirb* found some unaccessible directories (403 error, "Forbidden. You don't have permission to access this resource.") such as */css*, */images*, */js*, and */server-status*. The only one we can access is */index.php*, and this is the main page we landed on.\\
So, nothing interesting here. To be sure, we can run a second directory scan with *ffuf* and a different wordlist, and also *ffuf* to search for subdomains:

```
sudo ffuf -w /usr/share/seclists/Discovery/Web-Content/big.txt -u http://10.10.11.11 -H "Host: 10.10.11.11/FUZZ" -fs 15949
sudo ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -u http://10.10.11.11 -H "Host: FUZZ.10.10.11.11"
```
In the first command, we filtered the responses having a size of 15949, because it was only false positives. The subdomain scan didn't reveal anything.

Let's have a look at the formular. We can start Burp, intercept the request and send it to the repeater:

<div class="img_container">
![burp]({{https://jsom1.github.io/}}/_images/htb_board_burp.png){: height="400px" width = "500px"}
</div>

This is weird, because we would expect a formular of that type to send a POST request, with the provided data in the request's body.\\
However, we see it's a GET request that doesn't contain the expected parameters. The reason could be that there is an error in code that sends the formular.\\
There could also be a JavaScript script which intercepts the request and transforms it into a GET request...\\
To check if the request was submitted, I checked if I received any confirmation email, but that was not the case.

Next, we can use *nikto* to detect vulnerabilities in the web server. It looks for server misconfigurations, vulnerabilities in the softwares, and so on... In the command below, *-h* is used to specified to host:

```
sudo nikto -h 10.10.11.11
```
And here's the result:

<div class="img_container">
![nikto]({{https://jsom1.github.io/}}/_images/htb_board_nikto.png){: height="400px" width = "500px"}
</div>

We see the web server is Apache version 2.4.41, which we already knew from the *nmap* scan. The following security headers are absent:

- X-Frame-Options: its absence could make the site vulnerable to clickjacking
- X-XSS-Protection: the fact it is not defined could make the site vulnerable to XSS (cross-site scripting). Indeed, web browsers can use this header to detect and prevent XSS attacks. An XSS attack consists in injecting malicious scripts into web pages that are being viewed by other people (such as a comment section, for example). If this header is defined, it can ask the browser to block the loading of the page. Because it is not defined here, the browser might not have it by default.
- X-Content-Type-Options: the fact it is not defined could allow a user agent to transform the site content into another MIME (Multipurpose Internet Mail Extensions) type than the one specified. When the header is not defined, it means that the browser can try to guess the content's MIME type. In some cases, that could allow an attacker to execute malicious scripts.

Finally, we also see the server responds with junk HTTP methods, and this is why *ffuf* returned so many false positives.\\
Unfortunataly, nothing really helps us here. Since the form doesn't seem to be the way, we must have missed something during enumeration... Let's try again with *gobuster* and *ffuff*, with different wordlists.







## 2. Gaining a foothold
{:style="color:DarkRed; font-size: 170%;"}



<div class="img_container">
![webpage]({{https://jsom1.github.io/}}/_images/htb_headless_web.png){: height="400px" width = "500px"}
</div>



The command that worked comes from <a href="https://pswalia2u.medium.com/exploiting-xss-stealing-cookies-csrf-2325ec03136e" target="_blank">this article</a>:



## 3. Vertical privilege escalation
{:style="color:DarkRed; font-size: 170%;"}


<div class="img_container">
![pwn]({{https://jsom1.github.io/}}/_images/htb_headless_pwn.png){: height="400px" width = "420px"}
</div>


<ins>**My thoughts**</ins>


<ins>**Fix the vulnerabilities**</ins>

