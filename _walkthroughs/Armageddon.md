---
title: "Armageddon"
author: "Me"
date: "June 24, 2021"
output: html_document
---

# Armageddon

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: easy (?/10)</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Linux</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_arma_desc.png){: height="415px" width = 625px"}
</div>

**Ports/services exploited:** \\
**Tools:** \\
**Techniques:** \\
**Keywords:**


## 1. Port scanning
{:style="color:DarkRed; font-size: 170%;"}

Let's enumerate the running services with nmap and the flags *-sV* to have a verbose output, *-O* to enable OS detection, and *-sC* to enable the most common scripts scan.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_arma_nmap.png)
</div>

There's only SSH and HTTP, so it's a pretty straightforward start: we'll inspect the web page.

## 2. Find and exploit vulnerabilities
{:style="color:DarkRed; font-size: 170%;"}

By browsing to the box' IP, we land on the following page:

<div class="img_container">
![website]({{https://jsom1.github.io/}}/_images/htb_arma_site.png){: height="415px" width = 625px"}
</div>

While we inspect its content, we can start dirb and nikto in the background:

<div class="img_container">
![dirb]({{https://jsom1.github.io/}}/_images/htb_arma_dirb.png)
</div>

<div class="img_container">
![nikto]({{https://jsom1.github.io/}}/_images/htb_arma_nikto1.png)
</div>

<div class="img_container">
![nikto]({{https://jsom1.github.io/}}/_images/htb_arma_nikto2.png)
</div>

There's a lot of information and leads to investigate. Some of those directories contain dozens of scripts. I'll just quickly go through them, looking for usernames, passwords, interesting functiobs/scripts or any useful information.

One thing that frequently comes out is *Drupal*. It is a CMS written in php and based on the internet, is excellent from a security standpoint. Its flexibility makes it different from other CMS; modularity is one of its core principles. 

By inspecting the web page after a login attempt (admin/admin), we see information sent back from the server in the request's headers. It is Drupal 7 and PHP/5.4.16. Even though it is self-declared secure, we can look if there is any existing exploit on Metasploit with the command *searchsploit drupal*:

<div class="img_container">
![searchsploit]({{https://jsom1.github.io/}}/_images/htb_arma_search.png)
</div>

Well, it might not be that secure after all. We see in particular a few exploits for Drupal 7 up to Drupal 8, and some of them contain the string "geddon" in them. That could be a trap, but since the box' name is Armageddon, we have to have a look at that. There are RCE exploits but they seem to require authentication, so let's look at the SQLis. Reding the source code of those exploits seems to indicate we need a *https* url, which we don't have. Let's search on the internet to see if we can find a concrete example of this vulnerability.


<ins>**My thoughts**</ins>
 

 
 
