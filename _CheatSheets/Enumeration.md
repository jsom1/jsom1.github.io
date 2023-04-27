---
title: "Enumeration CheatSheet"
author: "Me"
date: "February 11, 2022"
output: html_document
---

# Enumeration/footprinting CheatSheet
{:.no_toc}

This "CheatSheet" aims to provide a few enumeration/footprinting commands for each stage of a penetration test (initial enumeration and Windows/Linux enumeration once a foothold has been obtained). 

Here's the content so far:

1. TOC
{:toc}
{:style="color:black; font-size: 150%;"}

# Domain information
**Service/protocol**: Hypertext Transfer Protocol\\
**Port(s)**: 80, 443\\
**Description**: the purpose is to get a picture of the target's online presence, including subdomains.
- Examine the main's website SSL certificate (could reveal subdomains)
- Check for information on crt.sh (alternatively *curl* it: *curl -s https://crt.sh/\?q\=<domainname.com>\&output\=json | jq .*)
- Filter hosts directly reachable from the internet (not hosted by third-party providers, as we are not allowed to test these without their permission): *for i in \$(cat subdomainlist);do host \$i \| grep "has address" \| grep \<domainname.com\> \| cut -d" " -f1,4;done*
- Repeat the previous command but keep only IP addresses: *for i in \$(cat subdomainlist);do host \$i \| grep "has address" \| grep <domainname.com> \| cut -d" " -f4 >> ip-addresses.txt;done*.
Then, run them through *Shodan*: *for i in \$(cat ip-addresses.txt);do shodan host $i;done*. Shodan can find devices and systems such as cameras, servers, smart home systems, and so on...
- Finally, inspect the DNS records: *dig any \<domainname.com\>*. Some of the records are: A = this record contains a domain's IP address. MX = shows which mail server manages the email in the company. NS = show which name servers are used to resolve the fully qualified domain name (FQDN) to IP addresses. TXT = often contains verification keys for third-party providers, ... This can reveal interesting third-parties such as Atlassian, Google Gmail, mailgun, outlook, and so on.


# Cloud Resources
Cloud providers secure their infrastructure centrally, but it doesn't mean that companies usign them don't have vulnerabilities. Indeed, that depends on  the configurations made by the administrators. Sometimes, it is possible to retrieve documents from the cloud. 
- For example, Google search for AWS: intext:<company name> inurl:amazonaws.com. For Azure, it would be: intext:<company name> inurl:blob.core.windows.net. Such researches could return PDF files, text files, presentations, code, and others. 
- Alternatively, we can check on https://buckets.grayhatwarfare.com/. If we're lucky, we could find for example a SSH private key. After downloading it, we could log onto one or more machines in the company, without using a password.
- Another good resource: https://domain.glass/ (it also shows the DNS records)

  
# FTP enumeration
**Service/protocol**: File Transfer Protocol\\
**Port(s)**: 21\\
**Description**: the File Transfer Protocol is a client-server protocol and one of the oldest on the internet. It is meant for transmitting files between computers over TCP/IP connections and relies on 2 communication channels between the client and sever: a control channel for controlling the conversation (TCP port 21) and a data channel for transmitting file content (usually TCP port 20).\\
FTP can operate in two modes: active or passive. In the active mode, the client establishes the connection and informs the server via which port the server can transmit its responses. The problem is that if the client is protected by a firewall, the server cannot reply since external connections are blocked. In the passive mode, the server announces a port through which the client can establish the data channel. Since it is the client who initiates the connection. the firewall doesn't block the transfer.\\
FTP is still commonly used to transfer files behind the scenes by applications. Although credentials are usually required to use FTP on a server, the server sometimes offers anonymous FTP. If this is the case, we can simply connect to it, list directories, and possibly even download and upload files.\\
VsFTPd is one of the most used FTP servers on Linux. Its configuration can be found in */etc/vsftpd.conf*. Also, a file called */etc/ftpusers* is used to deny certain users access to the service.\\
A few useful commands: ls -R (recursive listing), wget -m --no-passive ftp://anonymous_anonymous@<targetIP> (downloads all available files). So, if FTP is running:
- Use nmap NSE scripts listed with the following command: *find / -type f -name ftp\* 2>/dev/null \| grep scripts*
- Use nmap: *sudo nmap -sV -p21 -sC -A <targetIP>*
- Try to log in as anonymous (could be revealed by one of the previous scan)
- Interact with the service with netcat or telnet: *nc -nv <targetIP> 21* or *telnet <targetIP> 21*
- If the FTP server runs with TLS/SSL encryption, we need a client that can handle it. We can use openssl: *openssl s_client -connect <targetIP>:21 -starttls ftp*

  
# DNS enumeration
**Service/protocol**: Domain Name System\\
**Port(s)**: 53\\
**Description**: DNS is a system which is an integrel part of the internet; it resolves computer names into IP addresses. There are different types of DNS servers: DNS root servers, authoritative name servers, non-authoritative name servers, caching servers, forwarding servers, and resolvers. Although DNS is originally unencrypted, there are now ways to encrypt it (DNS over TLS (DoT), DNS over HTTPS (DoH), and the network protocol *NSCrypt*).\\
In addition to resolving names into IP addresses, the DNS also stores information about the services associated with a domain. Therefore, we can determine the role of a computer in a domain (for example a mail server) or what the domain's name servers are called.\\
The structure is as follows:
- Top Level Domains (TLD): .com, .net, .org, ...
- Second Level Domain: mydomain.com, mydomain.net, ...
- Sub-domain: www.mydomain.com, mail.mydomain.com, dev.mydomain.com, ...
- Hosts: WS01.dev.mydomain.com, ...\\
In the *Domain information* section, we saw different DNS records that are used for the DNS queries. For example, *A* returns an IPv4 address of the requested domain as a result, *MX* returns the mail servers, *SOA* returns information about the corresponding DNS zone and email address of the administrative contact, and so on. We can use the *dig* command to get those information, for example: *dig soa <sub-domain>*.\\
DNS servers work with 3 types of configuration files:\\
- Local DNS configuration files
- Zone files
- Reverse name resolution files
On Linux, the DNS server *Bind9* is often used and its configuration file is *named.conf*.
  
  
**Footprinting**: if a DNS server is found to be running, it can be footprinted as follows:
- Query the server to discover other name servers using the *ns* record: *dig ns <secondLevelDomain> @<targetDNSserverIP>*
- Query the server to get its version using the *TXT* record and a class *CHAOS*: *dig CH TXT version.bind <targetDNSserverIP>*
- Query the server to see all available records (that it is willing to disclose): *dig any <secondLevelDomain> @<targetDNSserverIP>*
- Query the server to get the zone file: *dig axfr <secondLevelDomain> @<targetDNSserverIP>* (axfr = Asynchronous Full Transfer Zone, which consists in the transfer of the zone file from a master DNS server to other slaves DNS servers. This is done to ensure that all the DNS servers have the same version of the zone file after some changes have been made - if this is not the case, there could be a DNS failure).
- Query the server to get the another zone: *dig axfr internal.<secondLevelDomain> @<targetDNSserverIP>*
- Brute force subdomains with a bash one-liner: *for sub in $(cat /opt/useful/SecLists/Discovery/DNS/subdomains-top1million-110000.txt);do dig $sub.<secondLevelDomain> @<targetDNSserverIP> | grep -v ';\|SOA' | sed -r '/^\s*$/d' | grep $sub | tee -a subdomains.txt;done*
- Brute force subdomains using *DNSenum*: *dnsenum --dnsserver <targetDNSserverIP> --enum -p 0 -s 0 -o subdomains.txt -f /opt/useful/SecLists/Discovery/DNS/subdomains-top1million-110000.txt <secondLevelDomain>*


# SMB enumeration
**Service/protocol**: Server Message Block\\
**Port(s)**: 139, 445\\
**Description**: SMB is a client-server protocol which regulates access to files and entire directories and other network resources such as printers and routers. SMB can also handle information exchange between different system processes. For example, devices running old versions of windows can communicate with devices running newer editions. With Samba, there is also a solution that enables the use of SMB in Linux and Unix distributions (thus making cross-platform communication possible with SMB).\\
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
**Service/protocol**: Network File System\\
**Port(s)**: 111, 2049\\
**Description**: NFS is a network file system which has the same purpose as SMB: access file systems over a network as if they were local. However, NFS uses a different protocol and is only used between Linux and Unix systems. Therefore, NFS clients cannot communicate directly with SMB servers. NFS version 3.0 authenticates the client computer, but more recent versions (NFSv4) authenticates the user (as with SMB). Because NFS has less options than FTP or SMB, it is easier to configure and might have less misconfigurations. Anyways, the */etc/exports* file contains a table of physical filesystem on an NFS server that are accessible by the clients. It also shows which options it accepts, some of which can be dangerous. For example, there is an *insecure* option which allows to use ports above 1024. If this is the case, any users can use those ports and interact with the service (the first 1024 can only be used by root). There can also be misconfigurations of file permissions, such as *rw* (read and write) permissions.\\
If NFS is found to be running on the host, it can be footprinted with the following commands:\\
- Use nmap: *sudo nmap <targetIP> -p111,2049 -sV -sC*
- Use nmap's NSE scripts (such as rpcinfo): *sudo nmap --script nfs* <targetIP> -sV -p111,2049*
- Show available NFS shares: *showmount -e <targetIP>*
- Mount a share on our local machine: we start by creating a new empty directory folder to which the NFS share will be mounted. Once mounted, we can access it as anything else on our local system. Create a folder: *mkdir <sharename>*, mount it with *sudo mount -t nfs <targetIP>:/ ./<sharename>/ -o nolock*. Finally, we can access it: *cd <sharename>*, *tree .*. At this point, we can see the rights and the usernames and groups to whom the files belong. With this information, we can create those usernames, groups, UIDs and GUIDs on our system and modify the files (see below).
- List contents with usernames and group names (once mounted): *ls -l mnt/nfs/*
- List content with UIDs and GUIDs: *ls -n mnt/nfs/*
- Finally, we unmount the share: *cd ..*, then *sudo umount ./<sharename>*.

Example: <a href="/_walkthroughs/Squashed">Squashed</a>

  
# SMTP enumeration
**Service/protocol**: Simple Mail Transfer Protocol\\
**Port(s)**: 25\\
**Description**: this protocol is for sending emails in a network. It can either be used between an email client and an outgoing mail server, or between 2 SMTP servers. It is often used combined with IMAP or POP3, two protocols used to access and send emails.\\
By default, SMTP servers accept connections on port 25 (newer servers can also use port 587) and it is unencrypted. As for DNS however, SMTP can be used in conjunction with SSL/TLS encryption to prevent unauthorized reading of data.\\
To prevent spam, most modern SMTP servers support the protocol extension ESMTP (Extended SMTP) with SMTP-Auth. This latter is used to authenticate users, so that only those latter can send emails. When a client (known as the Mail User Agent (MUA)) sends an email, it then converts it into a header and a body, and sends those to the SMTP server. There, the Mail Transfer Agent (MTA) checks the email for size and spam and stores it. The MTA is often preceded by a Mail Submission Agent (MSA, also called Relay Server), which checks the origin of the email. Finally, the MTA searches the DNS for the IP address of the recipient mail server. When the data packets arrive there, they are reassembled to form a complete email and the mail delivery agent (MDA) transfers it to the recipient's mailbox. Those steps can be summarized as follows:\\
Client (MUA) -> Submission Agent (MSA) -> Open Relay (MTA) -> Mail delivery agent (MDA) -> Mailbox (POP3/IMAP)\\
The SMTP configuration file can be inspected with the following command: *cat /etc/postfix/main.cf | grep -v "#" | sed -r "/^\s*$/d"*.\\
We can interact with an SMTP server by initiating a connection using telnet (the actual initialization of the session is done with the command *EHLO* or *HELO*): *telnet <tagetIP> 25*, then use a command like *EHLO <domainname>*. Depending on the configuration, the command *VRFY* can be used to check the existence of users on the system. After using telnet to connect (no need to use *EHLO*), we can for example *VRFY root*. Once again however, we could get false positives (response code 252) for any users we try depending on the configuration.\\
We could use SMTP to send an email as follows:\\
- Use telnet to connect: *telnet <FQDN/tagetIP> 25*
- Initialize the session: *EHLO <hostname>*
- Specify the sender: *MAIL FROM: <senderemail>*
- Specify the recipient: *RCPT TO: <recipientemail>*
- Type *data*
- Type the subject of the email: *subject:<subject>*
- Type the body of the mail. When done, type ".". There should be a confirmation message, saying the mail is queued. Finally, close the connection with *QUIT*.\\
If an SMTP server is found on the host, here's how it can be footprinted:\\
- Use nmap: *sudo nmap <targetIP> -sV -sC -p25*
- Use nmap "smtp-open-relay" NSE script: *sudo nmap <targetIP> -p25 --script smtp-open-relay -v
- Use smtp-user-enum tool (*sudo apt install smtp-user-enum* if necessary. Then the commands are shown with *smth-user-enum -h*): *smtp-user-enum -w 25 -M VRFY -U <wordlist.txt> -t <targetIP>*. In this command, -w is used to specify the waiting time for a reply, -M specifies the method to use for username guession (VRFY is the default, but it could also be EXPN or RCPT)
  

# IMAP/POP3
**Service/protocol**: Internet Message Access Protocol (IMAP) and Post Office Protocol (POP3)\\
**Port(s)**: 143 for IMAP2 and IMAP4, 220 for IMAP3, 993 IMAPS, 110 for unencrypted POP3, 995 for POP3S (secured POP3 over SSL/TLS)\\
**Description**: IMAP allows to access emails directly on a mail server, manage the emails on the server, and supports folder structures. In short, this protocol allows to manage emails on a remote server.\\
Its functioning is thus the opposite of POP3, which retrieves emails on the local machine and stocks them with a special software (by default, this software deletes the retrieved emails from the server). Note that newer versions of IMAP also give the possibility to retrieve emails locally.\\
As for any other service, IMAP and POP3 can be misconfigured with dangerous settings. Even though most companies use third-party email providers (Google, Microsoft, ...), some companies still use their own mail servers. If misconfigured, an attacker could potentially read all the emails... Here's how to footprint those services.\\
**Footprinting**:
- Use nmap: *sudo nmap \<targetIP\> -sV -sC -p110,143,993,995*
- Log into the mail server (given we have credentials): *curl -k 'imaps://\<targetIP\>' --user user:\<password\> -v* (the verbosity displays information about the version of TLS, the SSL certificate, and the banner which can contain the version of the mail server)
- Use openssl or ncat to interact with a IMAP or POP3 server over SSL: *openssl s\_client -connect \<targetIP\>:pop3s*, and *openssl s\_client -connect \<targetIP\>:imaps*
- Once connected and logged in to the target email server, we can interact with it with various commands (search for "imap commands" and "pop3 commands" on Google). For example once connected to imap, we can authenticate as follows: *1 LOGIN \<username\> \<password\>*. Then, we can list all directories: *1 LIST "" \** (this shows the inboxes names). We can then select a mailbox to access its messages: *1 SELECT \<inbox\>*. This shows the emails and their status. If there's one email with ID 1, we can get more information with *1 FETCH \<ID\> all*. To get the actual content of the email, use *1 FETCH \<ID\> RFC822*.
  
  
# SNMP enumeration
**Service/protocol**: Simple Network Management Protocol\\
**Port(s)**: 161, 162 (UDP)\\
**Description**: SNMP was created to monitor devices on networks. It can also be used to change settings remotely, such as those of routers, switches, servers, IoT devices, and more. In addition to the exchange of information, SNMP also uses UDP port 161 to transmit control commands. In normal communication, it is the client who requests information from the server. However, SNMP also allow the use of *traps* over UDP port 162. Traps consist of data packets sent from the server to the client, without the client requesting them. If a device is configured accordingly, an SNMP trap is sent to the client once a specific event occurs on the server-side.\\
The Management Information Base (MIB) was created to ensure that SNMP access works across manufacturers and different client-server combinations. The MIB consists in a text file that contains all queryable SNMP objects of a device. MIBs don't contain data, but they explain where to find which information.\\
The most recent version of SNMP is SNMPv3. The first version, SNMPv1, is still used in small networks and has no built-in authentication mechanism. Therefore, anyone with access to the network can read and modify network data. Also, SNMPv1 doesn't support encryption and all data is sent in plain text.\\
SNMPv2 existed in different versions, but today we mainly see SNMPv2c (c because it is based on community strings (like SNMPv1 and SNMPv3), which can be seen as passwords used to determine whether the requested information can be viewed or not. So, having/knowing the correct community string determines whether we can access information of devices in the network). This version isn't really better than the first one in terms of security, because it isn't encrypted either and community strings can be intercepted and read by anyone. Community strings can also be brute forced. The default community string of a device depends on the vendor who created it, but "public" is a very common one. There are 3 types of community strings: read-only (allows to request read-only information from a device), read-write (allows to read and change the configuration of a device), and SNMP trap (included when a device sends a trap).\\
Finally, SNMPv3 is much more secure, since it adds authentication and is encrypted. However, it is more complex and has more configuration options, so it might be more prone to misconfiguration. Because the transition from SNMPv2c to SNMPv3 is complex, SNMPv2c is still widely used in organizations.\\
We can look at the default configuration options of the SNMP daemon as follows: *cat /etc/snmp/snmpd.conf*. Some dangerous settings that can be overlooked by administrators are: *rwuser noauth*, *rwcommunity \<communityString\> \<IPaddress\>.
  
**Footprinting**: this can be done with specialized tools such as *snmpwalk*, *onesixtyone*, and *braa*.
- Use *onesixtyone* to identify (brute force) community strings (*sudo apt install onesityone*): *onesixtyone -c /opt/useful/SecLists/Discovery/SNMP/snmp.txt \<targetIP\>* (community strings bound to specific IP addresses are often named with the hostname of the host (administrators can name them as they want). To make them even harder to identify, symbols are sometimes added to these names. In these cases, we can create custom wordlists with *crunch*). Note that "public" is a very common one since it is the default.
- Use *snmpwalk* to query the OIDs (an OID is a node in a hierarchical namespace. A sequence of numbers uniquely identifies each node, so the position of the node in the tree can be determined) with their information (requires to know a community string (here "public") and a version which doesn't require authentication (here "v2c")): *snmpwalk -v2c -c public \<targetIP\>* 
- Use *Braa* with a known community string to brute force the OIDS and get information about them (*sudo apt install braa*): *braa \<communityString\>\@\<targetIP\>:.1.3.6.\** (for example)


# MySQL
**Service/protocol**: MySQL\\
**Port(s)**: 3306\\
**Description**: MySQL is an open-source SQL relational database management system. As its name suggests, the database is controlled using the SQL database language. MySQL is based on the client-server model and consists of a server and one or more clients. The clients can retrieve and modify the data in the database using SQL. MySQL is ideally suited for applications such as dynamic websites and is often combined with a Linux OS, PHP, and an Apache web server (this is known as a LAMP combination (LEMP is nginx is used in place of Apache)).\\
A MySQL database translates the commands into executable code and performs the requested actions. If there is an error during the processing, the web application informs the user. Various SQL injections can generate such error messages, and those latter often contain important information regarding the MySQL version, or the correct way to interact with it (which can be different from what the developers intended).\\
Managing a SQL database and its configuration can be very complex, and it is not rare to see misconfigurations. If a MySQL server is running, its configuration file can be inspected as follows: *cat /etc/mysql/mysql.conf.d/mysqld.cnf*. This file often contains a username and its password in clear text. So if the rights for this file aren't set properly, there is a good chance to find credentials there.\\
**Footprinting**: sometimes, even though it really isn't a good practice, a MySQL server can be accessed from an external network. if it is the case, it can be footprinted as follows:
- Use nmap: *sudo nmap \<targetIP\> -sV -sC -p3306*
- Use nmap NSE scripts: *sudo nmap \<targetIP\> -sV -sC -p3306 --script mysql\**
- Interact with the MySQL Server: *mysql -u \<username\> -p\<password\> -h \<targetIP\>* (there is **no space between "-p" and the password**). Once connected, use MySQL commands (depending on the version) such as *show databases;*, *select version()*, *use \<databasename\>*, *show tables;* (within a database), *select \* from \<table\>;*, *show columns from \<table\>;*, *select \* from \<table\ where \<columnName\> = '\<string\>';*, and so on. Some databases exist by default, and the most important ones for the MySQL server are the *system schema (sys)* and *information schema (information\_schema).
  
  
# MSSQL  
**Service/protocol**: Microsoft SQL\\
**Port(s)**: \\
**Description**: unlike MySQL, MSSQL is closed source and is popular among database administrators and developers who build applications that run on Microsoft'S .NET framework. Even though some versions of MSSQL can run on Linux and MacOS, it is more likely to see it on targets running Windows.
  
  
  
  
  
  
  
