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

It appears to be an online app that beautify and validate json. Let's try it with a JSON POST request syntax:

<div class="img_container">
![beautify]({{https://jsom1.github.io/}}/_images/htb_time_beautify.png){: height="320px" width = "550px"}
</div>

It seems to work. Let's copy and paste the output and try to validate it:

<div class="img_container">
![validate]({{https://jsom1.github.io/}}/_images/htb_time_validate.png){: height="320px" width = "550px"}
</div>

For some reason it fails, but we don't really know what it's doing. By trying some other commands with the validate function (here a GET request), there is an error message that says "validation failed: Unhandled Java exception: com.fasterxml.jackson.core.JsonParseException: Unrecognized token 'GET': was expecting ('true', 'false' or 'null'".

<ins>**My thoughts**</ins>

