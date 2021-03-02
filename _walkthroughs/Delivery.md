---
title: "Delivery"
author: "Me"
date: "March 02, 2021"
output: html_document
---

# Delivery

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: easy (2.9/10)</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Linux</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_delivery_desc.png){: height="250px" width = "280px"}
</div>

From this walkthrough on, the lab configuration won't be covered anymore.

## 1. Port scanning
{:style="color:DarkRed; font-size: 170%;"}

As usual, let's perform a nmap scan with the flags *-sV* to have a verbose output, *-O* to enable OS detection, and *-sC* to enable the most common scripts scan.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_delivery_nmap.png)
</div>

We see SSH and a web server running. Since we don't have any credentials yet, let's leave ssh aside for now and focus on the web server.

## 2. Find and exploit vulnerabilities
{:style="color:DarkRed; font-size: 170%;"}

We see it's a nginx 1.14.2 web server. The version might be useful later, but for now we will browse to it to see what it's hosting:

<div class="img_container">
![site web]({{https://jsom1.github.io/}}/_images/htb_delivery_site.png){: height="280px" width = "415px"}
</div>

At the bottom of the page, we see this page was made with HTML5up. Before bruteforcing directories, let's look at the clickable buttons *helpdesk* and *contact us*.
When we click on the first one, we get the following message:

<div class="img_container">
![message helpdesk]({{https://jsom1.github.io/}}/_images/htb_delivery_mess1.png){: height="280px" width = "415px"}
</div>

The address is weird and doesn't contain the IP 10.10.10.222 anymore. Could that be somehow related to hostname resolution? Let's look at the other button:

<div class="img_container">
![message contact]({{https://jsom1.github.io/}}/_images/htb_delivery_mess2.png){: height="280px" width = "415px"}
</div>

Interesting, we see we need an email address to access a MatterMost server, which is a messaging software. It is similar to Slack and is often used as an internal chat in organizations.
So we might need to create a new email address. Finally, let's click on the MatterMost link:

<div class="img_container">
![message mattermost]({{https://jsom1.github.io/}}/_images/htb_delivery_mattermost.png){: height="280px" width = "415px"}
</div>

We have the same problem again... We also see it is running on port 8065. Before creating a new email and digging into this hostname problem, let's use dirbuster to discover potential interesting directories, using the *-r* flag to not search recursively:

<div class="img_container">
![dirb]({{https://jsom1.github.io/}}/_images/htb_delivery_dirb.png){: height="280px" width = "415px"}
</div>

Those directories don't look very promising, and we can't access any of them anyway.
Next, let's try to add some hostnames in our /etc/hosts file and see if that resolves the problem. We can simply append text to the file using the following syntax: *echo "some text" >> /etc/hosts*. Note that it might be necessary to give write permissions beforehand, which can be done with *sudo chmod 777 /etc/hosts* (it could be a good idea to get back to default permissions once we're done editing the file).

<div class="img_container">
![add hostname]({{https://jsom1.github.io/}}/_images/htb_delivery_hostname.png){: height="280px" width = "415px"}
</div>

After adding this hostname and refreshing the MatterMost page, we get the following:

<div class="img_container">
![MatterMost works]({{https://jsom1.github.io/}}/_images/htb_delivery_MMok.png){: height="280px" width = "415px"}
</div>

Note that we can also now browse to http://delivery.htb instead of http://10.10.10.222. Did that also resolve the problem for the *helpdesk* button? Sadly it didn't. Before signing in to MatterMost, We need to create an email address with the domain name "delivery.htb", and to do that, we must be able to use the *helpdesk* button. So, let's add *echo "10.10.10.222 helpdesk.delivery.htb" >> /etc/hosts* to /etc/hosts and try refreshing the page:

<div class="img_container">
![Helpdesk works]({{https://jsom1.github.io/}}/_images/htb_delivery_helpdesk.png){: height="280px" width = "415px"}
</div>

Great, we "fixed" the two buttons! We can now click on sign in. At this point, we can either sign in or create a new account. Of course, we try a few obvious credentials such as admin/admin, but we are unlucky here. We see in the URL that this login page is a PHP script, so let's see if there is a */phpmyadmin* login page by browsing to *helpdesk.delivery.htb/phpmyadmin*. However, it is not the case. Since we can authenticate, there must also be a database behind this application. By default, nmap scans the most popular 1000 ports only, so let's run a full TCP scan with the flag *-p 1-65535*. While the scan is running, we can try a few SQL injections in the "email or username" field to see if get a promising error message:

<div class="img_container">
![SQLi]({{https://jsom1.github.io/}}/_images/htb_delivery_sqli.png){: height="280px" width = "415px"}
</div>

The SQL query for a login has th syntax *select \* from users where name = 'name' and password = 'password';*. Therefore, our query is translated as *select \* from users where name = admin' or 1=1;#' and password = 'password';*. Because # is a comment marker in MySQL, it removes the rest of the statement. Thus, our query asks to show all records and row for users with a name of admin OR where 1 = 1. Because 1 = 1 always evaluates to true, this injection would return all rows in the users table. Unfortunately, the error message is simple "Access denied" and doesn't give any useful information about a potential database. Let's look at the scan result:

<div class="img_container">
![Full TCP scan]({{https://jsom1.github.io/}}/_images/htb_delivery_nmap.png){: height="280px" width = "415px"}
</div>
adm
We see something running on port 8065, but we already know that this is MatterMost, as we saw previously in the URL. So no trace of a database. We're left with the possibility of creating an account (there is also a button "I'm an agent - sign in here" that redirects us to another login page (osTicket), but there doesn't seem to be anything there).



<ins>**My thoughts**</ins>

