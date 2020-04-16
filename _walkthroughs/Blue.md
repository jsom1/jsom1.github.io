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

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_blue_desc.png)
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

We see many open ports and interesting information: we have a host name (HARIS-PC) and an OS (Windows 7 Pro SP1). A script detected *smb-os*, running on port 445. Let's go through the open ports quickly:

- Port 135 and 4915? running **msrpc**: usually, msrpc is indeed on port 135. It's also called Microsoft EPMAP (End Point Mapper), and it's used to remotely manage services including DHCP server, DNS server and WINS. Apparently it's also running on ports 49152, 49153, 49154, 49155, 49156 and 49157.
- Port 139 running **netbios-ssn**: this port is used by old versions of SMB (file sharing) to work over NetBios.
- Port 445 running **microsoft-ds**: this port is used by recent versions of SMB.

Port 139 and 445 were also opened in **Lame**, where I provided more details about SMB. In *Lame*, I used an exploit against *Samba*, a *dialect* spoken by the SMB protocol. It was *exploit/multi/samba/usermap_script*. However, nmap didn't indicate that netbios-ssn is using Samba, so this exploit will likely fail.\\
Let's look at other possibilites we have.

## 2. Find and exploit vulnerabilities
{:style="color:DarkRed; font-size: 170%;"}

We know SMB is running on Windows 7 Pro SP1 on port 445, so let's see if we find any vulnerability:

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_blue_nmap2.png)
</div>

Apparently, ms17-010 is vulnerable to RCE (Remote Code Execution). Let's now search in Exploit-DB to see if there is any known exploit for this vulnerability.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_blue_srchsploit.png)
</div>

The 4th one should work on Windows 7, and we see **EternalBlue** in the results. The box probably gets its name from it, so it's a hint that we're on the right track. I had never heard of EternalBlue, and this makes me curious.\\
According to the internet, EternalBlue is an exploit developed by the NSA and revealed by the group of hackers *The Shadow Brokers* in April 2017.\\
This exploit uses a breach in the first version of SMB (SMBv1). The breach was patched in a Microsoft update in march 2017 (ms17-010).\\
On the one hand nmap says ms17-010 is vulnerable, on the other hand the internet says Microsoft patched the vulnerability in ms17-010... This is weird, but we'll see if it works. Maybe the version detected is wrong.\\
We start Metasploit with *msfconsole* and look if the exploit we found in Exploit-DB is available:

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_blue_srch.png)
</div>

Yes, the exploit is here. The description says it's for ms17-010, so it should work... Let's use it and look at the required options.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_blue_exp.png)
</div>

We have to set RHOSTS to 10.10.10.40, RPORT is already set to 445 and the target is set to any SP of Windows 7 by default. Perfect, we can try to use the exploit:

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_blue_win.png)
</div>

And it's a win! We don't have a nice shell, but we can execute commands (note that we could try to get one with the command *shell*). Let's see who we are:

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_blue_whoami.png)
</div>

We have system permissions, which is the highest level of permissions in Windows. We can now search for the flag using the commands *cd*, *dir* and *type*:

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_blue_cmd1.png)
</div>

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_blue_cmd2.png)
</div>

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_blue_cmd3.png)
</div>

This is it, we have the root flag!

<ins>**My thoughts**</ins>

Well, that was a straightforward machine. It's my third Windows box and it was very similar to the other ones. I had already seen SMB and *ms* vulnerabilities, but I didn't know about EternalBlue and ms17-010. This machine shows how important it is to frequently do updates, although we know systems are very often out of date. It is this laziness that allowed the well known ransomwares *WannaCry* and *NotPetya* to use this vulnerability and cause that much damage.\\
Overall, I feel like this box would be a very good first one for a total beginner.
