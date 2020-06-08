---
title: "ServMon"
author: "Me"
date: "April 20, 2020"
output: html_document
---

# ServMon

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: easy (3.7/10)</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Windows</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_servmon_desc.png){: height="250px" width = "280px"}
</div>

<ins>**Lab configuration**</ins>

First, download VirtualBox and Kali (or Parrot). When the machine is imported in VirtualBox, chose *bridged adapter* in the Network tab to have access to the internet. Start and set up the machine as you like.

*ServMon* is a retired box of Hack The Box, and it is necessary to get a **VIP access** in order to do it (10$/month). Then, it's super easy and convenient to connect to it. The first thing to do is to download the connection pack at <https://www.hackthebox.eu/home/htb/access>. Then, open a terminal (make sure you're in the same directory as your connection file (*yourname*.ovpn)) and type the following command:

~~~~
openvpn yourname.ovpn
~~~~~

Where *yourname* is your username on Hack The Box. 
If you're using **Kali 2020**, make sure to add **sudo** to the previous command.

When this is done, just look at the IP of the machine on HTB (Hack the Box). *ServMon* is given the IP 10.10.10.184.\\
We're ready to start !

## 1. Scan the ports of the target
{:style="color:DarkRed; font-size: 170%;"}

Let's start by performing the usual nmap scan with the flags *-sV* to have a verbose output and *-sC* to enable the most common scripts scan.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_servmon_nmap.png)
</div>
<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_servmon_nmap2.png)
</div>

There are many opened ports, but we quickly see interesting information: 
anonymous FTP is allowed, so we will be able to connect to the target and search for potentiel files.\\
There are a few common ports and services, among which 2 were also opened in the other Windows machines:

- **FTP** on port 21
- **SSH** on port 22
- **HTTP** on port 80
- **NetBIOS** on port 139 (NetBIOS is mainly used by Microsoft to establish sessions between computers on a network)
- **Microsoft-DS SMB** on port 445, depending on what *dialect* SMB speaks, I know it could be vulnerable

There are also a few ports I have never seen so far:

- **tcpwrapped** on port 5666: according to the internet, this port is commonly used by a **Nagios** plugin (NRPE).
**Nagios** is an application to monitor systems and networks. NRPE stands for Nagios Remote Plugin Executor and allows to
remotely execute Nagios plugins on other Linux/Unix machines, allowing to remotely monitor machines metrics (disk usage, CPU looad, etc...).
NRPE can also communicate with Windows machines.
- **tcpwrapped** on port 6699: this port is usually used by **Napster**, an online music shop. More precisely, it is used to share mp3 files.
- **ssl/https-alt** on port 8443: this port is the default port that **Tomcat** use to open SSL text service. Tomcat is a free web container for servlets and JSP.
Unlike HTTPS on port 443, it is necessary to specify the port with Tomcat.

It looks like we have a lot of possibilities! However, I don't think we could go anywhere with SSH at this point.
I will start by looking at FTP and HTTP, but I suspect we will have to exploit SMB on port 445...
However, I don't want to start Metasploit two minutes after starting this box... There are many things to explor, so let's go through them and learn some stuff!
We will also search Exploit-DB for any exploit for Nagios, NRPE and Napster.

Let's go!

## 2. Find and exploit vulnerabilities
{:style="color:DarkRed; font-size: 170%;"}

<u>**FTP**</u>

As we saw in the result of the nmap scan, anonymous login is allowed, so let's see if we can find interesting files:

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_servmon_ftp.png)
</div>

Note that I just pressed *enter* when I was asked for the password. Once loged in, we see the folder Users, and within it, the two userss Nadine and Nathan.
Great, we already have two usernames. Let's look into those folders:

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_servmon_ftp2.png)
</div>

Nadine has a *Confidential.txt* file that can be downloaded on our Kali machine with the command *get*. 
The file will be saved in the folder in which we launched FTP. Let's also see if Nathan has interesting files:

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_servmon_ftp3.png)
</div>

