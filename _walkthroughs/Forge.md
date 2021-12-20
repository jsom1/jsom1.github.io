---
title: "Forge"
author: "Me"
date: "December 13, 2021"
output: html_document
---

# Forge

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: Medium (?/10)</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Linux</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_forge_desc.png){: height="300px" width = 320px"}
</div>

**Ports/services exploited:** 80/web application, 21/FTP\\
**Tools:** wfuzz, ffuf\\
**Techniques:** subdomain enumeration, SSRF (server side request forgery)\\
**Keywords:** pdb (python debugger)

**In a nutshell**: A web server hosts an application that has a functionality to upload files by providing a URL, making it a typical candidate for SSRF. By manipulating the URL, we can access a directory that shouldn't be accessible. This latter contains credentials and information that can be used to perform SSRF with the *FTP* protocol, allowing us to retrieve SSH public/private keys and connect to the machine as a normal user. This user then has "sudo permissions" on a custom script which can be used to spawn pdb (python debugger). The debugger can be used to modify permissions of */bin/bash*, allowing us to spawn it as root.

## 1. Services enumeration
{:style="color:DarkRed; font-size: 170%;"}

As usual, we start by using *nmap* to discover the services running on the target. We use the flags *-sV* to determine service/version info, *-sC* to run default scripts (equivalent to *--script=default*), and *-O* to enable OS detection.

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_forge_nmap.png)
</div>

We see FTP on port 21, but its state is *filtered*. That means that a firewall, filter, or other network obstacle is blocking the port (the packets are being filtered). Therefore, *nmap* cannot tell whether it's open or closed. Sometimes it's a little bit more verbose, displaying for example an ICMP error message such as "*destionation unreachable: communication administratively prohibited*". It's not the case here however.\\
Next, we see SSH on port 21. This service is almost always running on HtB machines, but it's rarely the right attack vector. It's rather used once we discover credentials. From memory, I don't think this version is vulnerable.\\
Finally, there's a Apache web server on port 80, and this is where we will start this box.
## 2. Gaining a foothold
{:style="color:DarkRed; font-size: 170%;"}

I don't do it systemically, but let's add the IP adress and hostname to our */etc/hosts* file:

````
sudo echo "10.10.11.111 forge.htb" >> /etc/hosts
`````

Then, we can browse to *http://forge.htb* to discover what's hosted on the web server. The default page consists of a gallery of images and doesn't seem to have anything useful. In the upper right corner of the page, there's a *Upload an image* button. When clicking on it, we're redirected to */upload* and are presented with the following options:

<div class="img_container">
![site]({{https://jsom1.github.io/}}/_images/htb_forge_site.png){: height="300px" width = 320px"}
</div>

We can either upload a local file, or upload one from an url (in this case there's a blank space in which we can input an url). At this point, there are many things we can do: test for directory traversal vulnerabilities, try to upload a local file, and so on... While thinking about the possibilites, let's use *dirb* and *gobuster* to bruteforce directories. I'm using two different tools here to reduce the risk of missing something. Also, we see references to javascript when inspecting the page (right click on it -> Inspect element. In the Debugger tab, we see a script *main.js* in */static/js*. Therefore, we can ask *gobuster* to look for *.js* extensions specifically with the *-x* flag:

````
sudo dirb http://forge.htb
sudo gobuster dir -u http://forge.htb -w /usr/share/wordlists/dirb/big.txt
sudo gobuster dir -u http://forge.htb -w /usr/share/wordlists/dirb/big.txt -x js
`````

Note that by default, *dirb* uses the *common.txt* wordlist. So even though we used */dirb/big.txt* wordlist with gobuster, it's not exactly the same.\\
In this case, the result of the 3 scans are the same. The discovered directories are: */server-status* (code 403), */static* (code 301), */upload* (code 200) and */uploads* (code 301).

If we browse to */static*, we see 3 subdirectories: *css*, *images* (contains the images displayed on the default page) and *js* (contais the *main.js* script we saw by inspecting the page). Nothing really stands out here.

Next, let's try the functionnalities to upload a local file and one from an url, starting with the url. In the blank space, we input the following: *http://forge.htb/../../../../../* (if we don't start with *http* or *https*, we get an error message: "Invalid protocol! Supported protocols: http, https"). This results in another error message, "URL contains a blacklisted address!". It appears we can't use that IP...\\
Let's now try to upload a local file from our machine. In this example, I uploaded *linpeas.sh*:

