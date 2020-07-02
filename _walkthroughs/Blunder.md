---
title: "Blunder"
author: "Me"
date: "June 22, 2020"
output: html_document
---

# Blunder

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: easy (3.9/10)</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Linux</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_blunder_desc.png){: height="250px" width = "280px"}
</div>

<ins>**Lab configuration**</ins>

First, download VirtualBox and Kali (or Parrot). When the machine is imported in VirtualBox, chose *bridged adapter* in the Network tab to have access to the internet. Start and set up the machine as you like.

*Blunder* is a retired box of Hack The Box, and it is necessary to get a **VIP access** in order to do it (10$/month). Then, it's super easy and convenient to connect to it. The first thing to do is to download the connection pack at <https://www.hackthebox.eu/home/htb/access>. Then, open a terminal (make sure you're in the same directory as your connection file (*yourname*.ovpn)) and type the following command:

~~~~
openvpn yourname.ovpn
~~~~~

Where *yourname* is your username on Hack The Box. 
If you're using **Kali 2020**, make sure to add **sudo** to the previous command.

When this is done, just look at the IP of the machine on HTB (Hack the Box). *Blunder* is given the IP 10.10.10.191.\\
We're ready to start !

## 1. Scan the ports of the target
{:style="color:DarkRed; font-size: 170%;"}

Let's start by performing the usual nmap scan with the flags *-sV* to have a verbose output and *-sC* to enable the most common scripts scan.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_blunder_nmap.png)
</div>

We see that FTP is closed, so our only possibility is port 80.


## 2. Find and exploit vulnerabilities
{:style="color:DarkRed; font-size: 170%;"}

**<u>HTTP</u>**

We saw there is a web server running, so let's check what's there:

<div class="img_container">
![website]({{https://jsom1.github.io/}}/_images/htb_blunder_web.png)
</div>

The page talks about Stephen King, Stadia and USB. A quick look through these articles doesn't reveal anything interesting. We can use Gobuster to search for interesting directories.

<div class="img_container">
![Gobuster]({{https://jsom1.github.io/}}/_images/htb_blunder_gobuster.png)
</div>

The first 3 files look interesting, but we see a 403 error message, meaning we can't access them. Then, there's */admin* and */robots.txt*. This latter is a text file with instructions for search engine crawlers. The file says what directories crawlers cannot search, and it is the first file opened by crawlers when they visit the site. Here, the file contains the following instructions:

<div class="img_container">
![robots.txt]({{https://jsom1.github.io/}}/_images/htb_blunder_robots.png)
</div>

*User-agent* denotes the name of the crawler (which can be found in the Robots Database). Here, \* means that any crawler can access the site.\\
*Allow:* is used to specify what areas of the website can be visited by the crawler. Here, it can go anywhere. This file isn't particularly interesting in this case, but sometimes it lists juicy directories and files that we can inspect to gather information.\\
We also saw */admin*, so let's head there and see it:

<div class="img_container">
![login]({{https://jsom1.github.io/}}/_images/htb_blunder_login.png)
</div>

After trying a few obvious and common credentials, I thought I would use Hydra and try to bruteforce credentials. First, we must know how the request is sent and processed by the server. To do so, we can either use the web developer mode in Firefox and look at the parameters when we submit credentials, or we can use Burp suite.\\
To use Burp, first have to configure the Proxy correctly. In Firefox, go to *Preferences* and search for *proxy*. Open the *Settings...* and make sure that *Manual proxy configuration is selected*. Change *HTTP Proxy* to 127.0.0.1 and port to 80. The settings should look like the following:

<div class="img_container">
![Proxy settings]({{https://jsom1.github.io/}}/_images/htb_blunder_proxy.png)
</div>

When this is done, we can open Burp suite. In the tab *Proxy* and *Intercept*, make sure that *intercept is on*. We're ready to submit random credentials on the login screen and analyze what happens:

<div class="img_container">
![Burp]({{https://jsom1.github.io/}}/_images/htb_blunder_burp.png)
</div>

We see it's a POST request (no surprise), and we see the syntax for the username and password. Note that we also have information on the User-agent (for robots.txt). There is also a cookie and a tokenCSRF. At this point, we should be able to write the Hydra command.

<div class="img_container">
![Hydra]({{https://jsom1.github.io/}}/_images/htb_blunder_hydra.png)
</div>

It found something, but after trying the credentials on the login page, I was sad to see it didn't work. I tried many different commands with different wordlists for username and password (here we see we're not specifying the cookie), and Hydra always found wrong credentials (false positives).\\

I had to look at the forum to get some hints. Many people were talking about **fuzzing**, and I finally understood they were refering to **wfuzz**. Wfuzz is a tool I didn't know about, and is used to bruteforce web applications. After looking at the help (*wfuzz --help*), I fired the following command:

<div class="img_container">
![wfuzz]({{https://jsom1.github.io/}}/_images/htb_blunder_hydra.png)
</div>

Note that I had to try with different wordlists, and filtered the results so that it doesn't show 404 errors. We see some files that Gobuster found, but this time we get a new one: *todo*.\\
Back in the browser, we find the file by adding the *.txt* extension:

<div class="img_container">
![todo]({{https://jsom1.github.io/}}/_images/htb_blunder_todo.png)
</div>

We see that FTP was effectively turned off, but we also get a potential username. Also on the forums, people were talking about creating a custom wordlist to brute force the login. I then discovered another tool I didn't know about, **cewl**. This allows to generate a wordlist from a web page.

<ins>**My thoughts**</ins>
