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

# The OSI model

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

# Layer 1: connecting machines

A medium is required to communicate. For example, we need an **ethernet cable** or air (for Wi-Fi) to connect to the internet. So, layer 1 transmits electrical signals (0's and 1's). Nowadays, the optical fibre is common and transmits light instead of electrical signals. Those 0's and 1's are carried by the different transmission media (described in the book).
To interconnect several machines on a network, we need a connection device: the hub. The hub has a weird way of working. Imagine 5 machines (A B C D E) connected to a hub: if A wants to talk to C, it will send data to the hub. However, this latter can't read and will forward it to all the machines, supposing that one of them is the right one. The machines B, D and E will detect the data isn't for them and will discard it, whereas C will read it. So, the hub isn't the best tool when it comes to data privacy.

There are different network topologies: bus, ring, mesh, star and hybrid. The pros and cons of each are not discussed here, and let's focus on the star topology since bus and ring are rare nowadays. In this setup, all the communications go through a central point. A machine sends information with the recipient's name to the central point, which redirects it to the right machine.\\
The number of machines on this topology depends on the capacity of the central point to manage information. Nowadays, switches can manage thousands of machines. Also, we can link many central points together to create an almost infinite network. 

# Layer 2: allow communications between machines

The role of the second layer is to connect machines on a local network, and allow them to communicate. It also detects transmission errors (it detects but doesn't correct them!).

## An identifier, the MAC address
{:style="color:DarkRed; font-size: 170%;"}

In order to communicate with a specific machine on a network, we must be able to identify it precisely. The **MAC** address is a unique identifier that corresponds to the address of the network card. Each machine on earth has it own unique MAC address. It is written in hexadecimal, for example 00:23:5e:bf:45:6a.
We see it is coded on 6 bytes, where 1 byte represents 8 bits. So, 1 byte = 8 bits = 2^8 = 256 possible values -> a byte can take any value between 0 and 255.
Because the MAC address is coded on 6 bytes (6x8 = 48 bits), it can take 2^48 = 281,474,976,710,656 values. This gives a lot of potential MAC addresses.\\

A network card manufacturer buys a part of MAC addresses to make sure it is unique. The 3 first bytes represent the manufacturer.

There is one special MAC addresss, where all the bits are set to 1 (ff:ff:ff:ff:ff:ff): it is a univeral address called the **broadcast** address. It identifies any network card. It can be used to send a message to all the network cards of machines on a network. Every machine receiving a message containing the broadcast address will suppose it can read the message.

Now that we can identify machines, we need a language.

## A protocol, Ethernet
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
![Layer 2 packet structure]({{https://jsom1.github.io/}}/_images/endian.png){: height="250px" width = "180px"}
</div>

The size of the bold elements never changes and constitute the **header** (here, the ethernet header). Its size is always 18 bytes. Only tee size of the data part varies. What's the minimum and maximum size of a frame ?\\
The minimum size for an ethernet frame is **64 bytes**, the maximum is **1518 bytes**.

# Layer 2 hardware: the switch




