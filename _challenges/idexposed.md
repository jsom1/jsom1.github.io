---
title: "ID exposed"
author: "Me"
date: "September 17, 2020"
output: html_document
---

# ID exposed

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: easy </p>
 <p class="aligncenter">**Type**: OSINT</p>
 <p class="alignright">**OS**: Any</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

**Description**: *We are looking for Sara Medson Cruz's last location, where she left a message. We need to find out what this message is! We only have her email: saramedsoncruz@gmail.com*

We only have a gmail address, and we want to gather information from it. From the title of the challenge, we should be able to retrieve her ID from the address. I started by throwing keywords into Google and read a lot of articles about OSINT, and found one that could be useful:

<div class="img_container">
![Google search]({{https://jsom1.github.io/}}/_images/challenge_id_osint.png)
</div>

This looks promising, as it says we can get an exact location on Google Maps. In the article (<https://medium.com/week-in-osint/getting-a-grasp-on-googleids-77a8ab707e43>), it is said that we have to add the email in our contact list. To do this, we go to *contacts.google.com* and add it:

<div class="img_container">
![Add new contact]({{https://jsom1.github.io/}}/_images/challenge_id_contact.png)
</div>

Then, we have to open the inspector tab and refresh the page. At this point, we can either look at the Network tab, or directly in the elements.\\
I tried with the network tab, but couldn't find the information. I looked at the *batchexecute* requests (as explained in the article), but couldn't find Sara's ID.\\
Fortunately, the other option worked: in the Elements tab, we're supposed to search for the **data-sourceid**. I Found it, but the ID didn't look like it was supposed to (apparently, it consists of a 21 character long decimal numbers, starting with "10" or "11"). The one I had for data-sourceid was 16 character long, starting with "6f". So, I searched everywhere on this page for something that would look like an ID, and found the following:

<div class="img_container">
![Find ID]({{https://jsom1.github.io/}}/_images/challenge_id_id.png){: height="390px" width = "460px"}
</div>

It starts with "11" and is 21 digits long. It appears many times on the page and often with references to Sara. So, let's suppose it's her Google ID and look at the rest of the artice. It is said that we can get an exact location on Google maps.\\
To do this, we enter **https://www.google.com/maps/contrib/{userID}** in our browser:

<div class="img_container">
![Locate ID]({{https://jsom1.github.io/}}/_images/challenge_id_command.png)
</div>

And this locates the ID!

<div class="img_container">
![ID on the map]({{https://jsom1.github.io/}}/_images/challenge_id_map.png){: height="400px" width = "475px"}
</div>

There is a menu on the left where we see if the user imported pictures or wrote a feedback. There is no pictures, but the flag is in a feedback ! So, we know that Sara posted this message from Sao Paulo!

I really liked this challenge since this might come handy in real life situations. I didn't know we could get this information just by adding the address in our contact list.


