---
title: "Spectra"
author: "Me"
date: "June 22, 2021"
output: html_document
---

# Spectra

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: medium (4.2/10)</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Unknown</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_spectra_desc.png)
</div>

**Ports/services exploited:** \\
**Tools:** wpscan, hydra\\
**Techniques:** \\
**Keywords:** \\


## 1. Port scanning
{:style="color:DarkRed; font-size: 170%;"}

We start with the usual nmap scan with the flags *-sV* to have a verbose output, *-O* to enable OS detection, and *-sC* to enable the most common scripts scan.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_spectra_nmap.png)
</div>

There's a MySQL instance, SSH and a web server running. We'll start by visiting this latter to gain more information about the target. We don't really have any other choice anyways. 
Note that the OS isn't specified in the box' description and nmap didn't find anything about it either.

## 2. Find and exploit vulnerabilities
{:style="color:DarkRed; font-size: 170%;"}

By browsing to the target's IP, we see the following page:

<div class="img_container">
![website]({{https://jsom1.github.io/}}/_images/htb_spectra_site.png){: height="415px" width = 625px"}
</div>

Jira is a software that was designed to help work management. It was originally designed as a bug and issue tracker, but is today a powerful work management tool for all kinds of uses cases.
It is notably the ticketing system used by Hack The Box.\\
There are two clickable links, but there's an error when we click on them: we see in the URL *spectra.htb/testing/index.php*. 
This is a hostname resolution issue, and we can easily resolve it by adding the host into our */etc/hosts* file:

````
echo "10.10.10.229 spectra.htb" >> /etc/hosts
``````

We can now refresh the page to see what it contains. The link *Test* returns the following message:

<div class="img_container">
![website2]({{https://jsom1.github.io/}}/_images/htb_spectra_site2.png){: height="415px" width = 625px"}
</div>

This is probably related to the MySQL database revealed by nmap. Let's leave that aside for the moment and look at the second link, *Software Issue Tracker*:

<div class="img_container">
![website3]({{https://jsom1.github.io/}}/_images/htb_spectra_site3.png){: height="415px" width = 625px"}
</div>

There are several links and interesting information on this page. First, we see the CMS used for the site is Wordpress. Secondly, when we click on *comment*, we see the following message:\\
*Hi, this is a comment. To get started with moderating, editing, and deleting comments, please visit the Comments screen in the dashboard. Commenter avatars come from Gravatar*.\\
Thidrly, we see we can also post comment (without publishing our email address). Maybe the website is vulnerable to SQL injections? We'll have to check.
Finally, there is a *Log in* link that brings us to the following page:

<div class="img_container">
![login]({{https://jsom1.github.io/}}/_images/htb_spectra_login.png){: height="415px" width = 625px"}
</div>

We saw on the main page the username "administrator", so we could try to bruteforce the login with hydra for example. We can try a few obvious passwords such as admin, 1234, administrator, and spectra for example.
Nothing worked, we'll have to find another way. Now we know the CMS, we can use **wpscan** to scan the website for known vulnerabilities.\\
For a thorough scan, we'll provide the *enumerate* options *ap* (all plugins), *at* (all themes), *cb* (config backups), *dbe* (Db exports), and *u* (users). The command is the following:

````
wpscan --url spectra.htb/main --enumerate ap,at,cb,dbe,u
`````

The output is long and reveals a lot of interesting information, among which:

- Server: nginx/1.17.4
- X-Powered-By: PHP/5.6.40
- WordPress version 5.4.2 (insecure, released on 2020-06-10)
- WordPress theme in use: twentytwenty v. 1.2 (out of date, latest version is 1.7)
- Other themes: twentynineteen (v1.5, out of date), twentyseventeen (v. 2.3, out of date)
- 1 user found: administrator

There was no DB exports, no plugins and no config backups found. That's still a lot of things to investigate! Before spreading too much, let's simply try to bruteforce the credentials. Before using Hydra, we need to know the form of the request. To do so, we can open the developer mode on the page and submit random credentials. Then, we look at POST's request parameters:

<div class="img_container">
![POST request]({{https://jsom1.github.io/}}/_images/htb_spectra_post.png){: height="415px" width = 625px"}
</div>

The username is in a variable called *log*, the password is in *pwd*, there's a cookie in *testcookie: 1*, and there's the form *wp-submit: Log+In*. We must still look at the response from the server by clicking on the *Response* tab. This is because we will give the error message to Hydra so that it knows whether it has to keep trying or not. The message is the following: "Error: the password you entered for the username administrator is incorrect. Lost your password?".\\
With this information, we have everything we need for Hydra. The syntax is as follows:

```
sudo hydra -l administrator -P /usr/share/dirb/wordlists/small.txt spectra.htb http-post-form "/main/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&testcookie=1&login=Login:Lost your password?" -V -f
`````

I used a small wordlist here to see if it there was a low hanging fruit. Unfortunately, hydra found a false positive password, *websearch*.
Even though it didn't find a password, we know there's no limit to our attempts. I tried with another wordlist (*common.txt*) but hydra found a false positive once again. I'm not going to spend too much time with hydra because there are many other things to try and check, and bruteforcing is rather a last resort strategy.\\
We can try an SQL injection in the Username field to bypass the authentication process. The idea is the following: when a legitimate user submits their username and password, the application queries the underlying database with those values. The SQL statement uses an *and* logical operator in the *where* clause. This is why the database will only return records that have a user with a given username and password. For example in our case, we know the username *administrator*. Let's imagine the password is 1234. When this persons submits their credentials, the SQL query looks like thee following:

````
SELECT * FROM users WHERE name = 'administrator' and password = '1234';
`````
Now, we can alter the query's logic by submitting a username like *administrator' or 1=1;#*, which translate as:

````
SELECT * FROM users WHERE name = 'administrator' or 1=1;
`````

This request can be read as "show me all columns and rows for users with a name of administrator **or** where one equals one. Since this last condition always evaluates to true, all rows will be returned. So this request would return all records in the users table, creating a "valid password check".\\
Sometimes, applications have functions that query the database and expect a single record. If they get more than one row, they generate an error. Ideally, we want to have the application's source code to see the type of queries and if those functions are present. Most of the time we don't have the source code, so we must proceed by trial and error. Let's try an injection by limiting the number of rows returned with the *LIMIT* statement. To do so, we input the following command in the username and/or password field:

`````
administrator' or 1=1 LIMIT 1;#
``````

After a few tries, here's what comes out: if we only put it in the username, ther's an error "the password field is empty". If we put it in both fields, it returns "Unknown username". Finally if we input "administrator" in the username and the command in the password, it returns "the password you entered or the username administrator is incorrect".\\
This can either mean that the application is not vulnerable to SQL injection, or that we don't have the right command. Generally, we can spot SQLi vulnerabilities by injecting a simple "'" in every possible form on the webpage and look at the response. If the application doesn't handle it well, it might indicate the presence of a SQLi vulnerability.\\
I did that on the website but it seems the application handles that well. Instead of processing manually by trial and error, we can also use an automated tool such a **SQLmap**. For example, we can test different injections against a parameter. In our case, we get the parameter *p* in the URL when we click on the comment. The URL becomes *spectra.htb/main/?p=1#comments*. The SQLmap commands is the following:

````
sqlmap -u http://spectra.htb/main/?p=1#comments
``````

As we see in the ouput below, the tools tested many different injections:

<div class="img_container">
![SQmap]({{https://jsom1.github.io/}}/_images/htb_spectra_sqlmap.png){: height="415px" width = 625px"}
</div>

SQLmap didn't find anything either, so SQLi is probably not what we're looking for.

From research on the net and in Metasploit, Nginx 1.17.4 seems to be vulnerable to RCE but I can't find more info about it.\\
Versions of PHP prior to 5.6.40 were subject to many vulnerabilities, but this current version seems to be alright.\\









<ins>**My thoughts**</ins>
