---
title: "Pusheen Loves Graphs"
author: "Me"
date: "September 14, 2020"
output: html_document
---

# Pusheen Loves Graphs

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: easy </p>
 <p class="aligncenter">**Type**: Stego</p>
 <p class="alignright">**OS**: Any</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

**Description**: *Pusheen just loves graphs, Graphs and IDA. Did you know cats are weirdly controlling about their reverse engineering tools? Pusheen just won't use anything except IDA.*

This is my first stego challenge, and I have no clue about this topic. However, the description gives very useful information: we have to focus on graphs and IDA. But what's IDA in the first place?\\
From the internet, it's an Interactive Disassembler (IDA). It is used to disassemble software, from machine-executable code to the corresponding assembly source code. As mentionned in the description, this tool is often used in reverse engineering tasks.\\
Hex-rays is the company selling the software, and a free version can be downloaded here: <https://www.hex-rays.com/products/ida/support/download_freeware/>.\\

Since IDA isn't natively available on Kali, I decided to do this challenge on my Mac. The installation of IDA is very easy and isn't shown here. After installing it, we can download the file on Hack the box and verify that the checksum matches:

<div class="img_container">
![DL the file]({{https://jsom1.github.io/}}/_images/challenge_push_dl.png)
</div>

After unzipping it, we can open the file with any text editor. Let's do it with nano:

<div class="img_container">
![view file on nano]({{https://jsom1.github.io/}}/_images/challenge_push_nano.png)
</div>

Besides Pusheen the cat, we see at the beginning of the first line that the file is in *elf* format (Executable and Linkable Format). It is a common standard file format for executables files, object code, shared libraries and core dumps.\\
Because this is a stego challenge, we know that this file contains hidden information. However, after a quick glance at the content of the file, we understand that it's not going to be lying there in plaintext.\\
After trying to find a quick and easy video explaining IDA, I decided to give it a try and figure out by myself. We start the software, and select *Disassemble a new file* as follows:

<div class="img_container">
![Starting IDA]({{https://jsom1.github.io/}}/_images/challenge_push_ida.png){: height="250px" width = "200px"}
</div>

Then, we open the Pusheen file and get some error messages. Apparently, we can skip those until we get the following warning:

<div class="img_container">
![Node error]({{https://jsom1.github.io/}}/_images/challenge_push_mess.png){: height="380px" width = "450px"}
</div>

It says it's switching to text mode, but from the description, we know that Pusheen only loves graphs. So, let's try to change this parameter to stay in graph mode:

<div class="img_container">
![Config change]({{https://jsom1.github.io/}}/_images/challenge_push_conf.png){: height="300px" width = "250px"}
</div>

Then, it brings us back the the main window and displays the graph correctly. In the lower left window, there's a graph overview, and within it, we can read *fUn_w17h_CFGz*:

<div class="img_container">
![Flag]({{https://jsom1.github.io/}}/_images/challenge_push_flag.png){: height="380px" width = "450px"}
</div>

And that's the flag. Unfortunately, I'm not sure what *CFGz* means. It could be a reference to a game called Crossfire (referred to CFGZ for some reason), or a reference to Context-free Grammars, which are studied in fields of theoretical computer science, compiler design and linguistics. CFG's are used to describe programming language, and parser programs in compilers can be generated automatically from them.
