---
title: "Blue"
author: "Me"
date: "April 03, 2020"
output: html_document
---

# Blue

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: easy</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Windows</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<ins>**Lab configuration**</ins>


First, download VirtualBox and Kali (or Parrot). When the machine is imported in VirtualBox, chose *bridged adapter* in the Network tab to have access to the internet. Start and set up the machine as you like.

*Blue* is a retired box of Hack The Box, and it is necessary to get a **VIP access** in order to do it (10$/month). Then, it's super easy and convenient to connect to it. The first thing to do is to download the connection pack at <https://www.hackthebox.eu/home/htb/access>. Then, open a terminal (make sure you're in the same directory as your connection file (*yourname*.ovpn)) and type the following command:

~~~~
openvpn yourname.ovpn
~~~~~

Where *yourname* is your username on Hack The Box. 
If you're using **Kali 2020**, make sure to add **sudo** to the previous command.

When this is done, just look at the IP of the machine on HTB (Hack the Box). *Blue* is given the IP 10.10.10.40.\\
We're ready to start !

## 1. Scan the ports of the target
{:style="color:DarkRed; font-size: 170%;"}

Let's start by performing a usual nmap scan with the flags *-sV* to have a verbose output and *-sC* to enable the most common scripts scan.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_blue_nmap.png)
</div>



## 2. Find and exploit vulnerabilities
{:style="color:DarkRed; font-size: 170%;"}

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_blue_os.png)
</div>

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_blue_options.png)
</div>

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_blue_fail.png)
</div>

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_blue_nmap2.png)
</div>

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_blue_srchsploit.png)
</div>

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_blue_srch.png)
</div>

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_blue_exp.png)
</div>

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_blue_win.png)
</div>

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_blue_whoami.png)
</div>

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_blue_cmd1.png)
</div>

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_blue_cmd2.png)
</div>

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_blue_cmd3.png)
</div>


<ins>**My thoughts**</ins>

WIP
