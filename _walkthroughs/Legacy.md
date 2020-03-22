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
Like in the *Lame* walkthrough, we see the ports 139 and 445 which are used by the SMB (Service Message Block) protocol. This protocol allows resource sharing (files and printers) on networks with Windows machines.

In the *Lame* walkthrough, I used the exploit *usermap_script*. Instead of trying this here, let's do something else. Because it's a Windows machine, it might be vulnerable to the well known *ms08-067* vulnerability, which gives SYSTEM level privileges. Let's search more information about it.

## 2. Find and exploit vulnerabilities
{:style="color:DarkRed; font-size: 170%;"}

The command *searchsploit* looks for potential exploits:

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_leg_srch.png)
</div>

We see the exploit we're looking for, which is a buffer overflow vulnerability triggered by a crafted RPC (Remote Procedure Call). A remote procedure call is when a computer program causes a procedure to execute on another computer.\\
To be sure that the exploit works properly, let's try to obtain precise information on the target system by using the scanner *smb_version*:

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_leg_smb.png)
</div>

We see it's running the english version of Windows XP SP3. When we will set up the exploit, we'll be able to set the target. This is a good practice, because running an overflow against a wrong system could cause severe damage.\\
We are now ready to search and use the exploit.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_leg_msfsrch.png)
</div>

The command returned the exploit we were looking for. I then use the command *show options* to see which parameters are required: the only parameter we must set here is RHOST. We also see that by default, the target is set to *Automatic Targeting*. We will change that to use the specific version we found earlier with the scanner.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_leg_targ.png)
</div>

After setting RHOSTS to 10.10.10.4, I checked potential targets with the command *show targets* (only the first 15 are shown here). Oddly enough, there were many Windows XP SP3 versions, but not the english one... So, I didn't set any target and launched the exploit.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_leg_exploit.png)
</div>

Great, we got a meterpreter shell! Let's have a look at our privileges with the command *getuid*:

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_leg_perm.png)
</div>

We are on the SYSTEM account, which is the highest privilege level in the Windows user model. If we were unlucky and didn't get SYSTEM, we could try to run the command *getsystem*, which elevates the current privileges to SYSTEM.\\
Forrtunately we were lucky and at this point, the only remaining thing to do is to find the flags.\\
In Hack the Box CTFs on Windows machines, the flags can be found on the Desktop of the user. So, let's pop a shell and navigate on the system to find the flags.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_leg_shell.png)
</div>

Some basic commands on windows are: 

- *dir*: lists the directoy content 
- *cd*: used to change directory
- *type*: displays the content of text files

This is all we need to finish this box.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_leg_cd.png)
</div>

Finally, the two flags can be found in the following directories:

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_leg_root.png)
</div>

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_leg_user.png)
</div>

<ins>**My thoughts**</ins>

Just before trying this box, I was reading a book on Metasploit [a relative link](metasploit.md), and more precisely on how to attack a Windows system. I liked that this box allowed me to practice exactly what I just learned, exploiting the well known *ms08_067* vulnerability.\\
Also, I am not used to the Windows command prompt, and eventhough I only used 3 commands here, I got to read a list of them and familiriaze myself with it.
