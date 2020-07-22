---
title: "Buff"
author: "Me"
date: "July 21, 2020"
output: html_document
---

# Buff

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: easy (2.9/10)</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Windows</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_buff_desc.png){: height="250px" width = "280px"}
</div>

<ins>**Lab configuration**</ins>

First, download VirtualBox and Kali (or Parrot). When the machine is imported in VirtualBox, chose *bridged adapter* in the Network tab to have access to the internet. Start and set up the machine as you like.

*Buff* is a retired box of Hack The Box, and it is necessary to get a **VIP access** in order to do it (10$/month). Then, it's super easy and convenient to connect to it. The first thing to do is to download the connection pack at <https://www.hackthebox.eu/home/htb/access>. Then, open a terminal (make sure you're in the same directory as your connection file (*yourname*.ovpn)) and type the following command:

~~~~
openvpn yourname.ovpn
~~~~~

Where *yourname* is your username on Hack The Box. 
If you're using **Kali 2020**, make sure to add **sudo** to the previous command.

When this is done, just look at the IP of the machine on HTB (Hack the Box). *Buff* is given the IP 10.10.10.198.\\
We're ready to start !

## 1. Scan the ports of the target
{:style="color:DarkRed; font-size: 170%;"}

Let's start by performing the usual nmap scan with the flags *-sV* to have a verbose output and *-sC* to enable the most common scripts scan.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_buff_nmap.png)
</div>

There's only a web server running on port 8080, so we don't have many choices!

## 2. Find and exploit vulnerabilities
{:style="color:DarkRed; font-size: 170%;"}

Let's see this web server on port 8080:

<div class="img_container">
![site web]({{https://jsom1.github.io/}}/_images/htb_buff_site.png)
</div>

It's about gym, and after browsing the different tabs, we see the site is made using Gym Management Software 1.0. There's also a login page, but it requires an email and a password. I've never tried bruteforcing credentials with emails, so let's check if there is an easier way with this Gym Management Software. I had never heard about it, and Googling the first letters immediately returns something about an exploit (<https://www.exploit-db.com/exploits/48506>).\\
From Exploit-DB, the verion 1.0 of this software (it's the one used here) suffers from an **unauthenticated file upload vulnerability** allowing remote attackers to gain remote code execution (**RCE**) on the web server by uploading a maliciously crafted PHP file that bypasses the image upload filters.\\
The exploit consists in the following steps:

<div class="img_container">
![exploit description]({{https://jsom1.github.io/}}/_images/htb_buff_exploit.png)
</div>

Down the page, there is the script that performs all the steps listed above.
Thus, we can simply copy the script, create a file on our desktop (touch *exploit.py*), open it and paste the code. The first lines are:

<div class="img_container">
![exploit code]({{https://jsom1.github.io/}}/_images/htb_buff_code.png)
</div>

I then tried to launch it with something like *python exploit.py*, and we get a message telling us the right syntax:

<div class="img_container">
![exploit launch]({{https://jsom1.github.io/}}/_images/htb_buff_launch.png)
</div>

It worked, we're in as Shaun. From the documentation, we can now communicate with the webshell using get requests.

<ins>**My thoughts**</ins>

