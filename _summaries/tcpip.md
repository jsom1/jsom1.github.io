---
title: "TCP/IP"
author: "Me"
date: "July 23, 2020"
output: html_document
---

# TCP/IP
{:.no_toc}

This is a resumee of the book “Learn how TCP/IP networks work”. The book is only available in french, but the resumee is in english. It is excellent to understand the basics of networking. In particular, it explains exactly how data is transmitted over the internet, the most important protocols, and explains some well known attacks.

The chapters are the following:

1. TOC
{:toc}
{:style="color:black; font-size: 150%;"}

# First part: communication on a local network

In this part, we see the different elements that make up a network and how the machines can communicate.

## The OSI model

The OSI model was born in 1984 and is a standard for how computers should communicate with each other. For example, if we wanted to make a toaster communicate with a dishwasher, we would have to follow the OSI model. One of the most important concept is that the communication requires several layers (so the OSI model consists of different layers, where each of them is responsible for a specific task).

The OSI model has 7 layers:

- Layer 1 or **physical layer**: offers a transmission medium for communication. Associated hardware: the **hub**.
- Layer 2 or **data link layer**: connects machines together on a **local network** and detect transmission errors. Associated hardware: the **switch**.
- Layer 3 or **network layer**: interconnects different networks together and breaks down packets. Associated hardware: the **router**.
- Layer 4 or **transport layer**: manages connections of applications and make sure the connection is stable.
- Layer 5 or **session layer**: controls the conversations between different computers (this layer sets up, manages and terminates connections between machines).
- Layer 6 or **presentation layer**: formats or translates data for the application layer, can also handle encryption and decription required by the application layer.
- Layer 7 or **application layer**: at this layer, both the end user and the application layer interact directly with the software application. Associated hardware: the **proxy**.

The layers 1-4 are referred as the **network layers**. Note that this model is theoretical: the real model behind Internet today is TCP/IP, and this latter doesn't use the layers 5 and 6 this way.\\
If it's theoretical, why does it exist ? It gives manufacturers of network devices instructions on how to build those devices so that they can communicate with other devices from different manufacturers. We don't really think about it when we write an email from a Mac and send it to a Windows user, but there are many protocols behind that allow it.

There are 2 very important rules in the OSI model:

- Each layer is **independent**: a layer can be modified without breaking the whole communication schema.
- Each layer can only communicate with an adjacent layer: when we browse the internet, send an email, or communicate in any way on the internet (layer 7), the data go through all the layers subsequently.

Let's see those layers in more details.

## Layer 1: connecting machines

A medium is required to communicate. For example, we need an **ethernet cable** or air (for Wi-Fi) to connect to the internet. So, layer 1 transmits electrical signals (0's and 1's). Nowadays, the optical fibre is common and transmits light instead of electrical signals. Those 0's and 1's are carried by the different transmission media (described in the book).
To interconnect several machines on a network, we need a connection device: the hub. The hub has a weird way of working. Imagine 5 machines (A B C D E) connected to a hub: if A wants to talk to C, it will send data to the hub. However, this latter can't read and will forward it to all the machines, supposing that one of them is the right one. The machines B, D and E will detect the data isn't for them and will discard it, whereas C will read it. So, the hub isn't the best tool when it comes to data privacy.

There are different network topologies: bus, ring, mesh, star and hybrid. The pros and cons of each are not discussed here, and let's focus on the star topology since bus and ring are rare nowadays. In this setup, all the communications go through a central point. A machine sends information with the recipient's name to the central point, which redirects it to the right machine.\\
The number of machines on this topology depends on the capacity of the central point to manage information. Nowadays, switches can manage thousands of machines. Also, we can link many central points together to create an almost infinite network. 

## Layer 2: allow communications between machines

