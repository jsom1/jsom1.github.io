---
title: "Time"
author: "Me"
date: "March 07, 2021"
output: html_document
---

# Time

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: medium (4.1/10)</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Linux</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_time_desc.png){: height="250px" width = "280px"}
</div>

**Tools:** ?\\
**Techniques:** Exploit tweaking\\
**Keywords:** JSON, Jackson, serialization/deserialization\\


## 1. Port scanning
{:style="color:DarkRed; font-size: 170%;"}

As usual, let's perform a nmap scan with the flags *-sV* to have a verbose output, *-O* to enable OS detection, and *-sC* to enable the most common scripts scan.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_time_nmap.png)
</div>

We see SSH and a web server running. SSH is rarely if ever exploitable at first, so we don't have many choices on where to begin.

## 2. Find and exploit vulnerabilities
{:style="color:DarkRed; font-size: 170%;"}

While we have a look at the web page, let's launch *dirb* to look for potential interesting directories:

<div class="img_container">
![dirb]({{https://jsom1.github.io/}}/_images/htb_time_dirb.png){: height="320px" width = "550px"}
</div>

Let's look at the main page:

<div class="img_container">
![site web]({{https://jsom1.github.io/}}/_images/htb_time_site.png){: height="320px" width = "550px"}
</div>

Dirb found a few directories, but we cannot access any of them. The page appears to contain an online app that beautify and validate json. Let's try to beautify the following JSON POST request syntax :

```
{
"username": "admin",
"password": "admin",
"validation-factors": {
"validationFactors": [
{
"name": "remote_address",
"value": "127.0.0.1"
}
]
}
}
```

<div class="img_container">
![beautify]({{https://jsom1.github.io/}}/_images/htb_time_beautify.png){: height="320px" width = "550px"}
</div>

It seems to work. Let's copy and paste the output and try to validate it:

<div class="img_container">
![validate]({{https://jsom1.github.io/}}/_images/htb_time_validate.png){: height="320px" width = "550px"}
</div>

For some reason it fails, but we don't really know what it's doing. By trying the following random basic GET request:

````
GET /articles?include=author HTTP/1.1
````
We get the following error message: *validation failed: Unhandled Java exception: com.fasterxml.jackson.core.JsonParseException: Unrecognized token 'GET': was expecting ('true', 'false' or 'null')*. I tried many other commands, and they all returned a very similar error containing *com.fasterxml.jackson*. While we investigate this error, let's start a full TCP scan as well as un UDP scan to make sure there's not another open port and that we're on the right track. The command for the UDP scan is the following:

````
sudo nmap -sV -sU 10.10.10.214
````
And the full TCP one is:
````
sudo nmap -sV -sC -p 1-65535 10.10.10.214
````
UDP scans are way slower than TCP scans, so it is going to take some time. The TCP scan finishes in a few seconds and doesn't bring anything new. After a while, we see the UDP scan doesn't reveal anything either.\\
By googling the error we saw previously, I found a StackExchange describing a similar problem:

<div class="img_container">
![StackExchange]({{https://jsom1.github.io/}}/_images/htb_time_stack.png){: height="320px" width = "550px"}
</div>

The person gives a link and mentions the three CVEs we see in the image below:

<div class="img_container">
![CVE]({{https://jsom1.github.io/}}/_images/htb_time_cve.png){: height="320px" width = "550px"}
</div>

There is however no Metasploit module linked to any of those CVEs, and I don't really know what to do from there. Looking at other resources on the internet, it seems that this vulnerability is caused by an **insecure deserialization**. Serialization is the process of converting a an object, such as a Java object, from a programming language to a format that can be stored in a database or sent over the network. Desrialization is the opposite process.\\
Insecure deserialization happens when an attacker is able to manipulate the serialized object, eventually leading to DoS, authentication bypass or RCE. 

From what I understand in our case, we submit a JSON object to the application, and this latter serializes it before sending it to the server. This serialized object is then desrialized and "executed". The idea could then be to embed our own malicious code in the JSON object we submit to the application.

<ins>**My thoughts**</ins>

