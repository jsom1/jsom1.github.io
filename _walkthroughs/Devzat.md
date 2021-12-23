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

**Ports/services exploited:** ?\\
**Tools:** dirb, gobuster\\
**Techniques:** ?\\
**Keywords:** Golang (Go)?

**In a nutshell**: ?

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
![site]({{https://jsom1.github.io/}}/_images/htb_dz_site1.png)
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
![github page]({{https://jsom1.github.io/}}/_images/htb_dz_gh.png)
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
![Inspect page]({{https://jsom1.github.io/}}/_images/htb_dz_inspect.png)
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

The location where our input is being included affects the type of character we have to use. If it's added between *div* tags, we have to include our own script tags, and we're not able to use "<" and ">" in our payload. If the payload is inserted into an existing JavaScript tag, we might only need quotes and semicolons.


## 3. Vertical privilege escalation
{:style="color:DarkRed; font-size: 170%;"}



<ins>**My thoughts**</ins>


<ins>**Fix the vulnerabilities**</ins>

