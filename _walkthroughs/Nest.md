---
title: "Nest"
author: "Me"
date: "April 08, 2020"
output: html_document
---

# Nest

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

*Nest* is a retired box of Hack The Box, and it is necessary to get a **VIP access** in order to do it (10$/month). Then, it's super easy and convenient to connect to it. The first thing to do is to download the connection pack at <https://www.hackthebox.eu/home/htb/access>. Then, open a terminal (make sure you're in the same directory as your connection file (*yourname*.ovpn)) and type the following command:

~~~~
openvpn yourname.ovpn
~~~~~

Where *yourname* is your username on Hack The Box. 
If you're using **Kali 2020**, make sure to add **sudo** to the previous command.

When this is done, just look at the IP of the machine on HTB (Hack the Box). *Nest* is given the IP 10.10.10.178.\\
We're ready to start !

## 1. Scan the ports of the target
{:style="color:DarkRed; font-size: 170%;"}

Let's start by making sure we can communicate with the target (with a *ping*) and perform the usual nmap scan.
We are using the flags *-sV* to have a verbose output and *-sC* to enable the most common scripts scan.

<div class="img_container">
![nmap error]({{https://jsom1.github.io/}}/_images/htb_nest_nmap.png)
</div>

It says that the host seems down. However, we know it's up since the ping passed. Let's use the flag *-Pn* as it is advised.

<div class="img_container">
![nmap -Pn]({{https://jsom1.github.io/}}/_images/htb_nest_nmap2.png)
</div>

There is only one TCP port opened:

- Port 445 with **Microsoft-DS**: this port was also opened in Lame, Legacy and blue and is used by the protocol *SMB* for file sharing.\\
Depending on the version of SMB, we know that it could be vulnerable. Unfortunately, the scan couldn't detect the version.

We can try to use a script (*vuln*) to see if it finds anything vulnerable:

<div class="img_container">
![nmap vuln]({{https://jsom1.github.io/}}/_images/htb_nest_vuln.png)
</div>

There are errors mentionning **Samba** (a *dialect* spoken by SMB) and *ms10-054* as well as *ms10-061*.
For the port 445, there is an error so we won't have more information on SMB this way.
However, we can use an auxiliary module of Metasploit to try to detect it:

<div class="img_container">
![smb scan]({{https://jsom1.github.io/}}/_images/htb_nest_smbscan.png)
</div>

Well, it really seems there is no way to have more information about it.
Let's try to scan **UDP** ports to see if there is anything there:

<div class="img_container">
![nmap udp]({{https://jsom1.github.io/}}/_images/htb_nest_nmap3.png)
</div>

Note that:

- We have to use *sudo*, because an UDP scan requires nmap to access **ICMP** (Internet Control Message Protocol), which itself requires root access.
- The scan takes way more time then a TCP scan, because it doesn't work the same way as a TCP scan (it doesn't use a three-way handshake). 
This is why I omitted to use *-sV* and *-sC*, and scanned only the 100 most common ports with the flag *-F*.

The scan found nothing... At this point, we could still try to throw random SMB exploits and hope it works, but this is not really a good practice.
Let's try a last scan with more ports (by default, nmap scans 1663 ports): we will go for the 65'535 ports:

<div class="img_container">
![nmap all ports]({{https://jsom1.github.io/}}/_images/htb_nest_nmap4.png)
</div>

Great, we found something else. So, we have 2 leads: SMB and an unknown service running on port 4386. Because ports in the range 4380-4388 are unassigned, this service could be anything.

## 2. Find and exploit vulnerabilities
{:style="color:DarkRed; font-size: 170%;"}

**Port 4386**\\
Here, I've been lucky. I didn't really know what to do with this port, so I tried to use **Telnet** (Terminal Network), a protocol allowing to communicate with a remote server.
At first, I wanted to use it to have more information about the target OS (this is what I did in Basic pentesting): by starting a telnet session on the wrong port (here 4386, Telnet is usually on port 23), the target sends an error message with information on its OS.
Surprinsingly, it didn't generate an error here.

<div class="img_container">
![telnet]({{https://jsom1.github.io/}}/_images/htb_nest_telnet.png)
</div>

Every command I tried returned an "Unrecognised command" error. We see something interesting though: "HQK Reporting Service V1.2". I have no idea what this is, and as usual, let's look at the internet!\\
Although I can't find what HQK stands for, all the links returned by Google are about "SQL Server Reporting Services RCE" (Remote Code Execution).
If we could get a working RCE, this would be our foothold into this box!



<ins>**My thoughts**</ins>

This machine was awesome! I esecially liked the beginning. I've only done ~10 boxes so far, but usually it's always the same: a simple tcp scan reveals interesting ports, and very often port 80 (http) is among them.
Then, we browse to the webpage, use dictionnary attacks, and so on... Here, we had to perform multiple scans to find something, and this thing wasn't http!



