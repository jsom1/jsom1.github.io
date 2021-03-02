---
title: "Delivery"
author: "Me"
date: "March 02, 2021"
output: html_document
---

# Delivery

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: easy (2.9/10)</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Linux</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_delivery_desc.png){: height="250px" width = "280px"}
</div>

From this walkthrough on, the lab configuration won't be covered anymore.

## 1. Port scanning
{:style="color:DarkRed; font-size: 170%;"}

As usual, let's perform a nmap scan with the flags *-sV* to have a verbose output, *-O* to enable OS detection, and *-sC* to enable the most common scripts scan.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_delivery_nmap.png)
</div>

We see SSH and a web server running. Since we don't have any credentials yet, let's leave ssh aside for now and focus on the web server.

## 2. Find and exploit vulnerabilities
{:style="color:DarkRed; font-size: 170%;"}

We see it's a nginx 1.14.2 web server. The version might be useful later, but for now we will browse to it to see what it's hosting:

<div class="img_container">
![site web]({{https://jsom1.github.io/}}/_images/htb_delivery_site.png){: height="280px" width = "415px"}
</div>

At the bottom of the page, we see this page was made with HTML5up. Before bruteforcing directories, let's look at the clickable buttons *helpdesk* and *contact us*.
When we click on the first one, we get the following message:

<div class="img_container">
![message helpdesk]({{https://jsom1.github.io/}}/_images/htb_delivery_mess1.png){: height="280px" width = "415px"}
</div>

The address is weird and doesn't contain the IP 10.10.10.222 anymore. Could that be somehow related to hostname resolution? Let's look at the other button:

<div class="img_container">
![message contact]({{https://jsom1.github.io/}}/_images/htb_delivery_mess2.png){: height="280px" width = "415px"}
</div>

Interesting, we see we need an email address to access a MatterMost server, which is a messaging software. It is similar to Slack and is often used as an internal chat in organizations.
So we might need to create a new email address. Finally, let's click on the MatterMost link:

<div class="img_container">
![message mattermost]({{https://jsom1.github.io/}}/_images/htb_delivery_mattermost.png){: height="280px" width = "415px"}
</div>

We have the same problem again... We also see it is running on port 8065. Before creating a new email and digging into this hostname problem, let's use dirbuster to discover potential interesting directories:



<ins>**My thoughts**</ins>

