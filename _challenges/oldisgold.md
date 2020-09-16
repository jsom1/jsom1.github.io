---
title: "0ld is g0ld"
author: "Me"
date: "July 14, 2020"
output: html_document
---

# 0ld is g0ld

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: easy </p>
 <p class="aligncenter">**Type**: MISC</p>
 <p class="alignright">**OS**: Any</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

**Description**: *Old algorithms are not a waste, but are really precious...*

This challenge is rather easy and straightforward. The first thing to do is of course to download the file and unzip it with the given password *hackthebox*: 

<div class="img_container">
![dl and unzip file]({{https://jsom1.github.io/}}/_images/challenge_oig_dl.png)
</div>

The file is a PDF, and we can open it with the command *xdg-open* as follows:

<div class="img_container">
![open PDF]({{https://jsom1.github.io/}}/_images/challenge_oig_xdg.png)
</div>

The file is protected by a password. So, let's look on the internet if there is any tool to crack it. We quickly find the tool **pdfcrack**. I had an error when installing it, so I had to update first:

<div class="img_container">
![update and download pdfcrack]({{https://jsom1.github.io/}}/_images/challenge_oig_update.png)
</div>

Then, pdfcrack was installed successfully. The command *pdfcrack* shows the available options. I first tried to use pdfcrack without giving it any wordlist, but stoppd it after ~20 minutes because my CPU was heating too much. I then tried with the common rockyou file and got something:

<div class="img_container">
![rockyou]({{https://jsom1.github.io/}}/_images/challenge_oig_rockyou.png)
</div>

We should now be able to open the PDF. For some reason, I couldn't open the file on Kali and used my Mac instead. Here is the content:

<div class="img_container">
![PDF1]({{https://jsom1.github.io/}}/_images/challenge_oig_pdf1.png)
</div>

I have no clue who this person is. At the bottom of the page, there is something else:

<div class="img_container">
![PDF2]({{https://jsom1.github.io/}}/_images/challenge_oig_pdf2.png)
</div>

This strongly looks like Morse code, so we can check this assumption with any online tool (just type morse code translator). It returned "R1PSAMU3LM0RS3" (RIP Samuel Morse). It makes sense, the picture was in fact Samuel Morse! And this is also the flag for this challenge.


