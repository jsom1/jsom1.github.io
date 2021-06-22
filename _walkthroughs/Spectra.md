---
title: "Spectra"
author: "Me"
date: "June 22, 2021"
output: html_document
---

# Spectra

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: medium (4.2/10)</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Unknown</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_spectra_desc.png)
</div>

**Ports/services exploited:** \\
**Tools:** \\
**Techniques:** \\
**Keywords:** \\


## 1. Port scanning
{:style="color:DarkRed; font-size: 170%;"}

We start with the usual nmap scan with the flags *-sV* to have a verbose output, *-O* to enable OS detection, and *-sC* to enable the most common scripts scan.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_spectra_nmap.png)
</div>

There's a MySQL instance, SSH and a web server running. We'll start by visiting this latter to gain more information about the target. 
Note that the OS isn't specified in the box' description and nmap didn't find anything about it either.

## 2. Find and exploit vulnerabilities
{:style="color:DarkRed; font-size: 170%;"}

By browsing to the target's IP, we see the following page:

<div class="img_container">
![website]({{https://jsom1.github.io/}}/_images/htb_spectra_site.png){: height="415px" width = 625px"}
</div>

Jira is a software that was designed to help work management. It was originally designed as a bug and issue tracker, but is today a powerful work management tool for all kinds of uses cases.
It is notably the ticketing system used by Hack The Box.\\
There are two clickable links, but there's an error when we click on them: we see in the URL *spectra.htb/testing/index.php*. 
This is a hostname resolution issue, and we can easily resolve it by adding the host into our */etc/hosts* file:

````
echo "10.10.10.229 spectra.htb" >> /etc/hosts
``````

We can now refresh the page to see what it contains. The link *Test* returns the following message:

<div class="img_container">
![website2]({{https://jsom1.github.io/}}/_images/htb_spectra_site2.png){: height="415px" width = 625px"}
</div>

This is probably related to the MySQL database revealed by nmap. Let's leave that aside for the moment and look at the second link, *Software Issue Tracker*:

<div class="img_container">
![website3]({{https://jsom1.github.io/}}/_images/htb_spectra_site3.png){: height="415px" width = 625px"}
</div>

There are several links and interesting information on this page. First, we see the CMS used for the site is Wordpress. Secondly, when we click on *comment*, we see the following message:\\
*Hi, this is a comment. To get started with moderating, editing, and deleting comments, please visit the Comments screen in the dashboard. Commenter avatars come from Gravatar*.\\
Thidrly, we see we can also post comment (without publishing our email address). Maybe the website is vulnerable to SQL injections? We'll have to check.
Finally, there is a *Log in* link that brings us to the following page:

<div class="img_container">
![login]({{https://jsom1.github.io/}}/_images/htb_spectra_login.png){: height="415px" width = 625px"}
</div>

We saw on the main page the username "administrator", so we could try to bruteforce the login with hydra for example. We can try a few obvious passwords such as admin, 1234, administrator, and spectra for example.
Nothing worked, we'll have to find another way.








<ins>**My thoughts**</ins>
