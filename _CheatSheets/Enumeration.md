---
title: "Enumeration CheatSheet"
author: "Me"
date: "February 11, 2022"
output: html_document
---

# Enumeration CheatSheet
{:.no_toc}

This "CheatSheet" aims to provide a few enumeration commands for each stage of a penetration test (initial enumeration and Windows/Linux enumeration once a foothold has been obtained). 

Here's the content so far:

1. TOC
{:toc}
{:style="color:black; font-size: 150%;"}

# Domain information
Service/protocol: Hypertext Transfer Protocol
Port(s): 80, 443
Description: the purpose is to get a picture of the target's online presence, including subdomains.\\
- Examine the main's website SSL certificate (could reveal subdomains)
- Check for information on crt.sh (alternatively *curl* it: curl -s https://crt.sh/\?q\=<domainname.com>\&output\=json | jq .)
- Filter hosts directly reachable from the internet (not hosted by third-party providers, as we are not allowed to test these without their permission): for i in $(cat subdomainlist);do host $i | grep "has address" | grep <domainname.com> | cut -d" " -f1,4;done
- Repeat the previous command but keep only IP addresses: for i in $(cat subdomainlist);do host $i | grep "has address" | grep <domainname.com> | cut -d" " -f4 >> ip-addresses.txt;done.\\
Then, run them through *Shodan*: for i in $(cat ip-addresses.txt);do shodan host $i;done. Shodan can find devices and systems such as cameras, servers, smart home systems, and so on...
- Finally, inspect the DNS records: dig any <domainname.com>. Some of the records are: A = this record contains a domain's IP address. MX = shows which mail server manages the email in the company. NS = show which name servers are used to resolve the fully qualified domain name (FQDN) to IP addresses. TXT = often contains verification keys for third-party providers, ... This can reveal interesting third-parties such as Atlassian, Google Gmail, mailgun, outlook, and so on.

# Cloud Resources
Cloud providers secure their infrastructure centrally, but it doesn't mean that companies usign them don't have vulnerabilities. Indeed, that depends on  the configurations made by the administrators. Sometimes, it is possible to retrieve documents from the cloud. 
- For example, Google search for AWS: intext:<company name> inurl:amazonaws.com. For Azure, it would be: intext:<company name> inurl:blob.core.windows.net. Such researches could return PDF files, text files, presentations, code, and others. 
- Alternatively, we can check on https://buckets.grayhatwarfare.com/. If we're lucky, we could find for example a SSH private key. After downloading it, we could log onto one or more machines in the company, without using a password.
- Another good resource: https://domain.glass/ (it also shows the DNS records)

# HTTP enumeration
Service/protocol: Hypertext Transfer Protocol
Port(s): 80, 443
Description:

# FTP enumeration
Service/protocol: File Transfer Protocol
Port(s): 21
Description: the File Transfer Protocol is a client-server protocol and one of the oldest on the internet. It is meant for transmitting files between computers over TCP/IP connections and relies on 2 communication channels between the client and sever: a control channel for controlling the conversation (TCP port 21) and a data channel for transmitting file content (usually TCP port 20).\\
FTP can operate in two modes: active or passive. In the active mode, the client establishes the connection and informs the server via which port the server can transmit its responses. The problem is that if the client is protected by a firewall, the server cannot reply since external connections are blocked. In the passive mode, the server announces a port through which the client can establish the data channel. Since it is the client who initiates the connection. the firewall doesn't block the transfer.\\
FTP is still commonly used to transfer files behind the scenes by applications. Although credentials are usually required to use FTP on a server, the server sometimes offers anonymous FTP. If this is the case, we can simply connect to it, list directories, and possibly even download and upload files.\\
VsFTPd is one of the most used FTP servers on Linux. Its configuration can be found in */etc/vsftpd.conf*. Also, a file called */etc/ftpusers* is used to deny certain users access to the service.\\
A few useful commands: ls -R (recursive listing), wget -m --no-passive ftp://anonymous_anonymous@<targetIP> (downloads all available files). So, if FTP is running:
- Use nmap NSE script listed with the following command: find / -type f -name ftp* 2>/dev/null | grep scripts
- Use nmap: sudo nmap -sV -p21 -sC -A <targetIP>
- Try to log in as anonymous (could be revealed by one of the previous scan)
- Interact with the service with netcat or telnet: nc -nv <targetIP> 21 or telnet <targetIP> 21
- If the FTP server runs with TLS/SSL encryption, we need a vlient that can handle it. We can use openssl: openssl s_client -connect <targetIP>:21 -starttls ftp

# DNS enumeration
Service/protocol: Domain Name System
Port(s): 53
Description:

# SMB enumeration
Service/protocol: Server Message Block
Port(s): 139, 445
Description: SMB is a client-server protocol which regulates access to files and entire directories and other network resources such as printers and routers. SMB can also handle information exchange between different system processes. For example, devices running old versions of windows can communicate with devices running newer editions. With Samba, there is also a solution that enables the use of SMB in Linux and Unix distributions (thus making cross-platform communication possible with SMB).\\
An SMB server can provide arbitrary parts of its local file system as shares. Therefore, what we see is partially independent of the real structure of the server. Access rights are defined by Access Control Lists (ACL), which are defined based on the shares. SMB usually uses TCP ports 137, 138 and 138, but we can also see port 445 on Linux hosts since Samba implements the Common Internet File System (CIFS) network protocol, which only uses port 445.\\
Here also, we're looking for misconfigurations of SMB (if Samba is used, the config file can be found in */etc/samba/smb.conf*). Let's assume we found SMB with nmap. We can enumerate the service as following:
- Display the server's shares: *smbclient -N -L //<targetIP>* (-L to display a list, -N specifies a null session, which is anonymous access). Then, connect to a share: *smbclient //<targetIP>/<sharename>*. We can use the command *help* to get a list of possible commands.
- Get the server status: *smbstatus* (shows the version, and also who, from which host, and which share the client is connected)
- Use nmap: *sudo nmap <targetIP> -sV -sC -p139,445*
- Use rpcclient "manually": *rpcclient -U "" <targetIP>*. Then, some commands are: srvinfo, enumdomains, querydominfo, netshareenumall, netsharegetinfo <sharename>, enumdomusers, queryuser <RID>.
- Brute force User RIDs: *for i in $(seq 500 1100);do rpcclient -N -U "" <targetIP> -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name\|user_rid\|group_rid" && echo "";done* (alternatively use the Python script from *impacket* called *samrdump.py*.
- Alternatively to rpcclient, use SMBmap: *smbmap -H <targetIP>*
- Alternatively to rpcclient or SMBmap, use CrackMapExec: *crackmapexec smb <targetIP> --shares -U '' -p ''*
- Alternatively to the previous commands, use Enum4Linux-ng: *git clone https://github.com/cddmp/enum4linux-ng.git*, then *cd enum4linux-ng* and *pip3 install -r requirements.txt*. Finally, run *./enum4linux-ng.py 10.129.14.128 -A*.

# NFS enumeration
Service/protocol: Network File System
Port(s): 111, 2049
Description: NFS is a network file system which has the same purpose as SMB: access file systems over a network as if they were local. However, NFS uses a different protocol and is only used between Linux and Unix systems. Therefore, NFS clients cannot communicate directly with SMB servers. NFS version 3.0 authenticates the client computer, but more recent versions (NFSv4) authenticates the user (as with SMB). Because NFS has less options than FTP or SMB, it is easier to configure and might have less misconfigurations. Anyways, the */etc/exports* file contains a table of physical filesystem on an NFS server that are accessible by the clients. It also shows which options it accepts.

# SMTP enumeration
Service/protocol: Simple Mail Transfer Protocol
Port(s): 25
Description:

# SNMP enumeration
Service/protocol: Simple Network Management Protocol
Port(s): 161, 162 (UDP)
Description:


