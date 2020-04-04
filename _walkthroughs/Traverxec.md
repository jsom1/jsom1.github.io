---
title: "Traverxec"
author: "Me"
date: "April 03, 2020"
output: html_document
---

# Traverxec

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: easy</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Linux</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<ins>**Lab configuration**</ins>

First, download VirtualBox and Kali (or Parrot). When the machine is imported in VirtualBox, chose *bridged adapter* in the Network tab to have access to the internet. Start and set up the machine as you like.

*Traverxec* is a retired box of Hack The Box, and it is necessary to get a **VIP access** in order to do it (10$/month). Then, it's super easy and convenient to connect to it. The first thing to do is to download the connection pack at <https://www.hackthebox.eu/home/htb/access>. Then, open a terminal (make sure you're in the same directory as your connection file (*yourname*.ovpn)) and type the following command:

~~~~
openvpn yourname.ovpn
~~~~~

Where *yourname* is your username on Hack The Box. 
If you're using **Kali 2020**, make sure to add **sudo** to the previous command.

When this is done, just look at the IP of the machine on HTB (Hack the Box). *Traverxec* is given the IP 10.10.10.165.\\
We're ready to start !

## 1. Scan the ports of the target
{:style="color:DarkRed; font-size: 170%;"}

Let's start by performing the usual nmap scan with the flags *-sV* to have a verbose output and *-sC* to enable the most common scripts scan.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_tx_nmap.png)
</div>

There's not a lot going on here, only 2 opened TCP ports and services:

- **SSH** on port 22 (OpenSSH 7.9p1): we will maybe be able to connect via SSH at some point.
- **HTTP** on port 80, with nostromo web server version 1.9.6 (also called *nhttpd*): nhttpd is an open-source web server.

Based on my little experience, I don't see how we could exploit SSH at this point (we could maybe try to brute-force it if we had a username, but it is not the case here).
So, let's have a look at the web server.

## 2. Find and exploit vulnerabilities
{:style="color:DarkRed; font-size: 170%;"}

I didn't know the web server *nostromo*, and I found interesting information about it when I googled it:

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_tx_nos1.png)
</div>

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_tx_nos2.png)
</div>

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_tx_nos3.png)
</div>

One of the first thing returned by Google is the fact that this web server is vulnerable to RCE.
There is a reference and link (*CVE-2019-16278*) where more information is given. But maybe the most interesting thing here is that the vulnerability also concerns the version 1.9.6, which is the one running here.\\
This might be something we could exploit. But before looking for this potential exploit, let's have a look at the website.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_tx_web.png)
</div>

There is a page with content, and we see "David White". This could be useful at some point, who knows.
If we scroll all the way down, we see the following:

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_tx_basic.png)
</div>

Apparently, the website is using the template *Basic* from *TemplateMag*. 
I have never seen TemplateMag, and I didn't find anything interesting with a quick search on the web. 
But once again, it could be a track for later.\\
The website doesn't seem to have any other useful information, so let's use **dirbuster** to look for directories.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_tx_dirb.png)
</div>

There are a few directories, but a first quick search didn't give anything. I let it run a moment to search inside those directories, but stopped as it was taking a lot of time.
If I had nothing else to try, I would have waited until the end of the search, but can also search Exploit-DB for a *nhttpd* exploit.
It this doesn't work, we will come back to dirbuster.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_tx_srchsploit.png)
</div>

The 4th one is the one we are interested in. It's written version 1.9.3 and the web server is running 1.9.6, but we saw earlier that this latter is also vulnerable.\\
The exploit is a **directory traversal remote command**: I don't know what it is, and as usual, I will search for it on the web.\\
According to the internet, a *directory traversal* consists in exploiting insufficient security validation of user-supplied input file names, such that characters representing *traverse to parent directory* are passed through to the file APIs.

Let's start Metasploit and look if it contains the exploit.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_tx_srch.png)
</div>

Perfect, the exploit is here. We can use it and look at the required options.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_tx_options.png)
</div>

We only have to set RHOSTS (remote host, here 10.10.10.161) and LHOST (local host, here 10.10.14.18. This value is my VPN IP that can be found with *ifconfig* command, under *tun0*).\\
The default payload is *cmd/unix/reverse_perl*. A reverse shell is when a target machine initiates a connection to a user (the attacker), and the attacker listens on a specified port.\\
Usually, it's the opposite. A user initiates a shell connection with a remote machine (a server). However, this is often not possible because of firewalls and this is why we use reverse shells.\\
Indeed, attacked servers usually only allow connections on specific ports (a web server accepts coonnections on ports 80 and 443).\\
Since firewalls usually don't limit outgoing connections, the idea is that we can establish a server on our machine and create a reverse connection.
Here, the reverse shell used is written in Perl.\\
Let's try the exploit without changing the payload.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_tx_exploit.png)
</div>

It worked, we can execute commands! We can try to get a shell with the command *shell*:

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_tx_shell.png)
</div>

We see that we're the user *www-data*. This user has limited privileges, and we now need to find a way to escalate them: it is time for **enumeration**.
Because we're on a Windows machine, we navigate with *cd* to change directory, *dir* to list the content of a directory and *type* to display the content of a file.

<ins>**My thoughts**</ins>

WIP
