---
title: "Legacy"
author: "Me"
date: "March 21, 2020"
output: html_document
---

# Legacy

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

*Legacy* is a retired box of Hack The Box, and it is necessary to get a **VIP access** in order to do it (10$/month). Then, it's super easy and convenient to connect to it. The first thing to do is to download the connection pack at <https://www.hackthebox.eu/home/htb/access>. Then, open a terminal (make sure you're in the same directory as your connection file (*yourname*.ovpn)) and type the following command:

~~~~
openvpn yourname.ovpn
~~~~~

Where *yourname* is your username on Hack The Box. 
If you're using **Kali 2020**, make sure to add **sudo** to the previous command.

When this is done, just look at the IP of the machine on HTB (Hack the Box). *Legacy* is given the IP 10.10.10.4.\\
We're ready to start !

## 1. Scan the ports of the target
{:style="color:DarkRed; font-size: 170%;"}

I once again use nmap to scan the ports with the options *-sV* to have a more verbose output (more details), *-sC* to run a simple script scan using the default set of scripts, and *-oA* to store the results in normal, XML and grepable formats at once (so that I can easily come back to the scan results later).

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_leg_nmap.png)
</div>

There are 3 open TCP ports and services:

- **netbios-ssn** running on port 139
- **microsoft-ds** running on port 445
- **ms-wbt-server** running on port 3389

I don't know what *ms-wbt-server* is, so let's what we find about it on the internet. It appears that *ms-wbt-server* is a protocol that is used by Windows Remote Desktop on the port 3389. This port is vulnerable to DoS attacks against Windows NT Terminal Server. However, this is not the kind of attack we are interested in right now.\\
Like in the *Lame* walkthrough, we see the ports 139 and 445 which are used by the SMB (Service Message Block) protocol. This protocol allows resources sharing (files and printers) on networks with Windows machines.\\

In the *Lame* walkthrough, I used the exploit *usermap_script*. Instead of trying this here, let's do something else. Because it's a Windows machine, it might be vulnerable to the well known *ms08-067* vulnerability, which gives SYSTEM permissions. Let's search more information about it.

## 2. Find and exploit vulnerabilities
{:style="color:DarkRed; font-size: 170%;"}


<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_leg_srch.png)
</div>

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_leg_smb.png)
</div>

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_leg_msfsrch.png)
</div>

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_leg_targ.png)
</div>

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_leg_exploit.png)
</div>

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_leg_perm.png)
</div>

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_leg_shell.png)
</div>

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_leg_cd.png)
</div>

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_leg_root.png)
</div>

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_leg_user.png)
</div>
