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

Most of the time, there is a web server running on HtB machines. It's not the case here, so we start by Googling and searching for potential exploits on Metasploit. While we search, we can launch a full TCP scan in the background:

<div class="img_container">
![FULL tcp scan]({{https://jsom1.github.io/}}/_images/htb_explore_full.png)
</div>

We see two more opened ports with unknown services. I just browsed to those ports to see if it could be a web server, but it wasn't the case. We can try to get more information about those services by initiating a telnet or netcat session with the following syntax:

````
sudo telnet explore.htb 36199
`````

Once connected, we can issue a command. Obviously this will not work since telnet is usually running on port 23, but sometimes the server responds with an error message that includes more information about the service. It is not the case for any of the two services here.\\
Let's leave them aside for now and go back at port 5000. After a few Google searches, I found an interesting article (https://medium.com/@samsepio1/android4-vulnhub-writeup-3036f352640f). Apparently, this port is also used by **adb** (Android Debug Bridge). It's a tool that allows one to connect to an android device remotely and is generally used for developement. So why is it written freeciv? It appears that sometimes *nmap* indicates freeciv is running, whereas it is *adb* (there's an opened issue for that on nmap's github repo). In our case we don't know which service it is, but we can give *adb* a try. Following the article, we might be able to connect to the machine and get a shell.

We start by installing *adb* (warning: taking a snapshot before executing the next command is recommanded. I didn't and it pretty much broke my kali machine. After installation it restarted and I had no more GUI. I had to reinstall *xfce* and even after that my machine was altered):

````
sudo apt-get install adb
`````

We can then try to connect to it:

<div class="img_container">
![adb]({{https://jsom1.github.io/}}/_images/htb_explore_adb.png)
</div>

We see the daemon wasn't running and is being started on port 5037. I then tried to connect to it (*sudo adb connect explore.htb:5037*) but nothing happens and the connection times out... 


<ins>**My thoughts**</ins>
 
First Android box, cool that it doesn't start with a web serv.
