---
title: "Time"
author: "Me"
date: "March 07, 2021"
output: html_document
---

# Time

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: medium (?/10)</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Linux</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_time_desc.png){: height="250px" width = "280px"}
</div>

**Tools:** ?\\
**Techniques:** ?\\
**Keywords:** ?\\


## 1. Port scanning
{:style="color:DarkRed; font-size: 170%;"}

As usual, let's perform a nmap scan with the flags *-sV* to have a verbose output, *-O* to enable OS detection, and *-sC* to enable the most common scripts scan.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_time_nmap.png)
</div>

We see SSH and a web server running.

## 2. Find and exploit vulnerabilities
{:style="color:DarkRed; font-size: 170%;"}

While we have a look at the web page, let's launch *dirb* to look for potential interesting directories:

<div class="img_container">
![dirb]({{https://jsom1.github.io/}}/_images/htb_time_dirb.png){: height="320px" width = "550px"}
</div>

There are a few directories, but we cannot access any of them. Let's look at the main page:

<div class="img_container">
![site web]({{https://jsom1.github.io/}}/_images/htb_time_site.png){: height="320px" width = "550px"}
</div>




<ins>**My thoughts**</ins>

