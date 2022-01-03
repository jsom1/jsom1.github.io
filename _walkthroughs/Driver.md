---
title: "Driver"
author: "Me"
date: "January 03, 2022"
output: html_document
---

# Driver

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: Easy</p>
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
Here's the *nmap* output, performed by *autorecon*. The script took 1.5 hour to run! 

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_driver_nmap.png)
</div>

The following services are running:

- Microsoft IIS 10.0 web server on port 80
- Microsoft End Point Mapper (EPMAP), also known as MS-RPC, on port 135. It is used to remotely manage services such as DHCP server or DNS
- Microsoft-DS Active Directory (AD, Windows shares) or Microsoft-DS SMB (file sharing) on port 445
- WinRM or Wsman on port 5985, for remote management
- Something on port 7680 that could be, according to the internet, used by WUDO (Windows Update Delivery Optimization) in Windows LANs.

Also, the target OS is likely to be Microsoft Windows Server 2008 R2 at 90% or Windows 10 at 85%. It's not shown in *nmap*'s output above, but it also ran a few scripts such as *smb-os-discovery*, *smb-security-mode*, and so on. There was nothing interesting though.


## 2. Gaining a foothold
{:style="color:DarkRed; font-size: 170%;"}

Let's go through each directory *autorecon* created for those services, starting with http (*cat /results/10.10.11.106/scans/tcp80/tcp_80_http_nmap.txt*).\\
*Autorecon* performed an additional targeted *nmap* scan for port 80. This latter would inform us if it found any web vulnerability such as XSS or CSRF. it's not the case here, nothing stands out. It also performed directory bruteforcing with *feroxbuster* and different wordlists, but there's nothing interesting either. 

Next, we'll browse to *10.10.11.106* to have a look at that web page:

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
2nd time I use autorecon, looks great but ton of information - easy to get lost. 

<ins>**Fix the vulnerabilities**</ins>


