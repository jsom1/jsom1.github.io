---
title: "Hacking and pentesting with Metasploit"
author: "Me"
date: "May 25, 2015"
output: html_document
---

<style type="text/css">

body{ /* Normal  */
      font-size: 12px;
  }
td {  /* Table  */
  font-size: 12px;
}
h1.title {
  font-size: 38px;
  color: DarkRed;
}
h1 { /* Header 1 */
  font-size: 28px;
  color: DarkRed;
}
h2 { /* Header 2 */
    font-size: 22px;
  color: DarkRed;
}
h3 { /* Header 3 */
  font-size: 18px;
  font-family: "Times New Roman", Times, serif;
  color: DarkRed;
}
code.r{ /* Code block */
    font-size: 12px;
}
pre { /* Code block - determines code spacing between lines */
    font-size: 14px;
}
</style>


# Hacking and pentesting with Metasploit
{:.no_toc}

This is a resumee of the book “Hacking, security and penetration testing with Metasploit”.
The chapters are the following :

1. TOC
{:toc}
{:style="color:black; font-size: 150%;"}

# Pentesting basics

## The 7 PTES (Penetration Testing Execution Standard) phases

1. **Pre-Engagement Actions**\\
Discussion of the test’s scope with the client (what can we do, what methods we can use, which networks and addresses are in range, …).  
It is important to have a precise contract defining the scope and the engagements.
{:style="color:black; font-size: 150%;"}

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


# Metasploit basics : introoduction to the tools of Metasploit

# Information gathering : how to use Metasploit to gather intelligence in the early stages of a pentest

# Vulnerability scanning : how to identify vulnerabilities and scanning techniques

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


