---
title: "Cap"
author: "Me"
date: "June 14, 2021"
output: html_document
---

# Cap

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: easy (?/10)</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Linux</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_cap_desc.png){: height="300px" width = "400px"}
</div>

**Ports/services exploited:** ?\\
**Tools:** ?\\
**Techniques:** ?\\
**Keywords:** ?\\


## 1. Port scanning
{:style="color:DarkRed; font-size: 170%;"}

As usual, let's perform a nmap scan with the flags *-sV* to have a verbose output, *-O* to enable OS detection, and *-sC* to enable the most common scripts scan.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_cap_nmap.png)
</div>

Note that we already know the machine is running Linux, but sometimes we get useful information regarding the version. 
We see there's an FTP server, SSH and a web server running on the machine. Let's have a look at those services.

## 2. Find and exploit vulnerabilities
{:style="color:DarkRed; font-size: 170%;"}

As usual, we can leave SSH aside for the moment. We will probably use it later, once we discover some credentials. The main lead is most likely going to be the web server, but let's quickly check FTP first.
In particular, we want to make sure anonymous FTP isn't allowed. If it is the case, we might be able to connect anonymously and potentially retrieve useful data:

<div class="img_container">
![Anonymous FTP]({{https://jsom1.github.io/}}/_images/htb_cap_ftp.png)
</div>

Apparently, anonymous FTP isn't allowed (I tried with an empty password and a few other ones). I don't remember for sure, but I think nmap would have told us if it was allowed.\\
Let's now look at the web server. We browse at 10.10.10.245:

<div class="img_container">
![Web server]({{https://jsom1.github.io/}}/_images/htb_cap_web.png)
</div>

We see a dashboard with different statistics, and we're already logged in as Nathan. There is a menu in the upper left of the page with different links: Dashboard (this is where we are currently), Security Snapshot (5 Second PCAP + Analysis), IP Config and Network Status.
While we look at those pages, let's start dirbuster in the background:

````
sudo dirb http://10.10.10.245 -r
`````

Note that the *-r* flag prevents the script from being recursive. Let's look at "Network Status" for example:

<div class="img_container">
![Network Status]({{https://jsom1.github.io/}}/_images/htb_cap_network.png)
</div>

The output is somewhat similar to the one of the command *ps -aux* and lists the active connections (*ps -aux* lists the running processes). Dirbuster only found three directories: /data, /ip and /netstat.
Those 3 directories correspond to the different links in the menu.\\
It seems we have to find something on this web site. Before digging into it, let's quickly try to SSH as Nathan with passwords such as "admin", "1234", "nathan", and so on. We never know, it could be simpler than expected:

````
sudo ssh nathan@10.10.10.245
`````

Sadly the password isn't that obvious.


<ins>**My thoughts**</ins>




