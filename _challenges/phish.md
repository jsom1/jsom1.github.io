---
title: "Easy Phish"
author: "Me"
date: "September 18, 2020"
output: html_document
---

# Easy Phish

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: easy </p>
 <p class="aligncenter">**Type**: OSINT</p>
 <p class="alignright">**OS**: Any</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

**Description**: *Customers of secure-startup.com have been recieving some very convincing phishing emails, can you figure out why?*

I was interested in trying this challenge since phishing is one of the most effective and used technique nowadays. Indeed, humans are often an easy vulnerability, especially when they don't know much about security.\\
When private information has been properly gathered about somebody, it is possible to send very convincing phishing emails.\\
Moreover, I don't know much about it and thought it would be a good way to educate myself.\\

The first thing I did was to visit the website. I was of course being located and got targeted ads. I we click on a category, we get a few ads within it.
At the right of the link, there is an arrow and if we click on it, we see "why this ad?". Looking at it, we see that Google uses information about us to show us ads that could interest us.
Nothing surprising so far. After inspecting the page searching for anything that could help, I had to look at the forum to get a hint...\\
Someone advised to use **nslookup**. I had heard about it before, but didn't know much about it except that it somehow had to be related to **DNS** (Domain Name System).

In a nutshell, DNS translates domain names into IP addresses (and vice versa). Indeed, it is easier for us to remember the website "Google.com" instead of its IP address.
However, computers communicate with IP addresses and not domain names.\\
In some situations, for example when there are issues the name resolution (can't match the name to an IP or vice versa), it can be helpful to look behind the scenes.
This is where **nslookup** (Name server lookup) comes handy. It is natively present on Windows, MacOS and Linux, this is why we can do this challenge on any OS.

It's the first time I use this tool, so let's see what it returns with a basic command:

<div class="img_container">
![nslookup]({{https://jsom1.github.io/}}/_images/challenge_phish_nslookup.png)
</div>

We get the server's IP address. As we said, it is more convenient to remember and use *secure-startup.com* instead of *34.102.136.180*. Note that it is possible to use the IP in the browser:

<div class="img_container">
![Website]({{https://jsom1.github.io/}}/_images/challenge_phish_site.png)
</div>

For some reason, there is nothing on the page this time (there was content when I typed http://www.secure-startup.com/). Let's look at other nslookup options, for example on the site <https://www.geeksforgeeks.org/nslookup-command-in-linux-with-examples/>.
We can get plenty of information with various commands, by specifying the flag "-type=...". Some examples are:

- *-type=soa*: lookup for an soa (start of authority) record. Gives info about the domain, the email address of the domain admin, ...
- *-type=any*: lookup for any record
- *-type=ns*: lookup for an ns (name server) record
- *-type=nx*: lookup for an mx (mail exchange) record
- *-type=txt*: lookup for an txt record. Apparently, TXT records are useful for multiple types of records like DKIM, SPF, etc. **DKIM** (Domain Keys Identified Mail) is an authentication technique by email.
It allows the receiver to verify that an email has been sent and authorized by the domain's owner. This is done by adding a DKIM signature to the email. If the DKIM signature is valid, the email hasn't been modified.
**SPF** (Sender Policy Framework) is an email validation system to prevent spammers from sending messages with our domain's name. Finally, DKIM and SPF are used to build the **DMARC** (Domain-based message authentication, reporting and conformance), an email authentication, policy and reporting protocol.
- ...

A little bit about DMARC, since it seems to be of concern in this challenge and combines both DKIM and SPF... In short, it allows a sender to indicate that their message are protected by SPF and/or DKIM.
It also tells a receiver what to do if neither of those authentication method passes (junk or reject the message). So, DMARC limits or elimininates the user's exposure to fraudulent messages.\\
Thus, DMARC is used to **reduce spam and phishing on the internet**. Maybe we can answer the question *Customers of secure-startup.com have been recieving some very convincing phishing emails, can you figure out why?*: could it be because there is no DMARC ?

To explore this hypothesis, let's see the output of the command with *-type=txt*:

<div class="img_container">
![type=txt]({{https://jsom1.github.io/}}/_images/challenge_phish_typetxt.png)
</div>










<div class="img_container">
![Find ID]({{https://jsom1.github.io/}}/_images/challenge_id_id.png){: height="420px" width = "500px"}
</div>

