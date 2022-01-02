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

Note that the scan took **48 minutes** to run. Despite the length (which can most likely be reduced with parameters), it is interesting because it creates a directory structure that encourages one to have a certain methodology.\\
Let's look at the results in the */results/10.10.11.105* directory. We'll start by looking at the *scans* results so that we see something similar to *nmap*:




## 2. Gaining a foothold
{:style="color:DarkRed; font-size: 170%;"}

## 3. Horizontal privilege escalation
{:style="color:DarkRed; font-size: 170%;"}


## 4. Vertical privilege escalation
{:style="color:DarkRed; font-size: 170%;"}



<ins>**My thoughts**</ins>


<ins>**Fix the vulnerabilities**</ins>

