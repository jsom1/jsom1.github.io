---
title: "Lame"
author: "Me"
date: "March 17, 2020"
output: html_document
---

# Lame

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: beginner</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**Objective**: read the flag</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

**Lab configuration**

WIP
This is a retired box of Hack The Box, and it is necessary to get a VIP access in order to do it.

## 1. Scan the ports of the target
{:style="color:DarkRed; font-size: 170%;"}

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_lame_nmap.png)
</div>


## 2. Find and exploit vulnerabilities
{:style="color:DarkRed; font-size: 170%;"}

**FTP**\\

<div class="img_container">
![ftp]({{https://jsom1.github.io/}}/_images/htb_lame_ftp.png)
</div>

<div class="img_container">
![searchsploit]({{https://jsom1.github.io/}}/_images/htb_lame_srchsploit.png)
</div>

There are 2 modules for this vulnerability. Let's launch Metasploit with the following command:
~~~
msfconsole
~~~~
In Metasploit, the line starts with *msf5*. Now, we can search more information on the modules:

<div class="img_container">
![msfsearch]({{https://jsom1.github.io/}}/_images/htb_lame_msfsrch.png)
</div>

We see 4 different exploits. We could try some of these, but there's one that seems to be what we're looking for: the 4th one, *exploit/unix/ftp/proftpd_133c_backdoor*. It's rated excellent, it's the right version, and it's a backdoor command execution.\\
Let's launch this exploit and see what parameters it requires (with the *options* command):

<div class="img_container">
![msfsearch]({{https://jsom1.github.io/}}/_images/htb_lame_exploit.png)
</div>


<div class="img_container">
![msfsearch]({{https://jsom1.github.io/}}/_images/htb_lame_samba.png)
</div>


<div class="img_container">
![msfsearch]({{https://jsom1.github.io/}}/_images/htb_lame_samba_exp.png)
</div>


<div class="img_container">
![msfsearch]({{https://jsom1.github.io/}}/_images/htb_lame_pwn.png)
</div>
