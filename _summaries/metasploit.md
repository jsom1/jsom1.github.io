---
title: "Hacking and pentesting with Metasploit"
author: "Me"
date: "May 25, 2015"
output: html_document
---

# Hacking and pentesting with Metasploit
{:.no_toc}

This is a resumee of the book “Hacking, security and penetration testing with Metasploit”. The book contains many explanations and demos for specific tools; these will not be resumed here. Instead, I will focus on concepts and theory.

The chapters are the following:

1. TOC
{:toc}
{:style="color:black; font-size: 150%;"}

# Pentesting basics

## The 7 PTES (Penetration Testing Execution Standard) phases
{:style="color:DarkRed; font-size: 170%;"}

1. **Pre-Engagement Actions**\\
Discussion of the test’s scope with the client (what can we do, what methods we can use, which networks and addresses are in range, …).  
It is important to have a precise contract defining the scope and the engagements.

2. **Information gathering (Reconnaissance, footprinting)**\\
The idea of this phase is to gather as much information about the target as possible. We want to know how the target works and how it can be attacked. Commons reconnaissance methods include : Google hacking, domain name searches, WHOIS lookups, reverse DNS, social engineering, social networks footprinting, dumpster diving (various physical methods to get information) and tailgating (closely follow someone who has legitimate access into a secure building).

3. **Threat Modeling**\\
Think like an attacker about the company’s assets and how they could be used. It can be useful to make an organizational structure of the company : who works where, what is their role, can they be used to gain access to something (information gathered in the previous phase) ?
In general, we are interested in employee data, customer data and technical data. There is nothing concrete in this phase, we just elaborate a plan.

4. **Vulnerability identification**\\
Now that we identified the best attack methods, we have to determine how we will access the target. Typically, scanning tools (port scanners, vulnerability scanners) are used in this phase. Here we try to know what systems are up, what OS are they running, if there is a firewall, an antivirus (AV), intrusion detection system (IDS), etc…

5. **Exploitation**\\
An exploit should be made only if we are 100% sure that it will work. Before taking advantage of a vulnerability, we have to know that the system is vulnerable. The goal is to gain access to systems, and gain as high of administrator access as possible.

6. **Post-exploitation**\\
Starts after we have compromised one or more systems. In this phase, we target specific systems, identify critical infrastructures and look for data that the target tried to secure. To what point can we jeopardize a system, and what would the repercussions be ?
We really have to think like a pirate in this phase.

7. **Report**\\
The report is the most important part of a pentest. It contains what we did, how we did it, and most importantly, what could the organization do to correct the vulnerabilities we found and used. The purpose is that the client enhances the global security, not that he just patch the vulnerabilities we found.
The report should contain an abstract, a presentation and a technical part.


## Penetration test types

1. **White box (white hat)**\\
It is a visible test : we work with the client.
Pros : the client’s security team can show us the systems and everything we might want to know.
Cons : might be biased.
A white box test can be good when we have a limited time and can’t do a proper information gathering phase.

2. **Black box**\\
It is an invisible test, performed without the knowledge of the organization.
Pros : much more realistic than a white box test.
Cons : lasts way longer than a white box test and require more skills.
If possible, this should be the prefered over a white box test.


# Metasploit basics : introduction to the tools of Metasploit

## Terminology

<ins>**Exploit**</ins>\\
An exploit is the mean by which an attacker take advantage of a vulnerability in a system, an application or a service. **buffer overflows** and **SQL injections** are examples of exploits.

<ins>**Payload**</ins>\\
A payload is a piece of code that we want to be executed by the tarhet system. For example, a **reverse shell** is a payload that creates a connection from the target to the attacker, whereas a **bind shell** is a payload that binds a listener on a port of the target machine so that the attacker can connect to it.

<ins>**Shellcode**</ins>\\
A shellcode is a set of instructions used by a payload during an exploit. It is typically written in **assembly**. In most cases, a **shell** or a **meterpreter shell** is used after a set of instructions has been executed by the machine.

<ins>**Module**</ins>\\
A module is a part of a software that can be used by the Metasploit framework. It is sometimes necessary to use an **exploit module** that carries the attack, or an **auxiliary module** to scan or enumerate systems.

<ins>**Listener**</ins>\\
A listener is a procedure or function that awaits any entering connection. For example, once that target has been exploited, it can communicate with the attacker via internet. The listener handles this connection.

## The interfaces of Metasploit

There are 3 interfaces: **Msfconsole**, a **CLI** (command line interface) and a **GUI** (graphical user interface, Armitage), and different utilities. We will be using msfconsole only. The utilities are *direct interfaces* that can be useful for exploit development and are the following:

<ins>**MSFpayload**</ins>\\
Msfpayload is used to generate a shellcode, executables and more. Shellcode can be written in many languages (C, Ruby, JavaScript, ...). Each format is useful in different situations; for example if we want to create an exploit for a browser, JavaScript would be a good choice. The payload can then simply be inserted into an HTML file. To see the options used by the utility, we can use the command *msfpayload -h*.

<ins>**MSFencode**</ins>\\
The shellcode generated by MSFpayload works, but it contains null characters which, when interpreted by some programs, mean the end of a string. In other words, the code would stop before it reaches the end because of some *x00s* and *xffs*.

In addition to that, it's risky to send a shellcode in plaintext on a network; it could be easily detected by IDS (intrusion detection system) and AV (antivirus). 

