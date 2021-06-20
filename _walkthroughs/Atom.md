---
title: "Atom"
author: "Me"
date: "June 20, 2021"
output: html_document
---

# Atom

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: medium (?/10)</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Windows</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_knife_desc.png)
</div>

**Ports/services exploited:** -\\
**Tools:** -\\
**Techniques:** -\\
**Keywords:** -\\


## 1. Port scanning
{:style="color:DarkRed; font-size: 170%;"}

As usual, let's perform a nmap scan with the flags *-sV* to have a verbose output, *-O* to enable OS detection, and *-sC* to enable the most common scripts scan.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_atom_nmap1.png)
</div>
<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_atom_nmap2.png)
</div>

Well, that's a lot of information! The running services are:

- HTTP on port 80: running httpd 2.4.46 and PHP/7.3.27. The box *Knife* had a webserver running PHP/8.0.1-dev which was vulnerable to RCE. We'll check for this version as well.
- RPC (Remote Procedure Call) on port 135: a protocol that uses the client-server model in order to alllow one program to request service from a program on another computer without having to understand the details of that computer's network.
- HTTPS (HTP over TLS/SSL) on port 443
- SMB on port 445

Nmap also tried to guess the OS and thinks it could be Windows Server 2008 SP1 or R2 (85%) or Windows 7 (85%). Finally nmap scripts returned informations regarding SMB.\\
There are many things to go over, so let's begin!


## 2. Find and exploit vulnerabilities
{:style="color:DarkRed; font-size: 170%;"}

The easiest thing to do is to browse to the server to get more information about it:

<div class="img_container">
![website]({{https://jsom1.github.io/}}/_images/htb_atom_site1.png){: height="415px" width = 625px"}
</div>
<div class="img_container">
![website]({{https://jsom1.github.io/}}/_images/htb_atom_site2.png)
</div>

We learn from the page that Heed is a software company. They propose here a simple note taking application (v1.0.0) that can be downloaded (for Windows only).
There's also an email (MrR3boot@atom.htb (MrR3boot is the creator of this box)) and we see it was created with Codepen. Codepen is a development environment for front-end developers.

Before downloading the program, we'll use dirb to bruteforce directories:


<ins>**My thoughts**</ins>

Long time no Windows, maybe more frequet than Linux.
