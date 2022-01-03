---
title: "Driver"
author: "Me"
date: "January 03, 2022"
output: html_document
---

# Driver

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: Easy (?/10)</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Windows</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_driver_desc.png){: height="300px" width = 320px"}
</div>

**Ports/services exploited:** ?\\
**Tools:** autorecon?\\
**Techniques:** ?\\
**Keywords:** HP MultiFonction Printer

**TL;DR**: ?


## 1. Services enumeration
{:style="color:DarkRed; font-size: 170%;"}

Let's start this box by launching an *autorecon* scan:

````
sudo autorecon 10.10.11.105
`````

See *autorecon*'s official documentation or <a href="/_walkthroughs/Horizontall">Horizontall</a> for an explanation of the results. 
Here's the *nmap* output (performed by *autorecon*): 

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_driver_nmap.png)
</div>



## 2. Gaining a foothold
{:style="color:DarkRed; font-size: 170%;"}

We'll start by browsing to *10.10.11.106*:

<div class="img_container">
![site]({{https://jsom1.github.io/}}/_images/htb_driver_site.png)
</div>

By searching "MFP Firmware Update Center" on the internet, we learn that it stands for HP **MultiFonction Printer**. 
We can look at the documentation to see that the default credentials are *admin/admin*:

<div class="img_container">
![default creds]({{https://jsom1.github.io/}}/_images/htb_driver_creds.png)
</div>

So, let's see if the site uses the default credentials:

<div class="img_container">
![site]({{https://jsom1.github.io/}}/_images/htb_driver_site2.png)
</div>

And it does. Apparently, they conduct tests MFPs firmware updates and/or drivers. The only working button is *Firmware Updates* and it takes us to the following page:

<div class="img_container">
![site]({{https://jsom1.github.io/}}/_images/htb_driver_site3.png)
</div>

So, this is where we can upload a file. This latter is then supposedly being reviewed and tested.
There are 4 Printer models to chose from: *HTB DesignJet*, *HTB Ecotank*, *HTB Laserjet Pro* and HTB Mono*. It's too early to say if that matters, but it's good to have that in mind.
Also, we notice in the URLs that the site is made of PHP scripts. This can be useful to search for *.php* file extensions specifically with *gobuster*, for example.



## 3. Vertical privilege escalation
{:style="color:DarkRed; font-size: 170%;"}


<div class="img_container">
![pwn]({{https://jsom1.github.io/}}/_images/htb_driver_pwn.png){: height="380px" width = 390px"}
</div>


<ins>**My thoughts**</ins>

Original box
Interesting to see that printers can present a security risk if not properly secured or updated.

<ins>**Fix the vulnerabilities**</ins>