Msfencode is used to encode the payload in a way that avoids those undesired characters. We can see the list of options with the command *msfencode -h*. There are different encoders, useful for different situations. One of the best encoder is **x86/shikata_ga_nai** (*msfencode -l* shows a list of available encoders).


<ins>**Nasm Shell**</ins>\\
Useful to analyze assembly code, especially if we want to identify **opcodes** of an assembly instruction. For example, we can start the tool with *./nasm_shell.rb* and then ask for the code of *jmp esp*; it would return 00000000 FFE4.

# Information gathering : how to use Metasploit to gather intelligence in the early stages of a pentest

Information gathering is the second phase of the PTES and maybe the most important. It is the basis of all the work that will come after it.

## Passive information gathering

We collect information without touching the target systems. We can identidy the architecture, the network admins, what OS is used and what web server is used on the target network. **OSINT** (Open Source Intelligence) is a form of information gathering during which we use free free or easily obtainable information to chose and acquire information on the target.

A few examples are **whois** lookups (for example, *whois google.com* in a terminal gives a lot of information), **Netcraft** (a web tool to find the IP of a server hosting a website) and  **NSLookup** (gives more information on the server.)\\
Passive information gathering is useful to define what systems should be included in the pentest (in a legal point of view).

## Active information gathering

In active information gathering, we interact directly with a system to learn more about it. **Port scanning** is an example of active information gathering. **Nmap** is one of the most popular port scanner.

In a complex pentest, where there are multiple targets to scan, it is very important to keep a trace of what we do. Fortunately, Metasploit has its own databases (MySQL and PostgreSQL), PostgreSQL being used by default.

**Pivoting** is the process by which we can access and attack systems that would be out of reach by using interconnected systems to route traffic.\\
Example: let's suppose we compromised a system behind a Firewall using NAT (Network Address Translation). This system uses private IP addresses that we can't access from the internet. We can use this compromised system to avoid the firewall and access the internal network of the target.


## Targeted scans
It is sometimes useful to perform **targeted scans** to find specific services and/or versions that we know are vulnerable. Some examples are:

- Scan a target network to find the vulnerability **ms08-067**, because it is a very common vulnerability that gives access to SYSTEM.

- Detect the versions of Microsoft Windows with the module **smb_version** (*use scanner/smb/smb_version*). In the options, we can chose the number of threads; 1 (default) is enough if we scan a single system, but we could use more if we scan a subnet for example.

- Look for misconfigured Microsoft SQL servers (**MS SQL**), known to be an easy entry door in a network. This is due to the fact that many system admins don't even know that their machines use MS SQL, because it is preinstalled on the system. It is often misconfigured or outdated.\\
When installed, MS SQL listens on the TCP port 1433 by default (see the module *mssql_ping*). 

- Scan of **SSH** (Secure Shell) servers. If we find any, we have to determine its version. Many vulnerabilities have been identified in its implementations, and we could be lucky and find an old machine that hasn't been updated. To determine the version, we can use the modulee *ssh_version**. 

- Scan of **FTP** (File Transfer Protocol). FTP is a complicated and vulnerable protocol. FTP servers are often the easiest way in a target network. We can use the module *ftp_version*. If there is a FTP server, we can see if it allows anonymous connection with scanner *anonymous* (*use auxiliary/scanner/ftp/anonymous*). 

- **SNMP** (Simple Network Management Protocol) scan. Usually, this protocol is used by network devices. However, some OS have SNMP servers that provide information about the CPU usage, available memory and other details of the system. If there is such a server, it's a gold mine for a pentester because it gives a ton of information. We can use the auxiliary module *snmp_enum* for SNMP enumerations.

# Vulnerability scanning : how to identify vulnerabilities and scanning techniques

Finding vulnerabilities automatically can be done with the help of a **vulnerability scanner**. It basically sends data on a network and analyze the answers to enumerate vulnerabilities, using a vulnerability database. OS answer differently based on their network layers implementations, and those unique answers are used by the scanner to determine the OS version and its patch version. It can also connect to the target system and enumetate softwares and services to see if they are patched.

Vulnerability scanners are very noisy on a network. They should not be used if we want to be unnoticed, but can save valuable time if it doesn't matter.

## Base vulnerability scan

We can use **netcat** to perform a **banner grabbing** on the target. The command would be *nc 192.168.1.293 80*: here we connect to 192.168.1.293 on a web server (port 80), and we can then use the command *GET HTTP* to receive headers information returned by the server. Imagine we receive the information *Server: Microsoft-IIS/5.1*: we could then use a vulnerability scanner to determine if this ISS version has known vulnerabilities and if the server is up to date.\\
It can be harder in reality, because there are false positives and false negatives.

**NeXpose** and **Nessus** are examples of vulnerability scanners (see the book for more details on their use). NeXpose can either be used from a web interface or directly in Metasploit with the plugin NeXpose. Nessus can also be used in Metasploit with the plugin Nessus, or in a web interface.


# Exploitation : introduction to exploitation

# Meterpreter : presentation of the post exploitation swiss knife

# Avoid detection : ideas behind AVs evasion

# Client side attacks exploitation

# Metasploit : auxiliary modules

# Social engineering toolkit (SET)

# FAST-TRACK : an automated pentest infrastructure

# Karmetasploit : how to use it for wireless attacks

# Create your own module : how to build your own module for exploitation

# Create your own exploit : covers fuzzing and exploits for buffer overflows

# Carry exploits on the Metasploit framework

# Meterpreter scripts

# Pentest simulation


