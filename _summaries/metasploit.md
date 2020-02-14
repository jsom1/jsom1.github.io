# Hacking and pentesting with Metasploit

This is a resumee of the book “Hacking, security and penetration testing with Metasploit”.
The chapters and their content are the following :
1. Pentesting basics : presents the methodology of pentests
2. Metasploit basics : introoduction to the tools of Metasploit
3. Information gathering : how to use Metasploit to gather intelligence in the early stages of a pentest
4. Vulnerability scanning : how to identify vulnerabilities and scanning techniques
5. Exploitation : introduction to exploitation
6. Meterpreter : presentation of the post exploitation swiss knife
7. Avoid detection : ideas behind AVs evasion
8. Client side attacks exploitation
9. Metasploit : auxiliary modules
10. Social engineering toolkit (SET)
11. FAST-TRACK : an automated pentest infrastructure
12. Karmetasploit : how to use it for wireless attacks
13. Create your own module : how to build your own module for exploitation
14. Create your own exploit : covers fuzzing and exploits for buffer overflows
15. Carry exploits on the Metasploit framework
16. Meterpreter scripts
17. Pentest simulation

## 1. Pentesting basics
### The 7 PTES (Penetration Testing Execution Standard) phases

1. Pre-Engagement Actions
Discussion of the test’s scope with the client (what can we do, what methods we can use, which networks and addresses are in range (so that we do not take down a critical system), …)
It is important to have a precise contract defining the scope and the engagements.

2. Information gathering (Reconnaissance, footprinting)
The idea of this phase is to gather as much information about the target as possible. We want to know how the target works and how it can be attacked, what mechanisms protect it, ... Commons reconnaissance methods include :
Google hacking
Domain name searches, WHOIS lookups, reverse DNS (to get subdomains, people’s name and data about the attack surface)
Social engineering (to find out positions, technologies, email addresses, …)
Social networks footprinting
Dumpster diving (various physical methods to get information)
Tailgating (closely follow someone who has legitimate access)

3. Threat Modeling
Think like an attacker about the company’s assets and how they could be used. It can be useful to make an organizational structure of the company : who works where, what is their role, can they be used to gain access to something (information gathered in the previous phase) ?
In general, we are interested in employee data, customer data and technical data. There is nothing concrete in this phase, we just elaborate a plan.

4. Vulnerability identification
Now that we identified the best attack methods, we have to determine how we will access the target. Typically, scanning tools (port scanners, vulnerability scanners) are used in this phase. Here we try to know what systems are up, what OS are they running, if there is a firewall, an antivirus (AV), intrusion detection system (IDS), etc…

5. Exploitation
An exploit should be made only if we are 100% sure that it will work. Before taking advantage of a vulnerability, we have to know that the system is vulnerable. The goal is to gain access to systems, and gain as high of administrator access as possible.

6. Post-exploitation
Starts after we have compromised one or more systems. In this phase, we target specific systems, identify critical infrastructures and look for data that the target tried to secure. To what point can we jeopardize a system, and what would the repercussions be ?
We really have to think like a pirate in this phase.

7. Report
The report is the most important part of a pentest. It contains what we did, how we did it, and most importantly, what could the organization do to correct the vulnerabilities we found and used. The purpose is that the client enhances the global security, and doesn’t just patch the vulnerabilities.
The report should contain an abstract, a presentation and a technical part.


### Penetration test types
* White box (white hat)
It is a visible test : we work with the client.
Pros : the client’s security team can show us the systems and everything we might want to know.
Cons : might be biased.
A white box test can be good when we have a limited time and can’t do a proper information gathering phase.

* Black box
It is an invisible test, performed without the knowledge of the organization.
Pros : much more realistic than a white box test.
Cons : lasts way longer than a white box test and require more skills.
If possible, this should be the prefered over a white box test.