He has a "Notes to do.txt" file, which we download the same way we did for the previous one.
Now, let's look at what they contain.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_servmon_files.png)
</div>

In Nadine's note, we see that she left the Passwords.txt file on Nathan's Desktop. This might be interesting if we can access it at some point.\\
In Nathan's note, there is a todo list showing that he still hasn't uploaded the passwords, removed public access to NVMS and placed the secret files in SharePoint.

I don't know what NVMS and SharePoint are, so let's search on the internet. I found 2 things for **NVMS**, which could be:

- *Non-critical Virtual Machines*
- *Latitude NVMS*, a network based video management software platform that allows for streamlined provisioning of client software wiith automatic updates.

Microsoft **SharePoint** allows the creation of websites and to stock, organize and share information. It only requires a web browser.

Let's keep this iformation in mind, but not focus on it at the moment. We already have interesting information, so let's move to another port!


<u>**SSH**</u>

We found two usernames with FTP, and it's worth trying to SSH with them. 
Unfortunately, I couln't get in this way (I tried a few passwords such as admin, Nadine, Nathan, 1234, etc...)

<u>**HTTP**</u>

We saw there is a web server running, so let's check what's there!

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_servmon_web.png)
</div>

Well, it looks like *NVMS* is the platform we talked about earlier.
In the todo list, Nathan mentionned to *remove NVMS public access*. I checked on the internet for public credentials but didn't find anything.\\
I also tried different combinations of usernames and passwords, to no avail. Anyways, I don't know if having access to this platform would help us... 
Let's use **dirbuster** to see if there is any other interesting page.
If it is not the case, we might try to bruteforce the password for Nathan or Nadine.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_servmon_nmap2.png)
</div>

Dirbuster found 2 directories... One is just for an icon, and the other one redirects us on the NVMS login screen.
It didn't find */Pages* though, because it is not in the *common.txt* wordlist. Let's run another dirbuster against *10.10.10.184/Pages* then.
Once again, it didn't find anything. To bruteforce a password, we would need to know the type of request being sent to the server.
I used **Burpsuite** to intercept the request and look at its parameters:

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_servmon_burp.png)
</div>

However, it says that the connection is closed and the only parameter is the cookie...
Maybe this is related to the *public access to NVMS* removal, and we can't access it anymore ? Let's try another port for the moment.

<u>**Microsoft-ds**</u>

The port 445 is used by recent versions of SMB (older versions used port 139, where SMB was running over NetBios).
We can use a scanner to see if we can find any vulnerability:

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_servmon_vuln.png)
</div>

Nothing. Let's look deeper into SMB by performing **SMB enumeration**. According to the internet, this is a must-have skill for any pentester. SMB is a protocol that stands for Server Message Block; it's used for sharing resources such as files, printers, and anything retreivable/made available by a server. SMB is natively installed on Windows, but not on Linux. To have it on Linux, it is necessary to install a Samba server.\\
Some sort of authentication will be in place, and there are some security flaws. First, the default credentials, easily guessable ones or even no authentication.
Then, the Samba server in itself is known to be vulnerable, especially if it is not patched.\\

There are many tools and techniques in SMB enumeration: nmblookup, nbtscan, SMBMap, Smbclient, Rpcclient, Nmap, Enum4linux, etc. We will try a few oof those here.

Since we're attacking a Windows machine, there is no Samba Server. If it was the case, we would have to determine its version. In the following commands, we use the Nmap scripting engine (NSE):

<div class="img_container">
![SMB enumeration]({{https://jsom1.github.io/}}/_images/htb_servmon_SMBE1.png)
</div>

<div class="img_container">
![SMB enumeration]({{https://jsom1.github.io/}}/_images/htb_servmon_SMBE2.png)
</div>

There is some information that might be useful later. We can go further with **smbmap**.

<div class="img_container">
![smbmap]({{https://jsom1.github.io/}}/_images/htb_servmon_smbmap.png)
</div>

That might work if anonymous login is allowed, but it is not the case here.


NEXT TIME :
- Retry exploit NRPE
- Chercher exploit SMB


<ins>**My thoughts**</ins>
