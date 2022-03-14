---
title: "Pentesting CheatSheet"
author: "Me"
date: "November 03, 2021"
output: html_document
---

# Attack vectors
{:.no_toc}

This "Cheatsheet" lists the various tools and techniques used in the writeups to get a foothold.

Here's the content so far:

1. TOC
{:toc}
{:style="color:black; font-size: 150%;"}

# 21 - FTP

- Is anonymous FTP allowed (user *anonymous*, blank password)?

````
sudo ftp targetip
`````
Example: <a href="/_walkthroughs/ServMon">ServMon</a>

- Known vulnerable versions: ProFTPD 1.3.3c, vsFTPd 2.3.4, ... -> Search for Metasploit exploits

# 80 - HTTP

## Enumeration (Directories, files, subdomains, ...)

- Directories and files enumeration with **dirb** (-r for non recursive):

````
sudo dirb http://targetIP -r
``````

- Directories and files enumeration with **gobuster**:

````
sudo gobuster dir -u http://targetIP/ -w /usr/share/wordlists/...wordlist
sudo gobuster dir -u http://targetIP/ -w /usr/share/wordlists/...wordlist -x php # looks for php files
``````
Example: <a href="/_walkthroughs/BountyHunter">BountyHunter</a>

- Directories and files enumeration with **wpscan** (for Wordpress):

````
sudo wpscan --url targetIP # basic scan
sudo wpscan --url targetIP -e u # enumerate users
sudo wpscan --url targetIP -passwords file/path/passwords.txt # password attack
`````
Example: <a href="/_walkthroughs/basicpentest">Basic Pentesting: 1 (VulnHub)</a>

- Subdomains enumeration with **ffuf**:

````
sudo ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u http://targetIP/ -H "Host: FUZZ.targetIP" -t 200 -fl 10
`````
Example: <a href="/_walkthroughs/Forge">Forge</a>

- If the CMS is Wordpress: check **wp_admin_shell_upload**.
Example: <a href="/_walkthroughs/basicpentest">Basic Pentesting: 1 (VulnHub)</a>


## XXE Injection

Does the application parses XML? If yes, it might be vulnerable to XXE injection. PoC:

````
<?xml version="1.0"?>
<!DOCTYPE data [
<!ELEMENT data (#ANY)>
<!ENTITY file SYSTEM "file:///etc/passwd">
]>
<data>&file;</data>
`````

Example: <a href="/_walkthroughs/BountyHunter">BountyHunter</a>


## SQL Injection

Server-side vulnerability that targets the application's database.
Input strings (into every possible form of the app) that could trigger an error message. Examples:

````
admin' or 1=1;#
````

## XSS (cross-site scripting)

Client-side vulnerability that works by manipulating a vulnerable web site so that it returns malicious JavaScript to users. When the malicious code executes inside a victim's browser, the attacker can fully compromise their interaction with the application.

**Detection**: manually by submitting simple unique input into every entry point of the application, identifying every location where the submitted input is returned in HTTP responses, and testing each location individually to determine whether suitably crafted input can be used to execute arbitrary JavaScript. Example:

````
#<script>alert('test')</script>
`````
**Attention**: *alert()* doesn't work on Chrome from version 92 onward. In this case, we can use *print()* instead.\\
Automatically by using Burp's web vulnerability scanner.

**Stored XSS**\\
The malicious script comes from the website's database. PoC: input <script>alert('test')</script> in a field, submit it and refresh the page. If the browser interprets it, it should open a JavaScript pop-up alert.

**Reflected XXS**\\
Same principle, but the script isn't stored in a database. Instead, it comes from the HTTP request. Example: if the site/application has a research field, it does an http GET request with an URL parameter (*https://www.insecure.com/search?q=keyboard*). PoC: https://www.insecure.com/search?q=<script>alert</script>. We can then send this link to someone. When he/she visits the URL, the script executes in the user's browser.

**DOM (Document Object Model) based XSS**\\
The vulnerability resides in the client-side code (not the server-side). It happens when an application contains some client-side JavaScript that processes data from an untrusted source in an unsafe way, usually by writing the data back to the DOM. Example:\\
*var search = document.getElementById('search').value;\\
var results = document.getElementById('results');\\
results.innerHTML = 'You searched for: ' + search;*\\
If we can control the value of the input field, we ca craft a malicious value that causes our own script to execute:\\
*You searched for: <img src=1 onerror='/* malicious code here */'>*.


## CSRF (cross-site request forgery)

Principle: do http requests as the victim. In other words, induce a victim user to perform actions they don't intend to do.

## SSRF (server-side request forgery)

The target application may have functionality for importing data from a URL, publishing data to a URL or otherwise reading data from a URL that can be tampered with. The attacker modifies the calls to this functionality by supplying a completely different URL or by manipulating how URLs are built (path traversal etc.).\ When the manipulated request goes to the server, the server-side code picks up the manipulated URL and tries to read data to the manipulated URL. By selecting target URLs the attacker may be able to read data from services that are not directly exposed on the internet. The attacker may also use this functionality to import untrusted data into code that expects to only read data from trusted sources, and as such circumvent input validation.

Example: <a href="/_walkthroughs/Forge">Forge</a>

## SSTI (server-side template injection)

Some web applications use template engines to dissociate the visual display (HTML, CSS, ...) from the application logic (PHP, python, ...). Those engines create templates in the application, which are a mix of "hard-coded" data (the page layout's code) and dynamic variables. When the application is used, the template engine repalces the variables stored in the template, transforms it into a web page (HTML) and sends it to the client.\\
An SSTI vulnerability arises when users inputs are directly passed in a template, and executed by the template engine.\\

Detection: as for XXS and SQLi, SSTI vulnerabilities can be discovered by fuzzing the application's fields.

Example: <a href="/_walkthroughs/Bolt">Bolt</a>

## Login Bruteforcing

## Directory traversal

Directory traversal (also known as file path traversal) is a web security vulnerability that allows an attacker to read arbitrary files on the server that is running an application. This might include application code and data, credentials for back-end systems, and sensitive operating system files. In some cases, an attacker might be able to write to arbitrary files on the server, allowing them to modify application data or behavior, and ultimately take full control of the server.

Example: <a href="/_walkthroughs/Seal">Seal</a>

# 139, 445 - SMB

- SMB (Server Message Block, protocol to share resources on Windows local networks) enumeration and null sessions. Base commands:

````
sudo nmblookup -A targetIP
sudo nbtscan targetIP
sudo smbclient -L targetIP
sudo smbmap -H targetIP
````

Those commands are automated by *enum4linux*:

`````
sudo /usr/share/enum4linux/enum4linux.pl <targetIP>
``````

Run all nmap's SMB vulnerability checks:

````
sudo nmap --script smb-enum-shares -p139,445 <targetIP>
`````
Example: <a href="/_walkthroughs/Love">Love</a>


