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
![desc]({{https://jsom1.github.io/}}/_images/htb_arma_desc.png)
</div>

**Ports/services exploited:** \\
**Tools:** \\
**Techniques:** \\
**Keywords:** \\


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


<ins>**My thoughts**</ins>
 

 
 
