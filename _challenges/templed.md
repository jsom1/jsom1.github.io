---
title: "Templed"
author: "Me"
date: "September 20, 2020"
output: html_document
---

# Templed

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: easy </p>
 <p class="aligncenter">**Type**: Crypto</p>
 <p class="alignright">**OS**: Any</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

**Description**: *I found the following message in a temple, I had the sensation that they were hiding something. Could you help me discover what it was?*

We start by downloading the file, checking the hash and opening it. It's the following image:

<div class="img_container">
![image]({{https://jsom1.github.io/}}/_images/challenge_templed_img.png){: height="150px" width = "300px"}
</div>

It took me a while to find out that this is an old cipher, known as **the ciphers of the monks**. From Wikipedia, "The system uses a vertical straight line as its main symbol. This symbol is essentially an axis that divides the two-dimensional plane into four quadrants. Each of these four quadrants signifies one of the four digits. The number can then be determined by visual inspection.

The numeral system was invented in the 1300s by French Cistercian monks. It was later replaced by the Hindu–Arabic numeral system. In any case, this numeral system later inspired several shorthands and secret ciphers.

How do we decipher those symbols? We can understand the method with the following image:

<div class="img_container">
![Templar numbers]({{https://jsom1.github.io/}}/_images/challenge_templed_nbrs.png){: height="350px" width = "400px"}
</div>

Then, we know that any number necessarily consists of a central vertical bar: if nothing else is written, then it takes the value 0, otherwise, the quadrant at the top right corresponds to the units, the quadrant at the top left for the tens, the quadrant at the bottom right for the hundreds and the last quadrant at the bottom left for the thousands.

We only really need the digits 1-9 to encode/decode a number/symbol. For example, we see that 1 corresponds to one line in the upper right quadrant. If we mirror this bar in the upper left quadrant, we get 10. If we mirror it in the lower right quadrant, we get 100, and finally, we get 1000 if we mirror it in the lower left quadrant.

Let's look at the first symbol in our image. We see there is something in the tens quadrant (top left), and something in the units quadrant (top right). To decipher it, we see that the bars in the tens quadrant is a mirrored 7, so it's 70. The unit is 2, so the number is 72.\\
By doing the same with each symbol, we get the following numbers:

72 84 66 123 77 48 78 107 115 95 107 78 51 119 33 125

I then used a tool to identify this cipher (eventhough one could recognize it is decimal code), and decoded it with cryptii to get the flag:

<div class="img_container">
![Flag]({{https://jsom1.github.io/}}/_images/challenge_templed_flag.png){: height="350px" width = "450px"}
</div>

