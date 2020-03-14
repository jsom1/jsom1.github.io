---
title: "Hack the Box Access"
author: "Me"
date: "March 14, 2020"
output: html_document
---

**!!Spoiler!!**
{:style="color:Red; font-size: 200%;"}

# Hack the Box invite code

I got my invite code from my Mac, so there is no need to use Kali.\\ 
First, go to [https://www.hackthebox.eu/invite](https://www.hackthebox.eu/invite).
It says "Feel free to hack your way in". First, I naively tried a few classic passwords such as *admin*, *1234*, *root*, *toor*, and more.
Without surprise, it didn't work. The next thing to do was to have a look at what happens when we submit a password: right click anywhere on the page,
and click on *inspect*. Then, go in the *Network* tab.  

<div class="img_container">
![htb inspect]({{https://jsom1.github.io/}}/_images/htb_inspect.png){: height="500px" width = "550px"}
</div>

We see a script called *inviteapi.min.js*: at the end of the script, there are
a few commands and among them, we see *makeInviteCode*. So, let's check what this function does by typing it in the *Console* tab.

<div class="img_container">
![htb make inv]({{https://jsom1.github.io/}}/_images/htb_makeinv.png){: height="500px" width = "550px"}
</div>

There is an encrypted (BASE64) string that we can decipher with Cryptii for example:

<div class="img_container">
![htb cryptii]({{https://jsom1.github.io/}}/_images/htb_cryptii.png){: height="250px" width = "250px"}
</div>

The message tells us to do a POST request to generate an invite code. We could do it within a terminal, but I downloaded Postman
a while ago and used it instead. We just send an empty POST request to the given address:

<div class="img_container">
![htb postman]({{https://jsom1.github.io/}}/_images/htb_postman.png){: height="500px" width = "550px"}
</div>

In return, we got another encrypter string. Once again, I used Cryptii to decipher it:

<div class="img_container">
![htb code]({{https://jsom1.github.io/}}/_images/htb_code.png){: height="250px" width = "250px"}
</div>

And this is it, we have our invite code!
