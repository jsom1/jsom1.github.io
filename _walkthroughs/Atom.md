---
title: "Atom"
author: "Me"
date: "June 20, 2021"
output: html_document
---

# Atom

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: medium (?/10)</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Windows</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_knife_desc.png)
</div>

**Ports/services exploited:** SMB\\
**Tools:** dirb, smbclient\\
**Techniques:** Enumeration\\
**Keywords:** Electron builder\\


## 1. Port scanning
{:style="color:DarkRed; font-size: 170%;"}

As usual, let's perform a nmap scan with the flags *-sV* to have a verbose output, *-O* to enable OS detection, and *-sC* to enable the most common scripts scan.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_atom_nmap1.png)
</div>
<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_atom_nmap2.png)
</div>

Well, that's a lot of information! The running services are:

- HTTP on port 80: running httpd 2.4.46 and PHP/7.3.27. The box *Knife* had a webserver running PHP/8.0.1-dev which was vulnerable to RCE. We'll check for this version as well.
- RPC (Remote Procedure Call) on port 135: a protocol that uses the client-server model in order to alllow one program to request service from a program on another computer without having to understand the details of that computer's network.
- HTTPS (HTP over TLS/SSL) on port 443
- SMB on port 445

Nmap also tried to guess the OS and thinks it could be Windows Server 2008 SP1 or R2 (85%) or Windows 7 (85%). Finally nmap scripts returned informations regarding SMB.\\
There are many things to go over, so let's begin!


## 2. Find and exploit vulnerabilities
{:style="color:DarkRed; font-size: 170%;"}

The easiest thing to do is to browse to the server to get more information about it:

