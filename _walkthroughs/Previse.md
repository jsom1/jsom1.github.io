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
At this point, we pretty much covered the possible attack vectors without success. Let's iterate over them again, but this time with other tools (such as *gobuster* instead of *dirb*), other wordlists, other options, other SQLi tests, and so on...\\
Since there is only http to focus on, we know there must be a vulnerability there and we will find it.

We saw the login page is a *php* script. Let's search for *.php* file extensions:

<div class="img_container">
![php dirb]({{https://jsom1.github.io/}}/_images/htb_prev_dirb2.png)
</div>

There are a few *.php* files, and some of them are accessible (code 200). Even though */accounts.php* isn't accessible from the browser, we can *wget* it on our machine to have a look at it:

````
sudo wget http://10.10.11.104/accounts.php
`````

Once downloaded, we can *cat* it. There's not much going on there, we just see it sends a post request to log in. The second script */config* is accessible but shows a blank page. The only interesting thing here is */nav.php*:

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

I really wanted to do this machine on my own but sadly I had to look at the forums because I was stuck. It appears we were on the right track though: we discovered */nav.php* which had a few links. People say we have to click on those links and intercept the requests on Burp to spot something special. Let's do this. We set up Burp once again, intercept the request when clicking on *ACCOUNTS* and send it to the repeater. This time, we indeed see a different content than the one we saw when *cat*ing the file:

<div class="img_container">
![Burp2]({{https://jsom1.github.io/}}/_images/htb_prev_burp2.png)
</div>

Apparently, we shoud be able to *render* the page in Burp but it generates an error for some reason. We can still see what we need to create an account by sending a POST request to */accounts.php*:

<div class="img_container">
![Test user creation]({{https://jsom1.github.io/}}/_images/htb_prev_testuser.png)
</div>

This doesn't seem to work, we get an invalid user/password when we try to log in. I'm not sure the POST request has the right syntax. I learned something basic that I didn't know: we can (it sounds obvious) also intercept the response on Burp and modify it:

<div class="img_container">
![intercept response to request]({{https://jsom1.github.io/}}/_images/htb_prev_req.png)
</div>

<div class="img_container">
![modified response]({{https://jsom1.github.io/}}/_images/htb_prev_repmod.png)
</div>

Here, we replaced *302* by *200 OK* and forwarded it. Then, we see it actually works:

<div class="img_container">
![Create acc]({{https://jsom1.github.io/}}/_images/htb_prev_accs.png)
</div>

We fill the form and create the user. Burp intercepts the request, and we see the correct syntax:

<div class="img_container">
![Post]({{https://jsom1.github.io/}}/_images/htb_prev_post.png)
</div>

We indeed didn't have the right form. We can forward the request, which successfully creates a user. We then browse to the login page and login. At this point, we can see the different tabs. We see in *Management menu* that "MySQL server is online and connected", "There are 2 registered admins" and "There is 1 upoaded file". Browsing to *Files*, we indeed see the uploaded file (by a user called newguy):

<div class="img_container">
![uploaded file]({{https://jsom1.github.io/}}/_images/htb_prev_uplfile.png)
</div>

We can download the file and look at its content:

<div class="img_container">
![zip file]({{https://jsom1.github.io/}}/_images/htb_prev_zip.png)
</div>

There are MySQL credentials in *config.php*:

````
<?php
function connectDB(){
    $host = 'localhost';
    $user = 'root';
    $passwd = 'mySQL_p@ssw0rd!:)';
    $db = 'previse';
    $mycon = new mysqli($host, $user, $passwd, $db);
    return $mycon;
}
?>
`````

On the website, there's a *LOG DATA* tab that appears when we hover over *Management Menu*. There, we can request log data. Within the returned file, we see ourselves (I see netpal) as well as the previously seen user, *m4lwhere*. So, we know have a password and two potential users. Even though the password is for MySQL, we can try it with SSH. Indeed, passwords are often reused. Unfortunately, it's not the case here.

After looking at the other files from the zip, there is something interesting in *logs.php*. This script uses the *exec()* function, and this reminds me of another box, BountyHunter, which used the python function *eval()*. This function was being run in root context and could be used to import the os module, allowing us to read or write files.\\
The present situation looks similar :

<div class="img_container">
![exec function]({{https://jsom1.github.io/}}/_images/htb_prev_exec.png)
</div>

This is the function that is being called when we request logs on the server. We see that the *delim* parameter is directly taken from the input field, without any kind of validation. That looks promising, as we could potentially POST a malicious command in this parameter, which would then by *executed* by the *exec()* function. Also, the message about the easier use of Python could be a clue.\\
Let's request a log file and intercept the request with Burp to see the syntax. We can send it to the repeater to modify it afterwards:

<div class="img_container">
![post syntax]({{https://jsom1.github.io/}}/_images/htb_prev_burp3.png)
</div>

Line 14 shows the selected delim which is directly passed to the *logs.php* script. By researching more information about the python *exec()* command, I found the exact same article I read for BountyHunter, and talks about command injection in Python using *eval()* and *exec()*. It appears it is the same idea as the previous time. After playing a little bit with the command, I came up with the following one to get a reverse shell:

````
````


## 3. Privilege escalation
{:style="color:DarkRed; font-size: 170%;"}



<ins>**My thoughts**</ins>


<ins>**Fix the vulnerabilities**</ins>