The role of the second layer is to connect machines on a local network, and allow them to communicate. It also detects transmission errors (it detects but doesn't correct them!).

### An identifier, the MAC address
{:style="color:DarkRed; font-size: 170%;"}

In order to communicate with a specific machine on a network, we must be able to identify it precisely. The **MAC** address is a unique identifier that corresponds to the address of the network card. Each machine on earth has it own unique MAC address. It is written in hexadecimal, for example 00:23:5e:bf:45:6a.
We see it is coded on 6 bytes, where 1 byte represents 8 bits. So, 1 byte = 8 bits = 2^8 = 256 possible values -> a byte can take any value between 0 and 255.
Because the MAC address is coded on 6 bytes (6x8 = 48 bits), it can take 2^48 = 281,474,976,710,656 values. This gives a lot of potential MAC addresses.\\

A network card manufacturer buys a part of MAC addresses to make sure it is unique. The 3 first bytes represent the manufacturer.

There is one special MAC addresss, where all the bits are set to 1 (ff:ff:ff:ff:ff:ff): it is a univeral address called the **broadcast** address. It identifies any network card. It can be used to send a message to all the network cards of machines on a network. Every machine receiving a message containing the broadcast address will suppose it can read the message.

Now that we can identify machines, we need a language.

### A protocol, Ethernet
{:style="color:DarkRed; font-size: 170%;"}

In fact, Ethernet is the language used by the machines to communicate. Because we communicate with different machines (which have different OS such as Mac OS, Windows, Linux, ...), we need a common language to understand each other. We will call it a **protocol**. We know that 0's and 1's are transmitted, and information look like 00110010111100011001000... Without a protocol defining the meaning of this information (the order of bits, ...), it doesn't mean anything.

We need at least 3 things in a message: the address of the sender, the one of the receiver, and the message itself. For example, we could say that the first 48 digits represent the MAC address of the sender, the 48 next the one of the receiver, and then the message (it's not the real format).\\
The protocol defines the format of messages sent over the network. More precisely, we call the message a **frame**. 

So, what's the real format of a frame ? Which MAC address comes first, sender or receiver ? We can answer this question by imagining we're the receiver: it's more interesting to know the addresss of the receiver first, because we immediately know if the frame is for us or not. We read it if it's for us, discard it otherwise.\\
What's next ? After the receiver comes the sender address. And then the message ? No. Let's imagine we're sending information. If we think about the OSI model, we go through all the layers from the top (7) to the bottom (1). We went through layer 3 before layer 2: layer 3 can indicate to layer 2 what protocol was used in layer 3.\\
But why does that matter ? Because when the information is received by the layer 2 of the receiver, it must send the frame further to the right protocol of layer 3. So when we send information, it goes from layer 7 to 1. When we receive information, it goes from layer 1 to 7.

At this point, the structure of the frame is: DST MAC address, SRC MAC address, Layer 3 protocol, Message. We saw that layer 2 can also detect errors, so we are going to add that after the message and name it **CRC** (cyclic redundancy check). The CRC is a number (hash) that is specific to each message: when a machine sends information, it computes the CRC and appends it to the packet. The receiver performs the same computation and compares the value. If it's the same, the message is the same. If it's different, there was an error during transmission.

The ethernet frame is now complete and its structure looks like the following:

<div class="img_container">
![Layer 2 frame structure]({{https://jsom1.github.io/}}/_images/tcpip_L2packet.png)
</div>

The size of the bold elements never changes and constitute the **header** (here, the ethernet header). Its size is always 18 bytes. Only tee size of the data part varies. What's the minimum and maximum size of a frame ?\\
The minimum size for an ethernet frame is **64 bytes**, the maximum is **1518 bytes**.

## Layer 2 hardware: the switch

The switch allows us to interconnect several machines. We sometimes hear about a *bridge*, which is simply a switch with 2 ports. A switch is a box containing different ports in which we can plug cables. We plug our machines (computers, printers, ...) to the switch, and can also plug other switches to ours.\\
How does the switch know where to send a frame ? It simply uses the DST MAC address found in the frame's header.\\

### The CAM table
{:style="color:DarkRed; font-size: 170%;"}

The switch contains a **CAM** table (Content Addressable Memory) that links the switch ports with the MAC addresses of the machines connected to it. So, the switch reads the DST MAC address in the frame's header and looks up in its table to know on which port to forward it.

The CAM table is dynamically updated: the switch "learns" to which port is plugged each machine by seeing frames pass. For example, let's imagine 3 computers (21, 22, 23) freshly connected to a switch whose CAM table is empty.\\
Computer 21 (plugged on port 1) sends a frame to computer 23 (plugged on port 3): the frame arrives at the switch. This latter now knows that computer 21 is linked to port 1, so it updates the table. However, it doesn't know where to forward the frame at this point (it doesn't have the destination address in its CAM table). So, it sends it to all the computers (it's not broadcast, because the destination address isn't the broadcast one!). Computer 23 will receive it and be able to reply to computer 21. The switch can update its CAM table (it now knows that computer 23 is on port 3). The table is updated this way each time the switch sees a frame pass. 

Is the CAM table going to grow indefinitely ? No, thanks to the **TTL** (Time to live). The idea is that an information is valid for a certain amount of time, and then get removed from the table. Below is a representation of the CAM table for the previous example:

<div class="img_container">
![CAM table]({{https://jsom1.github.io/}}/_images/tcpip_cam.png){: height="250px" width = "180px"}
</div>

The switch has 2 entries in its table and the second one is more recent (higher TTL). The first entry will be removed in 91 seconds if computer 21 remains silent (as well as 23). If it speaks, the TTL will be resetted to 120s.\\

Knowing how a switch works, we can imagine how we could compromise it. There are 2 cases:

- Send a ton of frames to non-existing MAC addresses: because the switch doesn't know where to send them, it will send them to all the active ports, resulting in its saturation.
- Send a ton of frames with different source MAC addresses: the switch will update its table progressively. The bigger it gets, the slower the switch reads it, resulting in an increase of the latency. When the switch is saturated and doesn't have time to read the CAM table, it will send frames on all ports, revealing all its traffic (enventhough we'll see better techniques to analyze the traffic of a switch).

Compared to a hub (bus topology), a switch can isolate conversations. It can also store one or many frames. This could happen if computers 21 and 23 send a frame to computer 22 at the same time, because computer 22 only has 1 receiving pair of wires. This way of working allows what is called **full-duplex**: data can be transmitted in both directions on a signal carrier at the same time (one machine sends data, another receives data -> this doubles to debit). We say that the network cards are in full-duplex mode. In half-duplex, only one machine talks at a time. A network card can determine automatically if it has to work in full- or half-duplex. In short, it will work in half-duplex if it's connected to a hub, and in full-duplex when connected to a switch. If we plug a hub in a switch, the switch has to adapt (a hub only does half-duplex). But not the whole switch, only the port to which the hub is connected.

### VLANs
{:style="color:DarkRed; font-size: 170%;"}

A switch can do more than forwarding frames on the right ports; it can create **VLANs** (Virtual Local Area Network). This is done by separating the ports of the switch. After that, they can't communicate anymore (like if we had several switches). Imagine 6 computers (1-6) and a switch with 10 ports. We can create a VLAN for computers 1-3 on ports 1-3, and another for computers 4-6 on ports 8-10. Computers 1-3 can communicate together, but they can't reach computers 4-6 (and vice-versa).\\
What's the point ? It can be useful to separate networks (for example the student network, the teachers network and the administration network). This is more secure, and we can install big switches (256 ports) instead of many small ones.\\
Note that it is possible to pass from a VLAN to another. It's called **VLAN hopping**, but we're not supposed to do it!

## Layer 2 in practice

Where can we find the information we just saw, that is the information about the network card? It depends on the OS, and we'll see how to find it on Windows and Linux.

- Windows: there are 2 ways to access it, via the command line or the GUI. For the command line, we get a prompt by searching *cmd*. Then, the command is **ipconfig /all**. Among all the information, we see the MAC address (on 6 bytes). We can also access a GUI menu via the settings -> network. Then local network -> properties -> configuration -> advanced. Here we find all the information regarding the network card. It is usually possible to **modify the MAC address**.
- Linux: there's only the command line, and the command is **ifconfig**. The MAC address is written after the *HWaddr*. If the network card supports it, it is also possible to change it with a command like *ifconfig eth0 hw ether 00:01:02:03:04:05*. 

### Exercise 1: switching loop
{:style="color:DarkRed; font-size: 170%;"}

There's nothing to do in the exercise, it's just about understanding what a switching loop is. Let's imagine the following situation: there are 3 switches connected together, and a few machines connected to each of them. The setup looks like the following:

<div class="img_container">
![CAM table]({{https://jsom1.github.io/}}/_images/tcpip_lan.png){: height="400px" width = "500px"}
</div>

**Problem**: switch 5 broke down, so the machines of switches 1 and 9 can't communicate. How can we repair it, and how can we get a working network, even if a switch breaks down?\\
The instinctive answer would be to link the switches 1 and 9. It works, but one hour later, nobody can reach the internet or communicate with each other anymore.\\
What happened? We just created a switching loop, which offers 2 ways to reach a destination. When we send a frame to a machine, the switch will use both ways and the frame will arrive twice at the destination. This becomes a problem when broadcasting: our broadcast will be sent on both ways, and when it arrives at the next switch, it will be sent on the 2 possibles ways again, and the same at the next switch, and so on... This happens until a point where the switches have too many broadcasts to treat and get saturated. This is what we call a **broadcasts storm**.\\
This phenomenon is very powerful and can take down the biggest networks. So, how can we solve the initial problem?!

**Solution**: we must use the Spanning Tree Protocol (STP), a layer 2 network protocol. In short, it builds a loop-free logical topology for ethernet networks by following a few steps: election of the root switch, determination of the root port of each switch, determination of the designated port on each segment, blocking of other ports.

### Exercise 2: The network simulator
{:style="color:DarkRed; font-size: 170%;"}

The simulator can be downloaded at <http://www.sopireminfo.com/indexEn.html> (15 days trial version). It's a good idea to look at the documentation before starting.

**Hubs**\\
We'll start by creating a network with 3 hubs connected together, without using the rightmost port (similar to the previous image, but with hubs instead of switches). We then add one machine on each hub. Red ports mean that our cables are not configured as they should.\\
Now, we connect the 2 hubs that weren't connected, right click on a network card and select *send a frame* (broadcast). What happens? There's an error message indicating the presence of a loop.\\
Remove a cable, and try sending a frame again in broadcast and then in **unicast** (send a frame to a unique destination. It's the classical use of networks). What's the difference between broadcast and unicast ? There is no visible difference. Why? Because we're using a hub, and frames are inevitably sent to everyone, like in broadcast. 

**Switch**\\
Let's now create a network with a single switch and 3 machines. Right click on the switch and empty the MAC/port table (CAM). We then send a frame from machine 1 to another. What will happen? The switch will send the frames to all the machines, because its CAM table is empty.\\
If we resend the same frame, what will happen? The same thing. The switch knows the machine 1 is on port 1, but it still doesn't know on what port is the destination machine.\\
We'll now see how the switch works by following what's happening: click on *no traced node*, select sw1, that we pass in the traced nodes. Send a frame from a machine to another and observe the operating steps of the switch.\\
Now we get into manual mode and send a frame from a machine to another. We must decide what the switch has to do. That's it for the exercises.

# Second part: communication between networks

In this part, we see the layer 3 of the OSI model and everything that allows to identify networks and communicate between them.

## Layer 3: interconnect networks

We now know how machines communicate on a local network. We'll see how they can communicate with machines outside of that network. This is the flagship chapter of the course because it concerns **THE internet protocol, IP**.

Layer 3 is the network layer, allowing us to communicate between networks. Thus, its role is to interconnect networks. 

How can we send a message to a network to which we aren't directly connected, and across the world?\\
All networks are linked together, as a chain. The internet is a huge set of networks linked together. For example if we want to reach Google's network, our messsage goes through several intermediate networks. Also, there are many different possible paths to reach it. The layer 3 allows us to reach any other network on the internet.

This can be seen on Linux with the command **traceroute**: for example, *traceroute www.google.com* shows through which machines our message pass to reach Google's network. In the output, each line represents a machine that we met on the internet. We also see the time required to reach that machine. In other words, we see how many networks we went through (each line is a machine which is part of a network).

### An identifier, the IP address
{:style="color:DarkRed; font-size: 170%;"}

So far, we only know the MAC address, which is used on our local network.
The questions that arise with the interconnection of networks is: how can our network be recognised?  How can we identify networks? Do they have a name or an address? If we need an address for the network and one for my machine, do we need 2 addresses in layer 3?\\
The IP address is the answer to all those questions.

In fact, the IP address is the **address of the network and the one of the machine**. A part of the address represents the network, and the other represent the machine on that network.\\
An IP address is coded on **32 bits** (4 bytes). To facilitate the reading and writing for humans, we chose to write addresses with decimal notation. This latter splits the 4 bytes in 4 decimal digits going from 0 to 255. Example: 192.168.0.1.\\
So, the "smallest" address is 0.0.0.0 (when all bits are set to 0), and the "biggest" is 255.255.255.255. (all bits to 1). Note that hardware manipulate IP addresses in binary notation.

We know that a part of the address represents the network, and the other the machine. How can we know which part represents what? We need the **subnet mask**. The subnet mask is added to the IP address (it's always the case), and it's that mask that indicates which part is what.\\
The **bits set to 1 in the mask represent the network part of the IP addresss**. So, the bits set to 0 represent the machine part. Let's look at the following example: we have the IP addresss 192.168.0.1 with the mask 255.255.0.0.\\
In binary, those addresses are written:\\
255.255.0.0       11111111.11111111.00000000.00000000\\
192.168.0.1       11000000.10101000.00000000.00000001\\
The network part of the address is 11000000.10101000 (192.168), and the machine part is 00000000.00000001 (0.1).\\
In this example, the separation occurs in the middle of 2 bytes, but it's not always the case. Let's look at another, more complicated example: 192.168.0.1 with the mask 255.255.240.0. In binaries, we get:\\
255.255.240.0     11111111.11111111.11110000.00000000\\
192.168.0.1       11000000.10101000.00000000.00000001\\
We see the split occurs within the 3rd byte (11110000).
So, the network part is 11000000.101010000.0000 and the machine part is 0000.00000001. Note that because the split occurs in the middle of a byte, we can't write it in decimal and have to keep it in binary.\\
In a binary mask, 1's must be on the left and 0's on the right; they cannot mix. This way, we always find the same values for the bytes of a mask:\\
00000000 -> 0\\
10000000 -> 128\\
11000000 -> 192\\
11100000 -> 224\\
11110000 -> 240\\
11111000 -> 248\\
11111100 -> 252\\
11111110 -> 254\\
11111111 -> 255\\
With this is mind, we immediately see that the mask 255.255.128.0 is correct, whereas 255.255.173.0 isn't. The mask 255.128.255.0 is also wrong because it mixes 0's and 1's.

Now that we know how the mask works, we can use to find the address range associated to it, from the lowest to the largest. Let's see how we calculate the range for the address 192.168.0.1 with the mask 255.255.240.0.\\
The first thing to do is to transform those addresses into binary:\\
255.255.240.0     11111111.11111111.11110000.00000000\\
192.168.0.1       11000000.10101000.00000000.00000001\\
At this point, we know that 11000000.10101000.0000 represents the network part, and 0000.00000001 the machine part.\\
**All machines belonging to the same network have something in common: all the bits of the network part are identical**. If two machines have addresses with different networking part, they are not on the same network.\\
In our case, all the machines on our network will have the same network part, which is 11000000.10101000.0000. However, the bits of the machine part of the address can vary. We see here that we have many possibilities, such as:\\
11000000.10101000.00000000.00000000\\
11000000.10101000.00000000.00000001\\
11000000.10101000.00000000.00000002\\
11000000.10101000.00000000.00000003\\
...\\
11000000.10101000.00001111.11111110\\
11000000.10101000.00001111.11111111\\
We can find all the possibles addresses of the network by varying the bits of the machine part.\\
**Definition**: the 1st address of the network is the one that has all bits set to 0 in the machine part. The last address is the one that has all bits set to 1 in the machine part.\\
How many addresses are there in our network ? To answer this question, we only need to know the number of bits of the machine part. In our case, we see that 12 bits can vary. But it's even simpler: since the machine part is defined by the mask, **the number of machines available in a network depends on the mask**.\\
Nb. of addresses in a network = 2^Nb. of zeros in the mask.\\
In our case, we have 2^12 = 4096 possible addresses.

There are 2 particular addresses in the available address range: the first and the last:

- The first address of a range is the address of the network itself. Therefore, it cannot be used for a machine.
- The last address of a range is the broadcast address and cannot be used for a machine either. It is used to identify all the machines on the network: when we send a message to the broadcast address, the message will be received by every machine on the network.

So far, we've seen that one IP address with a given mask gives information on both the network AND a machine. A few interesting facts for a given IP address and mask:

- An address ending with 255 isn't necessarily a broadcast address (example: 10.0.0.255/255.255.254.0 -> machine address)
- An address ending with 0 isn't necessarily a network address (example: 192.168.1.0/255.255.254.0 -> machine address)
- All the broadcast addresses are odd, all the network addresses are even.

There are some particulars addresses that are reserved and can't be used on the internet. Those addresses are listed in the RFC 1918 (Request For Comment) at <https://tools.ietf.org/html/rfc1918>. RFCs describe how everyting on the internet work. For example, RFC 791 describes the IP protocol, and RFC 1149 proposes standard for the transmission of IP datagrams on avian carries (it was an April fools joke).

The RFC 1918 lists address ranges that have a particular use. Those addresses are reserved for a private use. If we create a network (at home or work), we have to use those addresses.\\
This is done to make sure that we don't use a network (let's say 92.243.25.0/255.255.255.0) that already exists. One could think that it doesn't matter if it's a private network, but this is not the case. Let's imagine that we want to go on Facebook, and that this latter uses the address 92.243.25.239. We see that the address belongs to our network: when our machine tries to reach that address, it thinks that the machine is on our network and can't reach it.\\

So, what address can we chose? By looking at the RFC 1918, we see that the ranges are:

- 10.0.0.0/255.0.0.0
- 172.16.0.0/255.240.0.0
- 192.168.0.0/255.255.0.0

For example, we could chose 10.0.0.0/255.255.255.0 or 192.168.0.0/255.255.255.0: those addresses are free and we'll be able to reach any website.

We're now going to see how to split address ranges.

## Split an address range

### Base method

Be able to split an address range correctly is important to quickly identify a network.\\
But what does splitting an address range mean? Let's imagine we're admin of a network in an IT school, where students learn how networks work and how to exploit their vulnerabilities. We configured the machines so that they all belong to the same large network 10.0.0.0/255.255.0.0. The problem is that on this network, there are students, teachers and the administration... And we suddenly realize that some students were able to access a teacher's machine and change their grades.\\
What could we do to prevent that ? We can split the large address range in different smaller subnetworks. We'll have one for the students, one for the teachers and one for the administration. 

So far, we have written the masks on 4 bytes (in decimal notation). This quickly becomes annoying when we have many masks to write. Thankfully, some people "created" the **CIDR notation**. We will see later what CIDR is, but now let's see how to use it. Recall that a masks is written on 32 bits, and 0's and 1's aren't mixed within a mask. This means that if we know the number of 1 in a mask, we know it completely.\\
Example: the mask 255.255.255.0 is written /24 in CIDR notation (because 255.255.255.0 is 11111111.11111111.11111111.00000000 in binary, and there are 24 bits set to 1). Instead of writing 192.168.0.1/255.255.255.0, we can write 192.168.0.1/24.

**Example - exercise**: let's imagine a company owning the address range 10.0.0.0/16. The company consists of 1000 technicians, 200 sale people and 20 directors. We're going to split the original range into 3 smaller subnets.

First, we check if our original range contains enough addresses for 1220 people. The mask has 16 bits to 1, therefore it also has 16 bits to 0. The number of available addresses is 2^16 = 65536, which is more than enough.

#### Determine the masks

We know how many addresses we want in the smaller subnets, and the previous formula gives us the relationship between the number of addresses and the number of zeros in the mask.\\
For the 1000 technicians, we need a subnet with at least 1000 addresses (in fact 1002 since the first and last addresses cannt be used!). We can deduce the required number of 0's in the mask with the formula. 1000 < 2^10, so if we set 10 bits to 0 in the mask, we can identify 2^10 = 1024 addresses. Also, the mask can be noted /22 in CIDR notation.\\
We proceed with the same logic for the 200 sales people. 200 < 2^8, so the mask will be /24.\\
Finally, the mask for the directors will be /27.

What do we do with those masks now? We must find the associated address ranges, and we have many possibilities given the large range we started with...

#### Chose address ranges

We have our large range 10.0.0.0/16 consisting of 65536 addresses, and we would like to find a range of 1024 addresses within it for our technicians. The simplest choice is to start with the lowest address (but not the only one). So, we start the range at the address 10.0.0.0. We can already identify the technician network with the pair 10.0.0.0/22, but we also need to know the last address of the range... We know how to do it. We usually started by transforming the pair into binary, but we see here that only one of the four bytes interests us: the one where the split occurs (3rd byte, 252 because we the mask is /22, that is 11111111.11111111.11111100.00000000, or 255.255.252.0). Therefore, we can focus on the 3rd byte.\\
Mask: 252 -> 11111100\\
Address: 0 -> 00000000\\
From the mask, all the addresses of the machines on this network will start with 000000 on the third byte. Therefore, the last address will be the one where all the bits are set to 1 in the machine part, that is 00000011 on the third byte (3 in decimal), and 11111111 on the fourth one (255 in decimal). This gives us the last address of the technicians range: 10.0.3.255.

Now, we need a range for the 200 sales people. However, we know the technicians are already using the range 10.0.0.0 - 10.0.3.255. We could decide to start the sales people range right after it, that is 10.0.4.0. We had the mask, so we can identiy this range with the pair 10.0.4.0/24. What's the last address of this range? This time, the split occurs between 2 bytes, so it's going to be easier: the last address will be 10.0.4.255.

Finally, we do the same for the directors, starting right after the previous range, at 10.0.5.0. By associating the mask to that addresss (10.0.5.0/27), we find the last address of the range: 10.0.5.31.

That's it, we have our 3 subnets that look like this:

- 10.0.0.0/22 -> 10.0.0.0 to 10.0.3.255 for the technicians
- 10.0.4.0/24 -> 10.0.4.0 to 10.0.4.255 for the sales people
- 10.0.5.0/27 -> 10.0.5.0 to 10.0.5.31 for the directors

**Important rule**: to avoid errors (not explained here), always start with the largest ranges! That's what we did (1000, 200, 20). This was a pretty easy example, it's sometimes more complicated in real life.

### The magical method

We just saw the base method to split address ranges, but fortunately, there is an easier and faster method in which we don't need to transform into binary.

#### The magical number

In order to use the magical method, we need the **magical number**: it is a simple calculation made from the significant byte (where the split occurs) of a mask, and is defined as **256 - significant byte**.\\
Example: the significant byte of the mask 255.224.0.0 is 224. Hence, the magical number is 256 - 224 = 32.

Great, what do we do now with this magical number? We can use it to **instantly determine the first and last addresses of our range**! To do so, we simply have to write all the multiples of the magical number, up to 256. So, for 32 we have: 0, 32, 64, 96, 128, 160, 192, 224, 256. Now, we apply the 2 following rules:

- The first address of the range will be the multiple of the magical number lower or equal to the corresponding byte in the address.
- The last addrress of the range will be the next multiple, minus 1

**Example:** we have the pair 192.168.0.1 with the mask 255.224.0.0. The significant byte in the mask is the second one, 224 (magical number = 256 - 224 = 32). Therefore, we take the second byte of our address, that is 168.\\
The first address of the network will be the multiple of the magical number, lower or equal to 168. We quickly find it is 160.\\
The last address will be the next multiple minus 1, that is 192 - 1 = 191.\\
And we have our addresses: the first one is 192.160.0.0 (we add the 0's), and the last one is 192.191.255.255 (we add the 255's).

**A concrete example of splitting an address range with the magical method**: let's assume we're in charge of the network in a small company. The general admin gives us the network 192.168.160.0/255.255.224.0. We want to split this newtwork into the 3 following subnets:

- 550 technicians
- 130 sales people
- 10 directors

We start by determining the original range. The magical number is 256 - 224 = 32. The significant byte of the mask is the third, so the corresponding byte of the address is 160 (a multiple of 32!). The first address is 192.168.160.0, the last is 192.168.191.255.

Then, we calculate the masks. We saw that the size of a range depends on the mask -> if we know the required number of addresses, we can determine the mask (Nb. of addresses = 2^Nb. of 0's in the mask).\\
For the 550 technicians, the network will need to have 2^10 = 1024 addresses. The corresponding mask is 11111111.11111111.11111100.00000000, or 255.255.252.0.\\
For the sales people, 130 < 2^8 so the mask is 255.255.255.0.\\
For the directors, 10 < 2^4, so the mask is 255.255.255.240.\\

Finally, now that we have the masks, we just have to associate addresses in order to have our ranges. Let's start with the technicians: we start at the beggining of the range, 192.168.160.0. We'll find the last address with the mask (255.255.252.0). The magical number is 256 - 252 = 4. The next multiple of 4 after 160 is 164 - 1 = 163. So, the last address for the technicians is 192.168.163.255.\\
For the sales people, we can start right after at 192.168.164.0. Since the split occurs between 2 bytes in the mask (255.255.255.0), we don't need the magical number: it's easy to see that the last address is 192.168.164.255.\\
For the directors, we start at 192.168.165.0. The magical number is 256 - 240 = 16. So, the last address is 192.168.165.15.\\

## Routing

This chapter explains how information goes from a network to another.

### The protocol, IP

Now that we know how to communicate on a network, we want to know how to communicate with other networks. To understand this, we must first see the layer 3 protocol, IP (Internet Protocol). As for layer 2, we will see what information are required by the protocol in the header, and in which order.\\
We'll start with the IP addresses of the sender and receiver. Let's imagine machine A (192.168.0.1/24) wants to send a message to machine B (192.168.1.1/24).

Is the mask required in the address ?\\
--> No, the IP address alone is enough: the only important thing to know is whether B is on the same network as A. If it is the case, A can talk to B with layer 2. If B is on another network, A will have to use layer 3.

How do we know if B is on our network?\\
--> we check if the address of B is part of our address range. In our case, our range is 192.168.0.0 - 192.168.0.255. We see that B isn't part of our network. Note that we didn't need B's mask.

We have 2 IP addresses so far, that we will call **source IP address** and **destination IP address**. In layer 2, the message was called an ethernet frame. In layer 3, we talk about a **datagram** or **packet**. 

#### The datagram (or packet)

As the ethernet frame of layer 2, the IP datagram of layer 3 consists of an organized sequence of 0's and 1's with the following structure:

<div class="img_container">
![Layer 3 packet structure]({{https://jsom1.github.io/}}/_images/tcpip_packet.png){: height="100px" width = "300px"}
</div>

Unlike the ethernet frame, the source IP address comes before the destination IP address. In the ethernet frame, we saw that the MAC destination address had to come first so that the receiving machine immediately knows whether the packets is for it or not...\\
But why is it different for the IP address ?!
--> To understand why the structure is the way it is, we musst first understand other concepts.

#### Encapsulation

Let's start by asking a question: what is circulating on the network? Frames ? Datagrames ? Both ?
--> To answer this question, we have to look at the OSI model once again:

<div class="img_container">
![OSI model]({{https://jsom1.github.io/}}/_images/tcpip_OSI2.png){: height="300px" width = "330px"}
</div>

As we see, a message is sent from the 7th layer of the OSI model, and it goes through all the layers above until layer 1 where it is sent on the network. When the message is received, it goes from layer 1 to layer 7.\\
What does our message and the headers of each layer become?\\
--> A header is added each time we go through a layer. Hence, we accumulate the headers of the different layers. When we go through layer 4, we add the layer 4 header, then layer 3, and so on. This mechanism is called **encapsulation**.

At this point, we can answer our previous question: what circulates on the network is a frame of layer 2, containing a datagram or packet of layer 3, containing the element of layer 4, ...
p. 115
