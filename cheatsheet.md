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
- Vulnerable versions: ProFTPD 1.3.3c

# 80 - HTTP

- Directories and files enumeration with **dirb** (-r for non recursive):

````
sudo dirb http://targetip -r


- Directories and files enumeration with **gobuster**:

````
sudo gobuster


- Directories and files enumeration with **wpscan** (if the CMS is Wordpress):

````
sudo gobuster
`````



- If the CMS is Wordpress: check **wp_admin_shell_upload**

## Directories and files enumeration

## XXE Injection

## SQL Injection

## Login Bruteforcing

# 139, 445 - SMB



