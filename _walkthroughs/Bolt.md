---
title: "Bolt"
author: "Me"
date: "February 11, 2022"
output: html_document
---

# Bolt

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: Medium</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Linux</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_bolt_desc.png){: height="300px" width = 320px"}
</div>

**Ports/services exploited:** 80/http\\
**Tools:** \\
**Techniques:** \\
**Keywords:** 

**TL;DR**: 



## 1. Services enumeration
{:style="color:DarkRed; font-size: 170%;"}

As usual, we scan open ports with nmap:

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_bolt_nmap.png)
</div>

We see SSH running on port 22, and two web servers on port 80 (http) and 443 (https). We will focus our attention on the web servers as this version of SSH isn't vulnerbable.

## 2. Gaining a foothold

Let's see the http web page:

<div class="img_container">
![site]({{https://jsom1.github.io/}}/_images/htb_bolt_site.png){: height="300px" width = 320px"}
</div>

The first thing we see is "*Administration using Admin LTE*". From the internet, it is a *fully responsive administration template, based on Bootstrap 4.6 framework and the JS/jQuery plugin*. Before we do anything else, we can start a *dirb* scan:

````
sudo dirb http://10.10.11.114 -r
`````
<div class="img_container">
![dirb]({{https://jsom1.github.io/}}/_images/htb_bolt_dirb.png)
</div>

We see the directories more or less match the tabs of the main page. By scrolling down the page, we see three people from the team:

<div class="img_container">
![people]({{https://jsom1.github.io/}}/_images/htb_bolt_ppl.png){: height="300px" width = 320px"}
</div>

Those names may be used in a later brute force attempt. In the *Contact Us* tab, we see two more people (*Christoper M. Wood* and *Neil D. Sims*). Also, *Bonnie Green*'s name is now displayed as *Bonnie M. Green*. There are also two generic contact addresses, *support@bolt.htb* and *sales@bolt.htb*. From this information, we can already build a text files containing the following addresses:

<div class="img_container">
![mails]({{https://jsom1.github.io/}}/_images/htb_bolt_mails.png)
</div>

We can then look at the login page:

<div class="img_container">
![login]({{https://jsom1.github.io/}}/_images/htb_bolt_login.png)
</div>

We start by trying a few obvious credentials such as *admin*/*admin* and so on. Unfortunately, it doesn't work here. At this point, we could try to brute force the login with *Hydra*, but since we don't have a username (we have a list of potential usernames), let's see if there isn't a tidier way. We see we can create an account, so let's try to fill this form and use Burp to intercept the request:

<div class="img_container">
![burp]({{https://jsom1.github.io/}}/_images/htb_bolt_burp.png){: height="300px" width = 320px"}
</div>

The account creation fails due to an internal error... We see it's HTML 3.2 Final, but I didn't find anything interesting about that. Let's look at the other tabs. Under */download*, we see we can download a Docker image. After downloading it, I had to install docker (it's a new Kali installation):

````
sudo apt install docker.io
`````

The *.tar* image can be directly loaded:

````
sudo docker load -i image.tar
sudo docker image ls
`````
Finally, we can run it:

````
sudo docker run --platform linux/arm/v7 859e74798e6c
`````
However, it seems it requires docker login... Anyways, we don't necessarily need to run the image. We can get more information by inspecting the unzipped file:

````
sudo tar -xf image.tar
`````

<div class="img_container">
![docker image layers]({{https://jsom1.github.io/}}/_images/htb_bolt_layers.png)
</div>

We see the various layers the image is made of, and the manifest. The manifest gives information about the image; it lists the different layers and specifies the OS and architecture the image was built for. By inspecting the file, we see the configuration comes from the *859e747....json* file. This latter contains the following:

<div class="img_container">
![docker image config]({{https://jsom1.github.io/}}/_images/htb_bolt_config.png)
</div>

We see the image was built for an amd64 architecture, this is why I specified *--platform linux/arm/v7* when I tried to run the image. I first tried without this argument but got an error regarding the architecture. We also see commands that are executed within the container. For example, it moves a file to "/", runs the application, installs pip3, then uses it to install the requirements, and finally exposes the port 5005 of the container.\\
I was hoping we could find something interesting there, but it doesn't seem to be the case.

Let's have a look at the *https* web server on port 443. We browse to *https://bolt.htb*. We land on a blank page (*https://bolt.htb/auth/login?redirect=%2F*)... Let's start a *dirb* scan once again:

````
sudo dirb https://bolt.htb -r
`````

The scans reveal a */login* page, and browsing to it brings us on a *passbolt* login page where we can input an email. A quick search informs that *Passbolt* is an open source password manager designed for collaboration. You can securely generate, store, manage and monitor your team credentials*. Since we discovered a few email addresses earlier, let's try to use one of those here, for example sales@bolt.htb:

<div class="img_container">
![fail login]({{https://jsom1.github.io/}}/_images/htb_bolt_fail.png){: height="300px" width = 320px"}
</div>

Unsurprisingly it doesn't work, and we see we have to contact an administrator to request an invitation link. *Dirb* also revealed a */register* page, but browsing there generates an error "404 - Registration is not opened to public. Please contact your administrator". 




Passbolt (voit sur nmap) is an open source password manager designed for collaboration. You can securely generate, store, manage and monitor your team credentials.

Get access to all of your logins and passwords from multiple browsers or even your mobile phone. Create strong random passwords, thanks to the fully customizable password generator, and share them instantly with your team. 

Passbolt is built on top of an open API made for developers and agile teams. Passbolt is extensible and yet usable by everyone. No technical knowledge is required to use it.

voir https....bolt.htb/passbolt/passbolt_api -> home / login on peut mettre mail. Check sur site principal les personnes et faire custom listes avec (voir format (atbolt.htb) mail dans contact)

## 3...

## 4. Vertical privilege escalation
{:style="color:DarkRed; font-size: 170%;"}



<ins>**My thoughts**</ins>


<ins>**Fix the vulnerabilities**</ins>

