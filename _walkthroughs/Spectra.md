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
**Tools:** wpscan, hydra\\
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
Nothing worked, we'll have to find another way. Now we know the CMS, we can use **wpscan** to scan the website for known vulnerabilities.\\
For a thorough scan, we'll provide the *enumerate* options *ap* (all plugins), *at* (all themes), *cb* (config backups), *dbe* (Db exports), and *u* (users). The command is the following:

````
wpscan --url spectra.htb/main --enumerate ap, at, cb, dbe, u
`````

The output is long and reveals a lot of interesting information, among which:

- Server: nginx/1.17.4
- X-Powered-By: PHP/5.6.40
- WordPress version 5.3.2
- WordPress theme in use: twentytwenty (v. 1.1, up to date)
- Other themes: twentynineteen (v1.4), twentyseventeen (v. 2.2)

There was no DB exports, no plugins and no config backups found. That's still a lot of things to investigate! Before spreading too much, let's simply try to bruteforce the credentials. Before using Hydra, we need to know the form of the request. To do so, we can open the developer mode on the page and submit random credentials. Then, we look at POST's request parameters:

<div class="img_container">
![POST request]({{https://jsom1.github.io/}}/_images/htb_spectra_post.png){: height="415px" width = 625px"}
</div>

The username is in a variable called *log*, the password is in *pwd*, there's a cookie in *testcookie: 1*, and there's the form *wp-submit: Log+In*. We must still look at the response from the server by clicking on the *Response* tab. This is because we will give the error message to Hydra so that it knows whether it has to keep trying or not. The message is the following: "Error: the password you entered for the username administrator is incorrect. Lost your password?".\\
With this information, we have everything we need for Hydra. The syntax is as follows:

```
sudo hydra -l administrator -P /usr/share/dirb/wordlists/small.txt spectra.htb http-post-form "/main/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&testcookie=1&login=Login:Lost your password?" -V -f
`````

I used a small wordlist here to see if it there was a low hanging fruit. Unfortunately, hydra found a false positive password, *websearch*.
Even though it didn't find a password, we know there's no limit to our attempts. I tried with another wordlist (*common.txt*) but hydra found a false positive once again. I'm not going to spend too much time with hydra because there are many other things to try and check, and bruteforcing is rather a last resort strategy.\\








<ins>**My thoughts**</ins>
