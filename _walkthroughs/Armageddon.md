---
title: "Armageddon"
author: "Me"
date: "June 24, 2021"
output: html_document
---

# Armageddon

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: easy (?/10)</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Linux</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_arma_desc.png){: height="415px" width = 625px"}
</div>

**Ports/services exploited:** HTTP, MySQL\\
**Tools:** Metasploit, JtR, MySQL\\
**Techniques:** Enumeration\\
**Keywords:** Drupal


## 1. Port scanning
{:style="color:DarkRed; font-size: 170%;"}

Let's enumerate the running services with nmap and the flags *-sV* to have a verbose output, *-O* to enable OS detection, and *-sC* to enable the most common scripts scan.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_arma_nmap.png)
</div>

There's only SSH and HTTP, so it's a pretty straightforward start: we'll inspect the web page.

## 2. Find and exploit vulnerabilities
{:style="color:DarkRed; font-size: 170%;"}

By browsing to the box' IP, we land on the following page:

<div class="img_container">
![website]({{https://jsom1.github.io/}}/_images/htb_arma_site.png){: height="415px" width = 625px"}
</div>

While we inspect its content, we can start dirb and nikto in the background:

<div class="img_container">
![dirb]({{https://jsom1.github.io/}}/_images/htb_arma_dirb.png)
</div>

<div class="img_container">
![nikto]({{https://jsom1.github.io/}}/_images/htb_arma_nikto1.png)
</div>

<div class="img_container">
![nikto]({{https://jsom1.github.io/}}/_images/htb_arma_nikto2.png)
</div>

There's a lot of information and leads to investigate. Some of those directories contain dozens of scripts. I'll just quickly go through them, looking for usernames, passwords, interesting functiobs/scripts or any useful information.

One thing that frequently comes out is *Drupal*. It is a CMS written in php and based on the internet, is excellent from a security standpoint. Its flexibility makes it different from other CMS; modularity is one of its core principles. 

By inspecting the web page after a login attempt (admin/admin), we see information sent back from the server in the request's headers. It is Drupal 7 and PHP/5.4.16. Even though it is self-declared secure, we can look if there is any existing exploit on Metasploit with the command *searchsploit drupal*:

<div class="img_container">
![searchsploit]({{https://jsom1.github.io/}}/_images/htb_arma_search.png)
</div>

Well, it might not be that secure after all. We see in particular a few exploits for Drupal 7 up to Drupal 8, and some of them contain the string "geddon" in them. That could be a trap, but since the box' name is Armageddon, we have to have a look at that. There are RCE exploits but they seem to require authentication, so let's look at the SQLis. Reding the source code of those exploits seems to indicate we need a *https* url, which we don't have. Let's search on the internet to see if we can find a concrete example of this vulnerability.\\
There is a ruby exploit that looks promising (https://github.com/dreadlocked/Drupalgeddon2). We can download or copy the code and create a script on our machine, for example *drupalgeddon2.rb*. I'm not showing the code here becaus it consists of a few hundreds line. Prior to running the script, it might be necessary to install the *highline* gem, as indicated in the troubleshooting part on the page:

````
sudo gem install highline
`````

Then we can run the script against the target:

<div class="img_container">
![Exploit]({{https://jsom1.github.io/}}/_images/htb_arma_exploit.png)
</div>

We get a shell! We can't do much however... Especially, we can't change directory. We can list the content in */var/www/html* and inspect it, but we could already do that from the browser... I tried downloading linpeas from my Kali with wget (command not found) and curl (connection refused/permission denied) but that doesn't work. It might be due to firewall rules...

Anyways, this shell isn't really useful. There were some other exploits, so let's start *msfconsole* and *search* for Drupal 7:

````
sudo msfconsole -q
search drupal 7
``````

Among some exploits we see, there is *drupal_drupalgeddon2*; from its name, it seems to be the one we tried "manually". Let's quickly try it on Metasploit to make sure it doesn't work:

````
use exploit/unix/webapp/drupal_drupalgeddon2
`````

We set the options correctly (with *SET* ...) and try it:

<div class="img_container">
![Exploit works]({{https://jsom1.github.io/}}/_images/htb_arma_exploitok.png)
</div>

This time we can change directories. I have no idea how the two exploits differ, but it doesn't really matter since it's working now. The difference between our shell and browsing to the files from the browser is that we can see the files' content with *cat* in our shell. If we try to see the content from the browser, the page is blank and we'd have to *wget* it to see the it. Therefore, it is not necessary to get this apache shell, but it's much more convenient for enumeration.

After going through several files, we finally find credentials in */var/www/html/sites/default/settings.php*:

<div class="img_container">
![Credentials]({{https://jsom1.github.io/}}/_images/htb_arma_creds.png)
</div>

The first thing I did was to try to log in in the CMS with those credentials, but it didn't work. However, they are supposed to be used to authenticate to a database... During enumeration, there were several references to MySQL and PgSQL. Let's try to connect to MySQL with the discovered credentials. I tried from within meterpreter, but it doesn't know the command *mysql*. Let's try to invoke a shell with the command *shell* and then spawn a bash shell:

<div class="img_container">
![shell]({{https://jsom1.github.io/}}/_images/htb_arma_shell.png)
</div>

We see the command didn't work, but we still get a basic shell. Let's retry to connect to MySQL:

<div class="img_container">
![sql]({{https://jsom1.github.io/}}/_images/htb_arma_sql.png)
</div>

We seem to be connected, but we don't see the command's output... In fact we sometimes do, the behavior is really weird. When we issue a command, we're disconnected from MySQL and have to reconnect. After playing a little bit with the commands, I think nothing happens when we issue a command, but it works after adding a *show;* as we see in the image below:

<div class="img_container">
![show]({{https://jsom1.github.io/}}/_images/htb_arma_show.png)
</div>

From the error message, there is a syntax error... The command somehow worked though, but we got disconnected... It appears we can pass commands to the *mysql* command thanks to the *-e* flag: for example, the equivalent command to see the databases is:

````
mysql -u drupaluser -p -e 'show databases;'
````` 

We can then list drupal's database tables with:

````
mysql -u drupaluser -p -D drupal -e 'show tables;'
``````

As usual, there is the *users* table and we can display its content with:

````
mysql -u drupaluser -p -D drupal -e 'select * from users;'
`````

The output of that last command shows a lot of information (many columns), so we can reduce it by keeping only the ones we want:

<div class="img_container">
![hash]({{https://jsom1.github.io/}}/_images/htb_arma_hash.png)
</div>

There we see the user *brucetherealadmin* and a hash. Note that there's also a line for myself, because I tried creating an account on the webpage (I didn't include this part in the writeup since it was useless).\\
I checked the hash type on Google and it appears to be *Drupal7*, which makes sense. Anyways, let's copy this hash into a file (I called it *hash.txt*) and use **JtR** (John the Ripper) to crack it:

<div class="img_container">
![cracked]({{https://jsom1.github.io/}}/_images/htb_arma_cracked.png)
</div>

JtR finds the plaintext password in a second! We can try to switch user with *su -l brucetherealadmin*. This results in a "System error". The next thing to try is obviously SSH:

<div class="img_container">
![user flag]({{https://jsom1.github.io/}}/_images/htb_arma_user.png)
</div>

SSH worked and we get the user flag! As "usual" (it's quite recent), the first thing I check is user permissions:

<div class="img_container">
![user perms]({{https://jsom1.github.io/}}/_images/htb_arma_sudol.png)
</div>

We see we can run the command */usr/bin/snap install* as root without providing a password.

<ins>**My thoughts**</ins>
 

 
 
