---
title: "Devzat"
author: "Me"
date: "December 13, 2021"
output: html_document
---

# Devzat

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: Medium (?/10)</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Linux</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_dz_desc.png){: height="300px" width = 320px"}
</div>

**Ports/services exploited:** ?\\
**Tools:** ?\\
**Techniques:** ?\\
**Keywords:** ?

**In a nutshell**: ?

## 1. Services enumeration
{:style="color:DarkRed; font-size: 170%;"}

Let's start by enumerating the services and their version that are running on the machine with nmap:

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_dz_nmap.png)
</div>

We see two instances of SSH and web server. I don't know what's SSH on port 8000 yet, but we'll probably have an answer soon. As usual, we will start by the web server as SSH is rarely if ever exploitable.

## 2. Gaining a foothold
{:style="color:DarkRed; font-size: 170%;"}

Nmap indicated *Did not follow redirect to http://devzat.htb*, indicating we might have to add the IP and hostname to our */etc/hosts*. 
We get the confirmation by browsing to *http://10.10.11.118*: we see the URL became *devzat.htb*, and the website was not found. We resolve this issue with the following command:

````
sudo echo "10.10.11.118 devzat.htb" >> /etc/hosts
`````

Now, the domain name should be mapped with the IP address and the page should render correctly:

<div class="img_container">
![site]({{https://jsom1.github.io/}}/_images/htb_dz_site1.png)
</div>

Apparently, it offers a way to chat with any device that has an SSH client. There are other interesting information on the page, such as:

- A username and email address (patrick@devzat.htb)
- Patrick did the site design using HTML5 UP
- The chat is developed in its own branch and aside from the stable release
- The app is located into one folder only
- "*The app is of high quality, coded by me personally. So it has to be good, trust me*". That's probably a note from Patrick, and it's more funny than useful. Patrick either is a very good developer or has a very high self-esteem.

There's also an instruction for testing the chat:

<div class="img_container">
![site2]({{https://jsom1.github.io/}}/_images/htb_dz_site2.png)
</div>

So this is what the second instance of SSH on port 8000 is. We see it's *SSH-2.0-Go*, which might be helpful later.\\
Let's try the command and see what happens:

````
ssh -l [username] devzat.htb -p 8000
`````
<div class="img_container">
![app test]({{https://jsom1.github.io/}}/_images/htb_dz_chat.png)
</div>

Oh, this is really cool! Unfortunately I'm alone and nobody is going to reply, but I wanted to input something to see what would happen. 
After exiting, I thought we could maybe issue some commands such as *ls* and connected back to the chat to try it:

<div class="img_container">
![app test2]({{https://jsom1.github.io/}}/_images/htb_dz_chat2.png)
</div>

This time, it's a little bit different, and we see the last connection as well as the chat history. 
I didn't see at first the *run /help to see what you can do*, so I issued a *ls* as intended. In return, we get a list of commands and a message indicating this is not a shell.
By running the */help* command, we discove the app is on Github (github.com/quackduck/devzat). Let's go there to see how it's organized:

<div class="img_container">
![github page]({{https://jsom1.github.io/}}/_images/htb_dz_gh.png)
</div>

The first thing I noticed here is that there are 3 branches, although I was expecting two. These are: *main*, *patch-1* and *v2*. We are currently on *main*, which is most likely the stable release. We see the app is mostly written in *Go* (98.8%) and in shell (1.2%). If we scroll down the page, we see...


Voir autres branches
inpsect scripts



## 3. Vertical privilege escalation
{:style="color:DarkRed; font-size: 170%;"}



<ins>**My thoughts**</ins>


<ins>**Fix the vulnerabilities**</ins>

