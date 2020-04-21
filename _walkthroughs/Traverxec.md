---
title: "Traverxec"
author: "Me"
date: "April 03, 2020"
output: html_document
---

# Traverxec

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: easy (4.7/10)</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Linux</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_tx_desc.png){: height="250px" width = "280px"}
</div>

<ins>**Lab configuration**</ins>

First, download VirtualBox and Kali (or Parrot). When the machine is imported in VirtualBox, chose *bridged adapter* in the Network tab to have access to the internet. Start and set up the machine as you like.

*Traverxec* is a retired box of Hack The Box, and it is necessary to get a **VIP access** in order to do it (10$/month). Then, it's super easy and convenient to connect to it. The first thing to do is to download the connection pack at <https://www.hackthebox.eu/home/htb/access>. Then, open a terminal (make sure you're in the same directory as your connection file (*yourname*.ovpn)) and type the following command:

~~~~
openvpn yourname.ovpn
~~~~~

Where *yourname* is your username on Hack The Box. 
If you're using **Kali 2020**, make sure to add **sudo** to the previous command.

When this is done, just look at the IP of the machine on HTB (Hack the Box). *Traverxec* is given the IP 10.10.10.165.\\
We're ready to start !

## 1. Scan the ports of the target
{:style="color:DarkRed; font-size: 170%;"}

Let's start by performing the usual nmap scan with the flags *-sV* to have a verbose output and *-sC* to enable the most common scripts scan.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_tx_nmap.png)
</div>

There's not a lot going on here, only 2 opened TCP ports and services:

- **SSH** on port 22 (OpenSSH 7.9p1): we will maybe be able to connect via SSH at some point.
- **HTTP** on port 80, with nostromo web server version 1.9.6 (also called *nhttpd*): nhttpd is an open-source web server.

Based on my little experience, I don't see how we could exploit SSH at this point (we could maybe try to brute-force it if we had a username, but it is not the case here).
So, let's have a look at the web server.

## 2. Find and exploit vulnerabilities
{:style="color:DarkRed; font-size: 170%;"}

I didn't know the web server *nostromo*, and I found interesting information about it when I googled it:

