---
title: "Knife"
author: "Me"
date: "June 18, 2021"
output: html_document
---

# Knife

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: easy (?/10)</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Linux</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_cap_desc.png){: height="600px" width = 800px"}
</div>

**Ports/services exploited:** -\\
**Tools:** dirb, nikto\\
**Techniques:** -\\
**Keywords:** \\


## 1. Port scanning
{:style="color:DarkRed; font-size: 170%;"}

As usual, let's perform a nmap scan with the flags *-sV* to have a verbose output, *-O* to enable OS detection, and *-sC* to enable the most common scripts scan.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_knife_nmap.png)
</div>

There's only SSH and a web server running.

## 2. Find and exploit vulnerabilities
{:style="color:DarkRed; font-size: 170%;"}

We will start by browsing to the web page and see what it contains:

<div class="img_container">
![website]({{https://jsom1.github.io/}}/_images/htb_knife_site.png){: height="400px" width = 600px"}
</div>

There are a few tabs on the page, but none of them are clickable. The next obvious thing we want to do is bruteforce directories, and we'll use dirb for that:

<div class="img_container">
![dirb]({{https://jsom1.github.io/}}/_images/htb_knife_dirb.png)
</div>

Dirb didn't find anything interesting (we can't access */server-status*). It's not the first time nothing stands out rapidly, so we might want to try other wordlists and/or other tools.
Let's try with dirb and another wordlist with the following syntax:

````
sudo dirb http://10.10.10.242/ /usr/share/dirb/wordlists/big.txt
``````

That doesn't reveal anything either. We can try with another tool like wfuzz:

`````
sudo wfuzz --hc 404 -w /usr/share/wordlists/rockyou.txt http://10.10.242/FUZZ
``````

We filter the output so that it doesn't return 404 errors. The result seems to contain only false positives as we see in the image below:

<div class="img_container">
![wfuzz]({{https://jsom1.github.io/}}/_images/htb_knife_wfuzz.png)
</div>

It seems that words starting with either "#" or a digit (at least 1) work. I stopped the scan quicly since it would take a very long time and does't seem to be the right way to go.
It's odd that we still haven't anything to continue, but there are a few things we can still try. Let's perform a full TCP scan as well as a top 1000 ports UDP scan. The syntax is the following:

````
sudo nmap -sV -sC -p 1-65535 10.10.10.242
sudo nmap -sU 10.10.10.242
`````

Both those scans bring nothing... We saw the versions of Apache (httpd 2.4.41) and ssh (OpnSSH 8.2p1) in the output of nmap. We can search if there is any existing module in Metasploit for those services and versions:

````
searchsploit httpd | grep linux
searchsploit openSSH
`````

There are some existing exploits but they don't look promising: it's either not for the good version or not what we want (there are DoS, buffer overflows, ...).

<ins>**My thoughts**</ins>


