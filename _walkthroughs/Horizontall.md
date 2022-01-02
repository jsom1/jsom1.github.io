---
title: "Horizontall"
author: "Me"
date: "December 31, 2021"
output: html_document
---

# Horizontall

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: Easy (?/10)</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Linux</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/horizontall/htb_hor_desc.png){: height="300px" width = 320px"}
</div>

**Ports/services exploited:** ?\\
**Tools:** AutoRecon\\
**Techniques:** ?\\
**Keywords:** ?

**TL;DR**: 


## 1. Services enumeration
{:style="color:DarkRed; font-size: 170%;"}

I've read a lot about an enumeration tool called **AutoRecon**, so I will give it a try on this box. All the installation steps are described in the Github repo (<https://github.com/Tib3rius/AutoRecon>) and won't be covered here. The author says that *Autorecon* works by firstly performing port scans / service detection scans. From those initial results, the tool will launch further enumeration scans of those services using a number of different tools. For example, if HTTP is found, feroxbuster will be launched (as well as many others). There are various possible parameters, but I'll try here with the "basic" syntax:

````
sudo autorecon 10.10.11.105
`````

Below is a copied/pasted explanation of the results:

*By default, results will be stored in the ./results directory. A new sub directory is created for every target. The structure of this sub directory is:*

````
.
├── exploit/
├── loot/
├── report/
│   ├── local.txt
│   ├── notes.txt
│   ├── proof.txt
│   └── screenshots/
└── scans/
	├── _commands.log
	├── _manual_commands.txt
	└── xml/
``````

*The exploit directory is intended to contain any exploit code you download / write for the target.*

*The loot directory is intended to contain any loot (e.g. hashes, interesting files) you find on the target.*

*The report directory contains some auto-generated files and directories that are useful for reporting:*

*local.txt can be used to store the local.txt flag found on targets.
notes.txt should contain a basic template where you can write notes for each service discovered.
proof.txt can be used to store the proof.txt flag found on targets.
The screenshots directory is intended to contain the screenshots you use to document the exploitation of the target.
The scans directory is where all results from scans performed by AutoRecon will go. This includes port scans / service detection scans, as well as any service enumeration scans. It also contains two other files:*

*_commands.log contains a list of every command AutoRecon ran against the target. This is useful if one of the commands fails and you want to run it again with modifications.
_manual_commands.txt contains any commands that are deemed "too dangerous" to run automatically, either because they are too intrusive, require modification based on human analysis, or just work better when there is a human monitoring them.*

Note that the scan took **48 minutes** to run. Despite the length (which can most likely be reduced with parameters), it is interesting because it creates a directory structure that encourages one to have a certain methodology. This can be especially useful if we have to write a report afterwards.

Let's look at the results in the */results/10.10.11.105* directory. We'll start by looking at the *scans* results:

<div class="img_container">
![scans]({{https://jsom1.github.io/}}/_images/horizontall/htb_hor_scans.png)
</div>

We see Autorecon performed a full TCP scan as well as a quick one, and also a top 100 udp ports scan. Because ports 22 and 80 are open, it created directories containing additional information about them. Let's look at *nmap's* output:

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/horizontall/htb_hor_nmap.png)
</div>

Note that we see exactly what *nmap* command the tool used. Finally, we see there's SSH on port 22 and a web server on port 80. Even though I like seeing web servers, they have a very large attack surface and it's not always easy to find a vulnerability...

## 2. Gaining a foothold
{:style="color:DarkRed; font-size: 170%;"}

Before visiting the web page, let's look into the */tcp80* directory. It's awesome, the folder contains results of directories bruteforce with different wordlists, and another *nmap* scan specifically for port 80.\\
This latter many http scripts and shows that it didn't find any CSRF vulnerabilities, could't determine the underlying framework or CMS, didn't find any DOM-based or stored XSS, and so on... Even though it didn't find anything we can use, it's still very useful to have this information so that we don't have to do these checks manually ourselves.

We saw in the *nmap* result that there was a redirect to *horizontall.htb*, so let's add it to our */etc/hosts* file and browse to it to see this website.

````
sudo echo "10.10.11.105 horizontall.htb" >> /etc/hosts
``````

<div class="img_container">
![site]({{https://jsom1.github.io/}}/_images/horizontall/htb_hor_site.png)
</div>

The page has many buttons, but nothing happens when we click on them. We see a bunch of *JavaScript* scipts when inspecting the page. At the bottom of it, there's a "Contact us" form. I tried sending something and intercept the request with *Burp*, but nothing happens when we click on *Send*.\\
Something *nmap* didn't say is wether it found any subdomain or not. Let's try this manually with the following command:

````
sudo wfuzz -c -f res.tst -Z -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --sc 200,202,204,301,302,307,4003 http://FUZZ.horizontall.htb
`````

The script crashes with the error "could not resolve host: ftp.horizontall.htb". I added this host into my */etc/hosts* file, but can't FTP there.

## 3. Horizontal privilege escalation
{:style="color:DarkRed; font-size: 170%;"}


## 4. Vertical privilege escalation
{:style="color:DarkRed; font-size: 170%;"}



<ins>**My thoughts**</ins>


<ins>**Fix the vulnerabilities**</ins>