<div class="img_container">
![web info nostromo]({{https://jsom1.github.io/}}/_images/htb_tx_nos1.png)
</div>

<div class="img_container">
![web info nostromo]({{https://jsom1.github.io/}}/_images/htb_tx_nos2.png)
</div>

<div class="img_container">
![web info nostromo]({{https://jsom1.github.io/}}/_images/htb_tx_nos3.png)
</div>

One of the first thing returned by Google is the fact that this web server is vulnerable to RCE.
There is a reference and link (*CVE-2019-16278*) where more information is given. But maybe the most interesting thing here is that the vulnerability also concerns the version 1.9.6, which is the one running here.\\
This might be something we could exploit. But before looking for this potential exploit, let's have a look at the website.

<div class="img_container">
![page web]({{https://jsom1.github.io/}}/_images/htb_tx_web.png)
</div>

There is a page with content, and we see "David White". This could be useful at some point, who knows.
If we scroll all the way down, we see the following:

<div class="img_container">
![page web]({{https://jsom1.github.io/}}/_images/htb_tx_basic.png)
</div>

Apparently, the website is using the template *Basic* from *TemplateMag*. 
I have never seen TemplateMag, and I didn't find anything interesting with a quick search on the web. 
But once again, it could be a track for later.\\
The website doesn't seem to have any other useful information, so let's use **dirbuster** to look for directories.

<div class="img_container">
![dirb]({{https://jsom1.github.io/}}/_images/htb_tx_dirb.png)
</div>

There are a few directories, but a first quick search didn't give anything. I let it run a moment to search inside those directories, but stopped as it was taking a lot of time.
If I had nothing else to try, I would have waited until the end of the search, but can also search Exploit-DB for a *nhttpd* exploit.
It this doesn't work, we will come back to dirbuster.

<div class="img_container">
![searchsploit]({{https://jsom1.github.io/}}/_images/htb_tx_srchsploit.png)
</div>

The 4th one is the one we are interested in. It's written version 1.9.3 and the web server is running 1.9.6, but we saw earlier that this latter is also vulnerable.\\
The exploit is a **directory traversal remote command**: I don't know what it is, and as usual, I will search for it on the web.\\
According to the internet, a *directory traversal* consists in exploiting insufficient security validation of user-supplied input file names, such that characters representing *traverse to parent directory* are passed through to the file APIs.

Let's start Metasploit and look if it contains the exploit.

<div class="img_container">
![search msf]({{https://jsom1.github.io/}}/_images/htb_tx_srch.png)
</div>

Perfect, the exploit is here. We can use it and look at the required options.

<div class="img_container">
![Exploit options]({{https://jsom1.github.io/}}/_images/htb_tx_options.png)
</div>

We only have to set RHOSTS (remote host, here 10.10.10.161) and LHOST (local host, here 10.10.14.18. This value is my VPN IP that can be found with *ifconfig tun0* command).\\
The default payload is *cmd/unix/reverse_perl*. A reverse shell is when a target machine initiates a connection to a user (the attacker), and the attacker listens on a specified port.\\
Usually, it's the opposite. A user initiates a shell connection with a remote machine (a server). However, this is often not possible because of firewalls and this is why we use reverse shells.\\
Indeed, attacked servers usually only allow connections on specific ports (a web server accepts coonnections on ports 80 and 443).\\
Since firewalls usually don't limit outgoing connections, the idea is that we can establish a server on our machine and create a reverse connection.
Here, the reverse shell used is written in Perl.\\
Let's try the exploit without changing the payload.

<div class="img_container">
![Exploit]({{https://jsom1.github.io/}}/_images/htb_tx_exploit.png)
</div>

It worked, we can execute commands! We can try to get a shell with the command *shell*:

<div class="img_container">
![whoami]({{https://jsom1.github.io/}}/_images/htb_tx_shell.png)
</div>

We see that we're the user *www-data*. This user has limited privileges, and we now need to find a way to escalate them: it is time for **enumeration**. We will be searching for any interesting information, be it usernames, passwords, misconfigurations, and so on.
Because we're on a Linux machine, we navigate with *cd* to change directory, *ls* to list the content of a directory and *cat* to display the content of a file.\\
By going a few directories back with *cd ..*, we quickly find */home*:

<div class="img_container">
![home dir]({{https://jsom1.github.io/}}/_images/htb_tx_home.png)
</div>

And we find david (whose name was on the webpage).
We can use the command *hostname* to get information on the host:

<div class="img_container">
![host]({{https://jsom1.github.io/}}/_images/htb_tx_host.png)
</div>

The host is traverxec, which is also the name of the box. Maybe it is because of the *directory traversal* RCE exploit, but it's probably not important. After searching a while, we find the following in */var/nostromo/conf*:

<div class="img_container">
![interesting config]({{https://jsom1.github.io/}}/_images/htb_tx_conf.png)
</div>

We see that the password is in */var/nostromo/conf/.htpasswd*, so let's look at this file:

<div class="img_container">
![password hash]({{https://jsom1.github.io/}}/_images/htb_tx_passwd.png)
</div>

We have a hashed password that we can crack with *JtR* (John The Ripper). I copied the password, opened a new terminal tab and pasted the password in a new file *pw.txt*. I then used JtR as follows:

<div class="img_container">
![JtR]({{https://jsom1.github.io/}}/_images/htb_tx_crack.png)
</div>

and we get the plaintext password. So, I tried to connect with SSH but it didn't work:

<div class="img_container">
![Try SSH]({{https://jsom1.github.io/}}/_images/htb_tx_ssh.png)
</div>

So, we must find another way. Looking back at the *nhttpd.conf* file, we see the 2 last lines which are:\\
homedirs /home\\
homedirs_public public_www\\
I had no clue what public_www was and checked on the internet. By typing it in Google, the first link is about *public_html*. It's not exactly the same, but maybe the idea is similar.\\
According to this page, *public_html* (so maybe *public_www* too?) is a **directory** on computers running Apache web servers that stores all HTML files and other web content to be wieved on the internet. Then, the following example is given:

~~~~
user/public_html/directory/file
~~~~~~

If we think about our *public_www*, maybe it is a directory in david's directory which contains interesting files. Let's try it:

<div class="img_container">
![Hidden dir]({{https://jsom1.github.io/}}/_images/htb_tx_hidden.png)
</div>

Bingo, we get something! Let's look into this *protected file area*:

<div class="img_container">
![Hidden dir]({{https://jsom1.github.io/}}/_images/htb_tx_hidden2.png)
</div>

And we find the *.htaccess* file that we saw in the *nhttpd.conf* file, and SSH credentials. We don't have the permission to unzip the file here, and it wouldn't be nice to other people hacking this box. Let's temporarily copy the file to a directory where we have more permissions, like */tmp*, and transfer it on our kali machine (don't forget to remove the file in */tmp* with *rm* once you're done, otherwise it would spoil other people!):

<div class="img_container">
![copy file]({{https://jsom1.github.io/}}/_images/htb_tx_cp.png)
</div>

Now, we *cd* in */tmp*. On our kali machine, we execute the following command:

<div class="img_container">
![netcat file]({{https://jsom1.github.io/}}/_images/htb_tx_nc1.png)
</div>

Which creates an empty file called "keys" (here on the Desktop). Then, from the target machine, we execute:

<div class="img_container">
![netcat file]({{https://jsom1.github.io/}}/_images/htb_tx_nc2.png)
</div>

Which transfers the file *backup-ssh-identity-files.tgz* on our kali. We see that the previous empty "keys" file is now a compressed file (gzip), that we can unzip with the following command:

<div class="img_container">
![unzip]({{https://jsom1.github.io/}}/_images/htb_tx_unzip.png)
</div>

It creates a new folder on the Desktop called *home*, in which there is a another folder, *david*. The keys were unzipped in this latter. We can see them by *cd*ing into it. At this point, we just have to crack the *id_rsa* key. However, before feeding the key to *JtR*, it has to be transformed into a format that it likes: this can be done with **ssh2john**. Then, we can use *JtR* to crack it. Those steps are presented below:

<div class="img_container">
![ssh2john]({{https://jsom1.github.io/}}/_images/htb_tx_rsa2j.png)
</div>

This created a new file *id_rsa_hash.txt* that we can give to *JtR*:

<div class="img_container">
![passphrase crack]({{https://jsom1.github.io/}}/_images/htb_tx_passphrase.png)
</div>

Perfect, it found the password **hunter** and we should now be able to log in with SSH! Let's try:

<div class="img_container">
![ssh david]({{https://jsom1.github.io/}}/_images/htb_tx_sshfin.png)
</div>

Note that we have to SSH from Kali. Technically, we could unzip the file on the target, but the keys would go in */home/david/.ssh* and we cannot access them there.\\
The first thing I did was of course to get my candy! As usual, the user flag is the */home* directory:

<div class="img_container">
![first flag]({{https://jsom1.github.io/}}/_images/htb_tx_flag.png)
</div>

Now, we must find a way to get root... Let's see what is in the */bin* directory:

<div class="img_container">
![interesting files]({{https://jsom1.github.io/}}/_images/htb_tx_ls.png)
</div>

There are 2 interesting files, but one has more open permissions (*server-stats.sh*), so let's look at what it contains:

<div class="img_container">
![script]({{https://jsom1.github.io/}}/_images/htb_tx_sss.png)
</div>

Let's roughly analyze it line by line:\\
The first thing it does it displaying the content of *server-stats.head*.\\
Then, it returns "Load:" followed by some value.\\
Empty line.\\
The text "Open nhttpd sockets:" is followed by a value.\\
It then displays "Files in the docroot:" and shows this number.\\
Empty line.\\
It returns "Last 5 journal log lines:"\\
And finally, it runs */usr/bin/journalctl* as */usr/bin/sudo*. This is interesting, it could be the way to root (because something is executed with high privileges).\\

Let's look at the real output of this script:

<div class="img_container">
![script exec]({{https://jsom1.github.io/}}/_images/htb_tx_script.png)
</div>

Apparently, *server-stats.head* draws a nice computer and prints some text. Then, we see the information listed above and the last 5 journal log lines...\\
Since *journalctl* is run as sudo, let's have a look at **GTFOBins** (this is what we did in OpenAdmin with a similar situation):

<div class="img_container">
![GTFOBins journalctl]({{https://jsom1.github.io/}}/_images/htb_tx_gtfo1.png)
</div>

We see that *journalctl* invokes the default pager, which is likely *less* (confirmed by a quick search on the internet). Logs are collected by a daemon called *journald*, and can be seen with the command *journalctl*. By looking at *less* in GTFO bins, we see interesting commands.\\
The thing is, we must find a way to execute them. To do so, we can copy and modify *server-stats.sh*. I created a new file (*.mod-script*) and copied the code inside. Then, we remove the pipe and what follows:

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_tx_modscript.png)
</div>

doing so, we actually have an instance of the pager *less* running as root, and we can enter the command found on GTFOBins to read files:

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_tx_cmd.png)
</div>

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_tx_root.png)
</div>

And this is it, we have the root flag!

<ins>**My thoughts**</ins>

Another great machine! This box was similar to [OpenAdmin](../_walkthroughs/OpenAdmin.md) in the sense that user flag required enumeration and root flag required to use GTFOBins. Furthermore, the steps from the initial shell to user via SSH was exactly the same than in [OpenAdmin](../_walkthroughs/OpenAdmin.md). Despite the resemblance, I really liked it because it reinforced what I had learned, and the box was funny.

We found a password for david (Nowonly4me) that we didn't use anywhere. Maybe there is another way to get the flags ?! I didn't try it, but it's something interesting!
