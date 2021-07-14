---
title: "Explore"
author: "Me"
date: "July 14, 2021"
output: html_document
---

# Explore

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: easy (?/10)</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Android</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_explore_desc.png){: height="415px" width = 625px"}
</div>

**Ports/services exploited:** \\
**Tools:** \\
**Techniques:** \\
**Keywords:**  


## 1. Port scanning
{:style="color:DarkRed; font-size: 170%;"}

Let's start by adding the box' IP to our hosts file with the following command:

````
echo "10.10.10.247 explore.htb" >> /etc/hosts
`````

Then, we'll use *nmap* to detect what ports and services are running on the machine. 
As usual, we specify the flags *-sV* to have a verbose output, *-O* to enable OS detection, and *-sC* to enable the most common scripts scan:

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_explore_nmap.png)
</div>

There's not much going there, only ssh on port 2222 (it's usually on port 22) and freeciv on port 5555, which appears to be a video game. 
We will therefore look for ssh 2.0, Banana Studio and freeciv vulnerabilities.

## 2. Find and exploit vulnerabilities
{:style="color:DarkRed; font-size: 170%;"}

Most of the time, there is a web server running on HtB machines. It's not the case here, so we start by Googling and searching for potential exploits on Metasploit.

<div class="img_container">
![website]({{https://jsom1.github.io/}}/_images/htb_love_site.png)
</div>



<ins>**My thoughts**</ins>
 
First Android box, cool that it doesn't start with a web serv.
