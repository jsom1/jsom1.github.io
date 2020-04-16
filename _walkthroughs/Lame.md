---
title: "Lame"
author: "Me"
date: "March 17, 2020"
output: html_document
---

# Lame

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: easy (2.6/10)</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Linux</p>
 </div>
 <div style="clear: both;"></div>
 </div> 
 
<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_lame_desc.png){: height="250px" width = "280px"}
</div>

<ins>**Lab configuration**</ins>

First, download VirtualBox and Kali (or Parrot). When the machine is imported in VirtualBox, chose *bridged adapter* in the Network tab to have access to the internet. Start and set up the machine as you like.

*Lame* is a retired box of Hack The Box, and it is necessary to get a **VIP access** in order to do it (10$/month). Then, it's super easy and convenient to connect to it. The first thing to do is to download the connection pack at <https://www.hackthebox.eu/home/htb/access>. Then, open a terminal (make sure you're in the same directory as your connection file (*yourname*.ovpn)) and type the following command:

~~~~
openvpn yourname.ovpn
~~~~~

Where *yourname* is your username on Hack The Box. 
If you're using **Kali 2020**, make sure to add **sudo** to the previous command.

When this is done, just look at the IP of the machine on HTB (Hack the Box). *Lame* is given the IP 10.10.10.3.
We're ready to start !

## 1. Scan the ports of the target
{:style="color:DarkRed; font-size: 170%;"}

I use nmap to scan the ports with the options *-sV* to have a more verbose output (more details), *-sC* to run a simple script scan using the default set of scripts, and *-oA* to store the results in normal, XML and grepable formats at once (so that I can easily come back to the scan results later).

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_lame_nmap.png)
</div>

There are 4 open ports and services:

- **FTP** running on port 21. I know that vsftpd 2.3.4 is vulnerable (from another box), and anonymous login is allowed. It will be interesting to see what we can get with that.
- **SSH** running on port 22. I have already tried to exploit OpenSSH in another box but didn't succeed. I'll probably leave it out at first.
- **Netbios-ssn** running on port 139. Apparently, the version of Samba is between 3.X and 4.X.
- **Netbios-ssn** running on port 445.

I don't know what Netbios-ssn is, so let's have a look on the internet.\\
The **SMB** protocol authorizes the communication between processes; it's the protocol that allows applications and services of computers on a network to communicate. The protocol *speaks different dialects*: for example, Common Internet File System (CIFS) is a specific implementation of SMB that allows to share files.\\ 
The dialect **Samba** is an implementation of Microsoft Active Directory that allows non-Windows machines to communicate with a Windows network.
In a nutshell, SMB is a protocol used to share files on a network and uses 2 TCP ports: 139 and 445.\\
**Port 139**: used by SMB to work over **NetBIOS** (NetBIOS is a transport layer that allows Windows machines to communicate on the same network).\\
**Port 445**: used by more recent versions of SMB (>Windows 2000) on top of a TCP stack, allowing SMB to work on the internet.\\
It seems that the older and more recent versions of SMB are running on this machine.

Finally, we also get information on the target OS, detected as being Unix/Linux (although we knew it from HTB). Great, we know where to look for vulnerabilities!


## 2. Find and exploit vulnerabilities
{:style="color:DarkRed; font-size: 170%;"}

Now, let's look at what we can find for FTP and Samba.

<ins>**FTP**</ins>

First, I tried to connect anonymously to see if there were any interesting files I could download on my machine.

<div class="img_container">
![ftp]({{https://jsom1.github.io/}}/_images/htb_lame_ftp.png)
</div>

Once connected (I used "guest" for the password, but I think anything would work), I listed files in the present directory (and others) but didn't find anything. I might have had higher chances if there was a web server running.\\
I remember exploiting vsftpd 2.3.4, but not in details, so let's look at the existing exploits:

<div class="img_container">
![searchsploit]({{https://jsom1.github.io/}}/_images/htb_lame_srchsploit.png)
</div>

There is only one exploit available, but it looks solid (backdoor command execution). Let's get more information on Metasploit. We launch it with the following command:

~~~
msfconsole
~~~~

In Metasploit, the commande *search* lists existing modules for the given vulnerability.

<div class="img_container">
![msfsearch]({{https://jsom1.github.io/}}/_images/htb_lame_msfsrch.png)
</div>

There are 4 different exploits. Our backdoor command execution is the 4th one (*exploit/unix/ftp/proftpd_133c_backdoor*). 
Let's launch this module (with the command *use*) and see what parameters it requires (with the command *options*):

<div class="img_container">
![msfsearch]({{https://jsom1.github.io/}}/_images/htb_lame_exploit.png)
</div>

We only have to set the RHOSTS parameter, which is the target's IP address (10.10.10.3). When I first tried the exploit, it failed as on the image above ("Exploit completed, but no session was created."). I looked on the internet and got 4 leads:

1. The receiving IP address is not the good one: the reason was that we're using a VPN to connect to HTB, and it might not use that address. I tried to set LHOSTS to my "VPN" address (the IP of *tun0* given with *ifconfig*. Mine is 10.10.14.9), but it didn't work.

2. It's not the right payload: I used the command *show payloads* to see what options I had, but there is only one. I still tried to set it manually, but it didn't change anything.

3. *Lame* is an unstable machine and/or try to restart Metasploit. I did it a few times, without success.

4. The machine has been patched. Well, there's not much I can do if this is the case.

Unfortunately, I decided to give up on the vulnerability and have a look at Samba.

<ins>**Netbios-ssn**</ins>

Let's look at potential exploits for Samba:

<div class="img_container">
![msfsearch]({{https://jsom1.github.io/}}/_images/htb_lame_samba.png)
</div>

There are many exploits available. Although all the exploits that are not for Unix/Linux can be filtered out, there are still a few remaining. I tried a heap overflow (out of the blue), but it didn't work. Then I searched on the internet and found that the *usermap_script* is a good one. So, let's launch it and look at the options. 

<div class="img_container">
![msfsearch]({{https://jsom1.github.io/}}/_images/htb_lame_samba_exp.png)
</div>

Once again we only have to provide the target's IP and launch the exploit. We also see that it used the port 139, so the older version of SMB. It could be interesting to change the port to 445 to see if it also works, but let's keep going with port 139. The exploit worked, we have an opened session! By the way, we see that the reverse TCP handler was made on the right address (10.10.14.9) and on the port 4444. Let's see if we can get a shell:

<div class="img_container">
![msfsearch]({{https://jsom1.github.io/}}/_images/htb_lame_pwn.png)
</div>

Perfect, we got one. In HTB boxes, there are 2 flags: on in the root folder, and one in home. I then just had to *cd* to those folders and *cat* the flags.

<ins>**My thoughts**</ins>

I liked it because it's the fist time I saw Samba and ports 139/445 and I learned something new. I am not sure wether exploiting vsftpd 2.3.4 was supposed to work or not. Overall, the box was easy and well adapted to my level since I only had to use Metasploit.