<div class="img_container">
![upload]({{https://jsom1.github.io/}}/_images/htb_forge_upload.png){: height="300px" width = 320px"}
</div>

We see the file was successfully uploaded. However, if we click on the link, we get a "Not Found" error. We also see it was uploaded in */uploads*, and not in */upload*. *Dirb* and *gobuster* found this directory, but indicated it returned the HTTP code 301 (Moved Permanently or redirect).\\
Next, let's try to add the *.jpg* extension to *linpeas.sh* and try again. If we can successfuly upload it and see it, we might be able to upload a reverse shell payload file with a *.png* extention. The file is also successfully uploaded but this time, clicking the link shows another error message: "The image http://forge.htb/uploads/A1uTy4XXSvCfrHXvxeao cannot be displayed because it contains errors". A few seconds later, it's not being found anymore... Maybe the file is uploaded in */uploads* where it is subject to some tests, and if it fails, it gets removed? Anyways, that's probably a dead end.

As usual, I had to look at the forum at this point because I was stuck. Apparently, we're supposed to enumerate subdomains. I don't remember how we're supposed to do that, and a quick search mentionned that *wfuzz* is a good tool. I tried with the following syntax (which is supposed to work):

````
sudo wfuzz -c -f res.txt -Z -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --sc 200,202,204,301,302,307,4003 http://FUZZ.forge.htb
`````
Where *res.txt* is the file in which the results are stored, and *-sc* specifies the responses we want to keep. Finally, FUZZ in http://FUZZ.forge.htb is the word that will be replaced by the wordlist (seclists wordlists can be obtained with *sudo apt install seclists*) values. This should work, but I get the error "Unhandled exception: cannot add/remove handle - multi_perform() already running". I'm not the only to encounter this error, but there doesn't seem to be a quick and easy fix so I'll rather look at another tool.\\

The other tool that I tried is **ffuf** (sudo apt-get install ffuf) with the following syntax:

````
sudo ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u http://forge.htb/ -H "Host: FUZZ.forge.htb" -t 200 -fl 10
`````

This time the scan is successful and returns one subdomain: *admin.forge.htb* (*admin* is the subdomain, *forge* is the primary domain and *.htb* is the top-level domain):

<div class="img_container">
![ffuf]({{https://jsom1.github.io/}}/_images/htb_forge_ffuf.png)
</div>

Why would there be a subdomain? The most common use-case of a subdomain is for creating a testing version of a website. Changes are done on the subdomain before publishing them live on the internet. It is similar to having a dev and a prod line.

Let's add this subdomain to our */etc/hosts* (*sudo echo "10.10.11.111 admin.forge.htb" >> /etc/hosts*) file. The first thing I did was to try *dirb* and *gobuster* once again agaisnt *admin.forge.htb*, but dirb didn't return anything, and gobuster didn't even work (*error: the server returns a status code that matches the provided options for non-existing urls*).\\
Let's browse to that address:

<div class="img_container">
![subdomain]({{https://jsom1.github.io/}}/_images/htb_forge_subdom.png)
</div>

It seems we can't access it from a different IP address. Could that be the reason why gobuster failed? Anyways, maybe we can use the file upload option and provide the subdomain address to access resources on *admin.forge.htb*... In this manner, it is *forge.htb* that would request *admin.forge.htb*. I tried using that address to upload a file, but once again we get the message "URL contains a blacklisted address".

By Googling "only localhost is allowed", the first string completion propistion is "only localhost is allowed bypass". By looking at this, the first links all mention SSRF (Server Side Request Forgery).\\
I'm not familiar at all with this topic and don't know how it works, so here's a quick explanation from <https://owasp.org/www-community/attacks/Server_Side_Request_Forgery>:\\
*The target application may have functionality for importing data from a URL, publishing data to a URL or otherwise reading data from a URL that can be tampered with. The attacker modifies the calls to this functionality by supplying a completely different URL or by manipulating how URLs are built (path traversal etc.).\\
When the manipulated request goes to the server, the server-side code picks up the manipulated URL and tries to read data to the manipulated URL. By selecting target URLs the attacker may be able to read data from services that are not directly exposed on the internet. The attacker may also use this functionality to import untrusted data into code that expects to only read data from trusted sources, and as such circumvent input validation.*

The situation we're facing looks similar to what's described here. In addition, the box' name *forge* could be a hint for SSRF. That sounds good, but I still had no idea about how to proceed and had to look for help. I was close to find the solution: let's use Burp and intercept the request when we try to upload from a URL and see what happens:

<div class="img_container">
![Burp]({{https://jsom1.github.io/}}/_images/htb_forge_burp.png){: height="550px" width = 650px"}
</div>

I send the request to the repeater and sent it to the the response. We see the URL is encoded and we see that it contains a blacklisted address. This is where we should have thought about changing the URL, for example by using capital letters:

<div class="img_container">
![Burp2]({{https://jsom1.github.io/}}/_images/htb_forge_burp2.png){: height="550px" width = 650px"}
</div>

This time, we see it was successfully uploaded. Then, instead of clicking the given link in the browser, we should have thought about *curl*ing it:

````
sudo curl http://forge.htb/uploads/WAxEuGrfveJ9MV453crc
````

<div class="img_container">
![curl]({{https://jsom1.github.io/}}/_images/htb_forge_curl.png)
</div>

This way, we can somehow access the page. In particular, we see another directory, */annoucements*. I tried *curl*ing again to access it, but it didn't work. Let's try to specify it in the URL upload: *http://ADMIN.FORGE.HTB/announcements*. We then copy the link location and *curl* it:

<div class="img_container">
![creds]({{https://jsom1.github.io/}}/_images/htb_forge_creds.png)
</div>

First, we discover FTP credentials. We indeed saw FTP in nmap's output, but it was filtered. We now have the confirmation it is there.\\
Then, it is said that the */upload* endpoint supports ftp, ftps, http and https protocols. We knew it supported -and so far thought it only accepted- http or https protocols, but the fact it also supports FTP might be the way to use the credentials.\\
Finally, we see we can upload images by passing a *?u* parameter in the URL.

Let's try to connect to FTP by providing the correct URL: *http://ADMIN.FORGE.HTB/upload?u=ftp://user:heightofsecurity123!@FORGE.HTB*. By *curl*ing to the provided link, we get the following:

<div class="img_container">
![user]({{https://jsom1.github.io/}}/_images/htb_forge_user.png)
</div>

Note that I also grabbed the user flag. To do so, I used the following URL: *http://ADMIN.FORGE.HTB/upload?u=ftp://user:heightofsecurity123!@FORGE.HTB/user.txt*. 

At this point, I was stuck and had to look at the forum. The solution was kind of obvious: we can retrieve the public and private SSH keys and use them to connect. By default, the private key is stored in *~/.ssh/id_rsa*, and the public one is in *~/.ssh/id_rsa.pub*. To do that, we will use the following URL commands: 

```
*http://ADMIN.FORGE.HTB/upload?u=ftp://user:heightofsecurity123!@FORGE.HTB/.ssh/id_rsa*
*http://ADMIN.FORGE.HTB/upload?u=ftp://user:heightofsecurity123!@FORGE.HTB/.ssh/id_rsa.pub*
`````

The content of those file is the following:

<div class="img_container">
![ssh key]({{https://jsom1.github.io/}}/_images/htb_forge_key.png)
</div>

We've got a user (*user*) and the private key, that's everything we need to connect. We can copy this latter into a file, for example *id_rsa*. Then, we can SSH into the machine:

<div class="img_container">
![ssh error]({{https://jsom1.github.io/}}/_images/htb_forge_error.png)
</div>

I got that exact same error in <a href="/_walkthroughs/OpenAdmin">OpenAdmin</a>, and I had to change the file permissions as follows:

````
sudo chmod 600 id_rsa
`````

<div class="img_container">
![ssh]({{https://jsom1.github.io/}}/_images/htb_forge_ssh.png)
</div>

We are now officially in and can look for a way up to root!


## 3. Vertical privilege escalation
{:style="color:DarkRed; font-size: 170%;"}

Let's start by looking at this user's permissions:

<div class="img_container">
![sudol]({{https://jsom1.github.io/}}/_images/htb_forge_sudol.png)
</div>

The user can execute python3 and a script called *remote-manage.py* as sudo, without providing a password. Let's have a look at that latter:

````
#!/usr/bin/env python3
import socket
import random
import subprocess
import pdb

port = random.randint(1025, 65535)

try:
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    sock.bind(('127.0.0.1', port))
    sock.listen(1)
    print(f'Listening on localhost:{port}')
    (clientsock, addr) = sock.accept()
    clientsock.send(b'Enter the secret passsword: ')
    if clientsock.recv(1024).strip().decode() != 'secretadminpassword':
        clientsock.send(b'Wrong password!\n')
    else:
        clientsock.send(b'Welcome admin!\n')
        while True:
            clientsock.send(b'\nWhat do you wanna do: \n')
            clientsock.send(b'[1] View processes\n')
            clientsock.send(b'[2] View free memory\n')
            clientsock.send(b'[3] View listening sockets\n')
            clientsock.send(b'[4] Quit\n')
            option = int(clientsock.recv(1024).strip())
            if option == 1:
                clientsock.send(subprocess.getoutput('ps aux').encode())
            elif option == 2:
                clientsock.send(subprocess.getoutput('df').encode())
            elif option == 3:
                clientsock.send(subprocess.getoutput('ss -lnt').encode())
            elif option == 4:
                clientsock.send(b'Bye\n')
                break
except Exception as e:
    print(e)
    pdb.post_mortem(e.__traceback__)
finally:
    quit()
`````

This script opens a listener on a random port (between 1025 and 65535). It is then possible to connect to it and authenticate. If the provided password matches *secretadminpassword*, we can 1) View processes, 2) View free memory, 3) View listening sockets and 4) quit.\\
At this end of the script, we see *pdb.post_mortem(e__traceback__)*: I didn't know what that was, and *pdb* appears to be a debugger for python. I'm not sure what this command does exacty (does it open a debugger?), so let's try this script and then connect to the randomly generated port:

<div class="img_container">
![listener]({{https://jsom1.github.io/}}/_images/htb_forge_listener.png)
</div>

It started a listener on port 16967. Since it is only opened locally, we must connect to it as the current user. To do so, we can simply open a new terminal window and open a second ssh connection (*sudo ssh -i id_rsa user@forge.htb*). From there, we use netcat to connect to the listener:

<div class="img_container">
![Program test]({{https://jsom1.github.io/}}/_images/htb_forge_prog.png)
</div>

Note that I couldn't juste type the password, as the first letter would generate an error and close the connection. Therefore, I just copied/pasted it. Once connected and as expected, the 4 options are proposed. In this case, I pressed 1 to see the processes. This is great, but nothing interesting stands out...

We probably have to do something with the debugger, so let's try to generate an error to see what the command does. But how can we generate an error? Looking at the script, the only thing I see is *option = int(clientsock.recv(1024).strip())*. From Python's doc, we learn that *s.recv(1024)*, or in our case *clientsock.recv(1024)*, means that the socket is going to attempt to receive data, in a buffer size of 1024 bytes at a time.\\
However, 1 character = 1 byte, so I doubt we have to input a >1024 characters string... I then tried typing an unlisted option (like 5, 6, etc...), but nothing happened. However, the debugger opened when we input characters!:

<div class="img_container">
![chars]({{https://jsom1.github.io/}}/_images/htb_forge_chars.png)
</div>

And in the other window, we get the debugger:

<div class="img_container">
![pdb]({{https://jsom1.github.io/}}/_images/htb_forge_pdb.png)
</div>

I looked at the debuggger documentation, and tried a few commands. For example, *l 5, 10* lists lines 1 through 5 of the program. Then, I wanted to see if we can list files of the current directory. To do so, we need to import the *os* module, and then we can use the *listdir()* function.\\
Since we can execute python commands in the root context, we can probably change permissions on */bin/bash* and execute it as root. The Python command for changing file permissions is *os.chmod(path, permissions)*. It didn't work with that syntax, but after some research, we can execute:

````
os.system (‘chmod u+s /bin/bash’)
`````

Once this is done, we can exit the debugger and execute */bin/bash*. Note that *+s* is a special mode called the SETUID (Set owner User ID up on execution) bit. It applies to executables and allows to allocate temporarily (during execution) to a user the rights of the file owner. In other words, it will allow us to run /bin/bash as root. This is very similar to what we did in <a href="/_walkthroughs/Previse">Previse</a>.

<div class="img_container">
![root]({{https://jsom1.github.io/}}/_images/htb_forge_root.png)
</div>

We're root, and we can grab the flag!

<div class="img_container">
![pw]({{https://jsom1.github.io/}}/_images/htb_forge_pwn.png){: height="400px" width = 5000px"}
</div>


<ins>**My thoughts**</ins>

That was my first experience with SSRF and I'm very happy to have learned about it. It's still not 100% clear to me, but it should be enough to make me think about this vector the next time I encounter a similar situation. From what I understand, this vulnerability can be present in applications that offer the possibility to import/read data from a URL. By manipulating this URL or by providing a new one, we try to modify this functionality. In particular, we try to make the server connect back to itself and display internal resources. This box was a good opportunity to practice with SSRF.\\


<ins>**Fix the vulnerabilities**</ins>

As usual, user inputs should be sanitized. In this case in particular, we shouldn't be able to use *ADMIN.FORGE.HTB* instead of *admin.forge.htb*. Once sanitized (convert any input into lowercase), the input should be validated. Another way to prevent SSRF I read about is to use whitelist any domain or address that the application accesses (instead of blacklisting like it is the case here).\\
Regarding privilege escalation, there is no reason for *user* to be able to execute *remote-manage.py* with sudo without providing a password and this should be changed. Those are the two most obvious changes that should be done, but there are probably other things that could prevent the exploitation of this machine.

