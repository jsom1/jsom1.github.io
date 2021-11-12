---
title: "Pentesting CheatSheet"
author: "Me"
date: "November 03, 2021"
output: html_document
---

# Pentesting CheatSheet
{:.no_toc}

This "CheatSheet" aims to provide a global vision of the various tools and techniques used in the writeups, so that I can easily reuse them (there are links to machines).

Here's the content so far:

1. TOC
{:toc}
{:style="color:black; font-size: 150%;"}

# 21 - FTP

- Check if anonymous FTP is allowed (user *anonymous*, blank password)
- Known vulnerable versions: ProFTPD 1.3.3c, vsFTPd 2.3.4

# 80 - HTTP

## Directories and files enumeration

- Directories and files enumeration with **dirb** (-r for non recursive):

````
sudo dirb http://targetIP -r
``````

- Directories and files enumeration with **gobuster**:

````
sudo gobuster dir -u http://targetIP/ -w /usr/share/wordlists/...wordlist
sudo gobuster dir -u http://targetIP/ -w /usr/share/wordlists/...wordlist -x php # looks for php files
``````
Example: BountyHunter

- Directories and files enumeration with **wpscan** (for Wordpress):

````
sudo wpscan --url targetIP # basic scan
sudo wpscan --url targetIP -e u # enumerate users
sudo wpscan --url targetIP -passwords file/path/passwords.txt # password attack
`````
Example: Basic Pentesting: 1

- If the CMS is Wordpress: check **wp_admin_shell_upload**. Example: Basic Pentesting: 1


## XXE Injection

Does the application parses XML? If yes, it might be vulnerable to XXE injection. POC:

````
<?xml version="1.0"?>
<!DOCTYPE data [
<!ELEMENT data (#ANY)>
<!ENTITY file SYSTEM "file:///etc/passwd">
]>
<data>&file;</data>
`````

Example: BountyHunter


## SQL Injection

Server-side vulnerability that targets tthe application's database.

## XSS (cross-site scripting)

Client-side vulnerability that works by manipulating a vulnerable web site so that it returns malicious JavaScript to users. When the malicious code executes inside a victim's browser, the attacker can fully compromise their interaction with the application.

**Detection**: submit simple unique input into every entry point in the applicationcon, identifying every location where the submitted input is returned in HTTP responses, and testing each location individually to determine whether suitably crafted input can be used to execute arbitrary JavaScript. Example:

````
<script>alert('test')</script>
`````
**Attention**: *alert()* is doesn't work on Chrome from verrsion 92 onward. In this case, we can use *print()* instead.

### Stored XSS
The malicious script comes from the website's database. PoC: input <script>alert('test')</script> in a field, submit it and refresh the page. If the browser interprets it, it should open a JavaScript pop-up alert.

### Reflected XXS 
Same principle, but the script isn't stored in a database. Instead, it comes from the HTTP request. Example: if the site/application has a research field, it does an http GET request with an URL parameter (*https://www.insecure.com/search?q=keyboard*). PoC: https://www.insecure.com/search?q=<script>alert</script>. We can then send this link to someone. When he/she visits the URL, the script executes in the user's browser.

### DOM (Document Object Model) based XSS 
The vulnerability resides in the client-side code (not the server-side). It happens when an application contains some client-side JavaScript that processes data from an untrusted source in an unsafe way, usually by writing the data back to the DOM. Example:\\
*var search = document.getElementById('search').value;\\
var results = document.getElementById('results');\\
results.innerHTML = 'You searched for: ' + search;*\\
If we can control the value of the input field, we ca craft a malicious value that causes our own script to execute:\\
*You searched for: <img src=1 onerror='/* malicious code here */'>*.


## CSRF (cross-site request forgery)

Principle: do http requests as the victim. In other words, induce a victim user to perform actions they don't intend to do.

## Login Bruteforcing

# 139, 445 - SMB

- SMB (Server Message Block, protocol to share resources on Windows local networks) enumeration and null sessions:

````
sudo nmblookup -A targetIP
sudo nbtscan targetIP
sudo smbclient -L targetIP
sudo smbmap -H targetIP
````


