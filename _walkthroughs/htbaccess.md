---
title: "Hack the Box Access"
author: "Me"
date: "March 14, 2020"
output: html_document
---

# Hack the Box invite code

<div id="bckgd">
<p>Spoiler!</p>
</div>

Note: It is not necessary to use Kali to get the invite code. I got mine from my Mac.

Go to [https://www.hackthebox.eu/invite](https://www.hackthebox.eu/invite): it says "Feel free to hack your way in". First, I naively tried a few classic passwords such as *admin*, *1234*, *root*, *toor*, and more.
Without surprise, it didn't work. The next thing to do was to have a look at what happens when we submit a password: right click anywhere on the page,
and click on *inspect*. Then, go in the *Network* tab.  

<div class="img_container">
![htb inspect]({{https://jsom1.github.io/}}/_images/htb_inspect.png){: height="5700px" width = "620px"}
</div>

We immediately see a bunch of scripts, and there is one called *inviteapi.min.js*: at the end of the script, there are
a few commands and among them, *makeInviteCode* looks interesting. So, let's check what this function does by typing it in the *Console* tab.

<div class="img_container">
![htb make inv]({{https://jsom1.github.io/}}/_images/htb_makeinv.png){: height="5700px" width = "620px"}
</div>

It returns an object containing an encrypted (BASE64) string that we can decipher with Cryptii ([https://cryptii.com/](https://cryptii.com/)) for example:

<div class="img_container">
![htb cryptii]({{https://jsom1.github.io/}}/_images/htb_cryptii.png){: height="250px" width = "250px"}
</div>

The message tells us to do a POST request to generate an invite code. We could do it within a terminal, but I downloaded Postman
a while ago and used it instead. We just send an empty POST request to the given address:

<div class="img_container">
![htb postman]({{https://jsom1.github.io/}}/_images/htb_postman.png){: height="5700px" width = "620px"}
</div>

In return, we got another encrypter string. Once again, I used Cryptii to decipher it:

<div class="img_container">
![htb code]({{https://jsom1.github.io/}}/_images/htb_code.png){: height="250px" width = "250px"}
</div>

And this is it, we have our invite code!