<div class="img_container">
![website]({{https://jsom1.github.io/}}/_images/htb_atom_site1.png){: height="415px" width = 625px"}
</div>
<div class="img_container">
![website]({{https://jsom1.github.io/}}/_images/htb_atom_site2.png)
</div>

We learn from the page that Heed is a software company. They propose here a simple note taking application (v1.0.0) that can be downloaded (for Windows only).
There's also an email (MrR3boot@atom.htb (MrR3boot is the creator of this box)) and we see it was created with Codepen. Codepen is a development environment for front-end developers.

Before downloading the program, we'll use nikto to scan the web server for vulnerabilities and dirb to bruteforce its directories. The command for nikto is:

````
sudo nikto -h 10.10.10.237
`````
The command returns *0 host(s) tested*, that's weird... Let's leave it aside for now and check dirb:

<div class="img_container">
![dirb]({{https://jsom1.github.io/}}/_images/htb_atom_dirb.png)
</div>

It found 17 directories but most of them are forbidden to access (code 403). The two "images" contain the heed logo, and */releases* contain a zip file (*heed_setup_v1.0.0.zip*). It's the same file we can download on the main page. We also see */phpmyadmin*, but we can't access it. However, there must be a database somewhere for authentication, so let's perform a full TCP scan to check higher ports:

<div class="img_container">
![Full TCP scan]({{https://jsom1.github.io/}}/_images/htb_atom_fullscan.png)
</div>

We see wsman and redis on ports 5985 and 6379 (default port) respectively. I don't know those services, so here's what I found on Google:

- WSMan (also called WinRM): it's a DMTF (Distributed Management Task Force) open standard defining a SOAP(Simple Object Access Protocol)-based protocol (SOAP is a messaging protocol specification for exchanging structured information in the implementation of web services in computer networks. It uses XML Information Set for its message format, and relies on application layer protocols, most often Hypertext Transfer Protocol (HTTP)) for the management of servers, devices, applications and web services.
- Redis (REmote DIctionary Server): it's an in-memory data structure store, used as a cache and database.

Let's try to gather more information about redis. There's an nmap script to enumerate the instance:

<div class="img_container">
![redis scan]({{https://jsom1.github.io/}}/_images/htb_atom_redis.png)
</div>

It doesn't really help us since it doesn't return any version. Because redis is a text based protocol, we can send a commad in a socket and the returned values should be readable. We can try with netcat for example (or telnet) and the command *info* to get information:

<div class="img_container">
![netcat redis]({{https://jsom1.github.io/}}/_images/htb_atom_nc.png)
</div>

The message *-NOAUTH Authentication required* means that we need valid credentials to access the instance. By default, Redis can be accessed without credentials but that can be configured to support only password or username + password. That seems to be the case here.

We can't get the version of the service. I still checked on Google for *redis key-value store exploit* and found a few pages among which a promising RCE for Redis 4.0.9 (also found on Metasploit). However, it requires that Redis supports anonymous authentication which we saw was not the case. This service might not be our way in and we must find something else.\\
A quick search about WSMan makes me think it's not the right way either, so let's look for something else. Maybe instead of digging into one of the many possibilities I'll just scratch the surface of the possible attack vector and try to find a low hanging fruit.\\

We could go and download the app on the webpage, but it's a .exe... So we would either need a Windows VM or install Wine on Kali to run it. Because I don't have a Windows VM nor Wine installed, let's look at something else for now.

Let's look at SMB and do a quick enumeration. SMB (Server Message Block) is designed to be used as a file sharing protocol. With SMB, an authorized user/application can access files on a remote server.\\
We saw nmap already ran the script *smb-os-discovery*. It showed the computer's name on the network (ATOM), the account used (guest) and stated that "message_signing is disabled" and that it is dangerous. Signing in SMB is a feature that allows SMB communications to be digitally signed at the packet-level. This allows the recipient to verify the authenticity of the source.\\
A quick Google search reveals this is a medium risk vulnerability that can allow MitM (man-in-the-midddle) attacks against the server. It is also said that this vulnerability is prone to false positive reports by most vulnerability assessment solutions.\\
Let's continue SMB enumeration. We can look at share names with **smbclient**:

<div class="img_container">
![Smbclient]({{https://jsom1.github.io/}}/_images/htb_atom_smb.png)
</div>

Note that I tried with different passwords such as *root*, *admin* and empty password, it always return this information. A few words about shares: SMB supports two levels of security. The first one is at the share level and protects the server. Each share has a password that the client user has to provide to access its information. The second one is at the level of the user: it is applied to individual files and each share is based on specific user access rights.\\
In the picture above we see four shares, *ADMIN$*, *C$*, *IPC$* and *Software_Updates*.\\

We can then use *smbclient* to access those shares:

<div class="img_container">
![shares]({{https://jsom1.github.io/}}/_images/htb_atom_shares.png)
</div>

In the share Software_Updates, there is one document which doesn't have an empty size. We can download it on our machine with the command *get* (it is downloaded in the directory from which we executed the *smbclient* command). Let's look at this file that we can open on Kali with the command:

````
xdg-open UAT_Testing_Procedures.pdf
`````

<div class="img_container">
![PDF]({{https://jsom1.github.io/}}/_images/htb_atom_pdf.png)
</div>

We see on the page that the application was built with *electron-builder*. It is also mentionned that it is a one-tier application. In other words, it's the simplest architecture of a database in which the client, server and database all redise on the same machine. Finally we see the steps of the QA (Quality Assurance) process: build the application and place the updates in one of the "client" folders.\\

Let's look at what *electron-builder* is. It's an open-source software framework developed and maintained by GitHub. It allows developers to create cross-platform applications with web technologies such as HTML, CSS and Javascript. Some well-known Electron applications include **Atom editor**, Visual Studio Code, and Slack.\\
So this is why the box is called Atom! We'll search for known vulnerabilities by Googling "electron-builder exploit". The first result mentions a signature validation bypass leading to RCE (https://blog.doyensec.com/2020/02/24/electron-updater-update-signature-bypass.html). That sounds interesting!\\

Apparently, Electon-builder is frequently used for software updates. The auto-update feature is provided by its electron-updater submodule, internally using NSIS for Windows. It features a dual code-signing method for Windows.\\
The article states there's an overall lack of secure coding practices in the update mechanism. In particular, a vulnerability can be used to bypass the signature verification check, eventually leading to **remote command execution**.\\
The signature verification check is based on a string comparison between the installed binary's *publisherName* and the certificate's *Common Name* attribute of the update binary. When there's a software update, the application will request a file named *latest.yml* from the update server. This file contains the definition of the new release, including the binary filename and hashes.

The module executes the following code to retrieve the update binary's publisher. It leverages the cmdlet *Get-AuthenticodeSignature* from PowerShell.

````
    execFile("powershell.exe", ["-NoProfile", "-NonInteractive", "-InputFormat", "None", "-Command", `Get-AuthenticodeSignature '${tempUpdateFile}' | ConvertTo-Json -Compress`], {
      timeout: 20 * 1000
    }, (error, stdout, stderr) => {
      try {
        if (error != null || stderr) {
          handleError(logger, error, stderr)
          resolve(null)
          return
        }

        const data = parseOut(stdout)
        if (data.Status === 0) {
          const name = parseDn(data.SignerCertificate.Subject).get("CN")!
          if (publisherNames.includes(name)) {
            resolve(null)
            return
          }
        }

        const result = `publisherNames: ${publisherNames.join(" | ")}, raw info: ` + JSON.stringify(data, (name, value) => name === "RawData" ? undefined : value, 2)
        logger.warn(`Sign verification failed, installer signed with incorrect certificate: ${result}`)
        resolve(result)
      }
      catch (e) {
        logger.warn(`Cannot execute Get-AuthenticodeSignature: ${error}. Ignoring signature validation due to unknown error.`)
        resolve(null)
        return
      }
    })
``````

This code translates to the following PowerShell command:

````
powershell.exe -NoProfile -NonInteractive -InputFormat None -Command "Get-AuthenticodeSignature 'C:\Users\<USER>\AppData\Roaming\<vulnerable app name>\__update__\<update name>.exe' | ConvertTo-Json -Compress"
`````

The idea is that since the *${tempUpdateFile}* variable is provided unescaped to the *execfile* utility, we can bypass the signature verification by triggering a parse error in the script. It can be done by using a filename containing a sinngle quote and then by recalculating the file hash to match our provided binary (with *shasum -a 512 maliciousupdate.exe | cut -d " " -f1 | xxd -r -p | base64*).\\
When serving a **latest.yml** file to a vulnerable Electron app, our chosen setup executable will be run without warnings. We could for example leverage the lack of escaping to pull a command injection as follows:

````
version: 1.2.3
files:
  - url: v';calc;'ulnerable-app-setup-1.2.3.exe
  sha512: GIh9UnKyCaPQ7ccX0MDL10UxPAAZ[...]tkYPEvMxDWgNkb8tPCNZLTbKWcDEOJzfA==
  size: 44653912
path: v';calc;'ulnerable-app-1.2.3.exe
sha512: GIh9UnKyCaPQ7ccX0MDL10UxPAAZr1[...]ZrR5X1kb8tPCNZLTbKWcDEOJzfA==
releaseDate: '2019-11-20T11:17:02.627Z'
``````

So from what I understand of the article, we have to write a malicious *latest.yml* file and place it in one of the client folders (we saw that on the PDF). This will automatically initiate the QA process. Therefore, we must find how to write that file.




<ins>**My thoughts**</ins>

Long time no Windows, maybe more frequet than Linux. Not used to so many information, struggle to know where to start.
