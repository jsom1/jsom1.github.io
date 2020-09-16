---
title: "misDIRection"
author: "Me"
date: "July 27, 2020"
output: html_document
---

# misDIRection

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: easy </p>
 <p class="aligncenter">**Type**: MISC</p>
 <p class="alignright">**OS**: Linux</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

**Description**: *During an assessment of a unix system the HTB team found a suspicious directory. They looked at everything within but couldn't find any files with malicious intent.*

I personally started the challenge on my Mac, but switched to Linux when I saw it was about file manipulations. 
From what I read on the forums, it is **recommended to do this challenge on Linux**. The reason is that other OS can unzip and display files differently, which could potentially make the challenge hard or impossible to solve.

After downloading the file, we can unzip it with the command *unzip*. We're asked for a password, and we have to use the one given in the descritpion (*hackthebox*): 

<div class="img_container">
![dl and unzip file]({{https://jsom1.github.io/}}/_images/challenge_misd_dl.png)
</div>

We see that the directory *.secret* is created, and directories are extracted there. Some of them contain values: we see for example that /.secret/V/ contains the value 35.
Let's *cd* to this *.secret* directory. As we just saw, there are many directories, and some of them have values. We can see the structure of the directory with the command *ls -LR*.
The notation looks like hexadecimal (there are 0-9 and A-F folders). Maybe we'll have to decode a hex string ?

After looking at the structure hoping to find something useful, i decided to remove all empty directories. But before doing that, I checked that the command worked by displaying those empty directories:

<div class="img_container">
![list empty dirs]({{https://jsom1.github.io/}}/_images/challenge_misd_empty.png)
</div>

We can now delete them with the command *find . -type d -empty -delete*, and look at the structure with *ls -LR*:

<div class="img_container">
![directory structure]({{https://jsom1.github.io/}}/_images/challenge_misd_LR.png)
</div>

At this point, it took me a while to figure out what to do. Every time I had a string (for example by summing values in directories, or concatenating values), I tried to decipher it with a hexadecimal decoder.
None of those strings returned somehing that looked like a flag... I had to look at the forums to understand that it wasn't hex code (it should have been obvious).
However, one of my idea was correct: every number appear only once (for example, 1 is in directory S, 2 is in F, and so on).
By listing the directories in the right order, we get the following string: SFRCe0RJUjNjdEx5XzFuX1BsNDFuX1NpN2V9.
I used a website (<https://www.tunnelsup.com/hash-analyzer/>) to detect the encryption, which appears to be Base64. Then, I used Cryptii to decode it:

<div class="img_container">
![Flag!]({{https://jsom1.github.io/}}/_images/challenge_misd_flag.png){: height="250px" width = "180px"}
</div>

I obtained the string manually by writing down the directories in the order given by the values 1-27. However, it is possible to do it with a command.

