---
title: "Previse"
author: "Me"
date: "November 14, 2021"
output: html_document
---

# Previse

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: Easy (?/10)</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Linux</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_prev_desc.png){: height="415px" width = 625px"}
</div>

**Ports/services exploited:** 80/web application, ssh\\
**Tools:** dirb, hydra, nikto, gobuster, burp\\
**Techniques:** ?\\
**Keywords:** ?\\ 
**In a nutshell**: ?

## 1. Services enumeration
{:style="color:DarkRed; font-size: 170%;"}

We start with the usual *nmap* scan with the flags *-sV* to determine service/version info, *-sC* to run default scripts (equivalent to *--script=default*), and *-O* to enable OS detection.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_prev_nmap.png)
</div>

There are only 2 services running, ssh and a web server. From experience, I don't think OpenSSH 7.6p1 is vulnerable so we'll start with the web server.
SSH is most likely here to be used once we discover credentials on the web server.

## 2. Gaining a foothold
{:style="color:DarkRed; font-size: 170%;"}

When browsing to *10.10.11.104*, we land on the following page:

<div class="img_container">
![site]({{https://jsom1.github.io/}}/_images/htb_prev_site.png)
</div>

It's a simple php login page with a username at the bottom (M4LWHERE). If we click on the username, it brings us to another page:

<div class="img_container">
![site2]({{https://jsom1.github.io/}}/_images/htb_prev_site2.png)
</div>

We see in the URL that it is an *https* page, however *nmap* didn't reveal any *ssl/https* service. 
Moreover, "M4lwhere" is the box' creator and a member of HtB, so this page might simply be his/her own private blog. 
This is the reason why we will leave it aside for now and focus on the login page. 

While we inspect the login page, we can launch *dirb* in the background to find potential directories:

````
sudo dirb http://10.10.11.104
`````

The first thing we can do is to try obvious credentials such as admin/admin. Unfortunately, nothing I tried worked. 
Then, we can test the application to see if it is vulnerable to SQL injections or XSS (cross-site scripting). 
Let's start with XSS by inputting the following text into the username and password fields:

````
<script>alert('xss')</script>
`````

When refreshing the page, the apparition of a popup would indicate a XSS vulnerability. It is not the case here. 
We can also test for SQLi by inputting the following text:

````
admin' or 1=1;#
`````

Any weird error message could indicate a vulnerability to SQL injections, but it's not the case either.
Finally, we can try to bruteforce credentials with *hydra*. To do so, we need to know the form of the POST request as well as the server's response.
Those can be found by intercepting the request with *Burp*, or by *Inspecting* the page (right click -> Inspect element -> Network -> analyze POST request parameters).
More precisely, we need to know the parameter names (is it user, username, password, passwd, and so on), the eventual cookies and the error message when a login failed.
In this case, the *hydra* command is the following:

````
sudo hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.11.104 http-post-form "/login.php:username=^USER^&password=^PASS^:F=Invalid Username or Password:H=Cookie: PHPSESSID=nh5564crcqp4vdfs38c6l9r3cv" -V -f
``````

Sadly, *hydra* doesn't find any password for the username "admin". We could still try "m4lwhere" as username, or even use a wordlist for it.\\
At this point, we pretty much covered the possible attack vectors without success. Let's iterate over them again, but this time with other tools (such as *gobuster* instead of *dirb*), other wordlists, other optios, other SQLi tests, and so on...\\
Since there is only http to focus on, we know there must be a vulnerability there and we will find it.

We saw the login page is a *php* script. Let's search for *.php* file extensions:

<div class="img_container">
![php dirb]({{https://jsom1.github.io/}}/_images/htb_prev_dirb2.png)
</div>

There are a few *.php* files, and some of them are accessible (code 200). Even though */accounts.php* isn't accessible from the browser, we can *wget* it on our machine to have a look at it:

````
sudo wget http://10.10.11.104/accounts.php
`````

Once downloaded, we can *cat* it. I'm not showing the output here because there is nothing very interesting. The second script */config* is accessible but shows a blank page. The only interesting thing here is */nav.php*:

<div class="img_container">
![nav]({{https://jsom1.github.io/}}/_images/htb_prev_nav.png)
</div>

However, all those links bring us back to the */login.php* page... We'll now use *nikto* to complete our manual enumeration:

<div class="img_container">
![nikto]({{https://jsom1.github.io/}}/_images/htb_prev_nikto.png)
</div>

Nothing really stands out. Let's make sure there isn't another service running on a higher port by doing a full TCP scan:

````
sudo nmap -p 1-65535 10.10.11.104
`````

There are only http and ssh... We used *dirb* earlier to bruteforce directories, let's now try with *gobuster* and a different wordlist. We will also look for *.php* file extensions one again:

````
sudo gobuster dir -u http://10.10.11.104/ -w /usr/share/wordlists/dirb/big.txt
sudo gobuster dir -u http://10.10.11.104/ -w /usr/share/wordlists/dirb/big.txt -x php
`````

Gobuster reveals a few different directories such as */.htaccess* and */.htpasswd*, but nothing really useful...

I really wanted to do this machine on my own but sadly I had to look at the forums because I was stuck. It appears we were on the right track though: we discovered */nav.php* which had a few links. People say we have to click on those links and intercept the requests on Burp to spot something special. Let's do this:



## 3. Privilege escalation
{:style="color:DarkRed; font-size: 170%;"}



<ins>**My thoughts**</ins>


<ins>**Fix the vulnerabilities**</ins>

