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

It worked, we're in as Shaun. From the documentation, we can now communicate with the webshell using get requests. At this point, I thought I had to upload something on but didn't know what and how (there wasn't anything else to do). I had to ask for help...
We indeed have to upload 2 executables (maybe there are other ways to do it): **nc.exe** and **plink.exe**. I had already seen nc.exe somewhere but didn't know what it was, and had nevear heard of plink. So, let's look at those executables.

- **nc.exe**: is a software component of NetCat Netwoork Control Program. It's a tool for testing TCP/IP connections and ports. Some features are port scanning, port listening, file transferring, proxying and requesting HTTTP. Apparently, this file is often used by attackers as a backdoor. Its name can also be used to disguise malwares (the amount of CPU used by nc.exe is a good indicator; high numbers are suspicious). In our case, we will use it to **get a reverse shell**.
- **plink**: is a command line application, similar to ssh in Linux. It can be used to make an interactive connection to a remote server. Here, we will use it to **forward port**.

To upoad those files, we will start a HTTP server on our machine, and curl the files from the server as Shaun:

<div class="img_container">
![HTTP server]({{https://jsom1.github.io/}}/_images/htb_buff_server.png)
</div>

Note that since we didn't specify a port, it picked 8000 by default. Now, we can curl the file. However, it didn't work for me at first because the command couldn't find the file... To resolve this, we can see the tree of files by browsing to our address on the specified port (get the address with the command *sudo ifconfig tun0*):

<div class="img_container">
![Tree of files]({{https://jsom1.github.io/}}/_images/htb_buff_files.png)
</div>

And locate the files we need to curl:

<div class="img_container">
![Locate the files]({{https://jsom1.github.io/}}/_images/htb_buff_locate.png)
</div>

The problem is that we need to find nc.exe and plink.exe in the file system. I didn't find them, but we see that *Desktop* is a directory that is listed here. So, we can copy the files there:

<div class="img_container">
![copy files on Desktop]({{https://jsom1.github.io/}}/_images/htb_buff_copy.png)
</div>

Check it's here:

<div class="img_container">
![Check files are on the Desktop]({{https://jsom1.github.io/}}/_images/htb_buff_check.png)
</div>

Now, we should be able to curl those files on the server:

<div class="img_container">
![curl]({{https://jsom1.github.io/}}/_images/htb_buff_curl.png)
</div>

We see that the files are listed, and we can make sure they were well transfered (they would appear here even if the curl failed. That happened to me a few times: I got 404 error - file not found, yet they were listed by *dir*) by looking at the web server:

<div class="img_container">
![check web server]({{https://jsom1.github.io/}}/_images/htb_buff_transfer.png)
</div>

Great, it worked. We can stop the webserver (Ctrl + c) and start a listener. Indeed, we will now get a revershe shell. Start the listener:

<div class="img_container">
![Start listener]({{https://jsom1.github.io/}}/_images/htb_buff_listener.png)
</div>

Now, we will use the telepathy parameter to use nc.exe that will get us a command prompt. We enter the following command in the browser:

<div class="img_container">
![reverse]({{https://jsom1.github.io/}}/_images/htb_buff_reverse.png)
</div>

This should give us a command prompt. Let's look at our listener:

<div class="img_container">
![get the prompt]({{https://jsom1.github.io/}}/_images/htb_buff_shellok.png)
</div>

We see that our target connected to our machine, and we get our reverse shell. We can then navigate and search the flag:

<div class="img_container">
![search the flag]({{https://jsom1.github.io/}}/_images/htb_buff_toflag1.png)
</div>

<div class="img_container">
![search the flag]({{https://jsom1.github.io/}}/_images/htb_buff_toflag2.png)
</div>

And that's it for the user!

<ins>**My thoughts**</ins>

