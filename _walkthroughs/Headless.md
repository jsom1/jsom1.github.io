---
title: "Headless"
author: "Me"
date: "May 27, 2024"
output: html_document
---

# Headless

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: Easy</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Linux</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<!---
<div class="img_container">
![desc]({{https://jsom1.github.io/}}/_images/htb_squashed_desc.png){: height="300px" width = 320px"}
</div>
-->
  
**Ports/services exploited:** \\
**Tools:** \\
**Techniques:** \\
**Keywords:** 

**TL;DR**: 

## 1. Services enumeration
{:style="color:DarkRed; font-size: 170%;"}

Let's start by enumerating the running services with nmap:

```
sudo nmap -sV -sC 10.10.11.8
```
The results are the following:

```
Starting Nmap 7.92 ( https://nmap.org ) at 2024-05-27 20:55 CEST
Nmap scan report for Headless (10.10.11.8)
Host is up (0.030s latency).
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey: 
|   256 90:02:94:28:3d:ab:22:74:df:0e:a3:b2:0f:2b:c6:17 (ECDSA)
|_  256 2e:b9:08:24:02:1b:60:94:60:b3:84:a9:9e:1a:60:ca (ED25519)
5000/tcp open  upnp?
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/2.2.2 Python/3.11.2
|     Date: Mon, 27 May 2024 18:56:01 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 2799
|     Set-Cookie: is_admin=InVzZXIi.uAlmXlTvm8vyihjNaPDWnvB_Zfs; Path=/
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="UTF-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1.0">
|     <title>Under Construction</title>
|     <style>
|     body {
|     font-family: 'Arial', sans-serif;
|     background-color: #f7f7f7;
|     margin: 0;
|     padding: 0;
|     display: flex;
|     justify-content: center;
|     align-items: center;
|     height: 100vh;
|     .container {
|     text-align: center;
|     background-color: #fff;
|     border-radius: 10px;
|     box-shadow: 0px 0px 20px rgba(0, 0, 0, 0.2);
|   RTSPRequest: 
|     <!DOCTYPE HTML>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <title>Error response</title>
|     </head>
|     <body>
|     <h1>Error response</h1>
|     <p>Error code: 400</p>
|     <p>Message: Bad request version ('RTSP/1.0').</p>
|     <p>Error code explanation: 400 - Bad request syntax or unsupported method.</p>
|     </body>
|_    </html>
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port5000-TCP:V=7.92%I=7%D=5/27%Time=6654D741%P=aarch64-unknown-linux-gn
SF:u%r(GetRequest,BE1,"HTTP/1\.1\x20200\x20OK\r\nServer:\x20Werkzeug/2\.2\
SF:.2\x20Python/3\.11\.2\r\nDate:\x20Mon,\x2027\x20May\x202024\x2018:56:01
SF:\x20GMT\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Leng
SF:th:\x202799\r\nSet-Cookie:\x20is_admin=InVzZXIi\.uAlmXlTvm8vyihjNaPDWnv
SF:B_Zfs;\x20Path=/\r\nConnection:\x20close\r\n\r\n<!DOCTYPE\x20html>\n<ht
SF:ml\x20lang=\"en\">\n<head>\n\x20\x20\x20\x20<meta\x20charset=\"UTF-8\">
SF:\n\x20\x20\x20\x20<meta\x20name=\"viewport\"\x20content=\"width=device-
SF:width,\x20initial-scale=1\.0\">\n\x20\x20\x20\x20<title>Under\x20Constr
SF:uction</title>\n\x20\x20\x20\x20<style>\n\x20\x20\x20\x20\x20\x20\x20\x
SF:20body\x20{\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20font-famil
SF:y:\x20'Arial',\x20sans-serif;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20background-color:\x20#f7f7f7;\n\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20margin:\x200;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20padding:\x200;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0display:\x20flex;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20jus
SF:tify-content:\x20center;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\
SF:x20align-items:\x20center;\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20height:\x20100vh;\n\x20\x20\x20\x20\x20\x20\x20\x20}\n\n\x20\x20\x
SF:20\x20\x20\x20\x20\x20\.container\x20{\n\x20\x20\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20text-align:\x20center;\n\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20background-color:\x20#fff;\n\x20\x20\x20\x20\x20\x20\
SF:x20\x20\x20\x20\x20\x20border-radius:\x2010px;\n\x20\x20\x20\x20\x20\x2
SF:0\x20\x20\x20\x20\x20\x20box-shadow:\x200px\x200px\x2020px\x20rgba\(0,\
SF:x200,\x200,\x200\.2\);\n\x20\x20\x20\x20\x20")%r(RTSPRequest,16C,"<!DOC
SF:TYPE\x20HTML>\n<html\x20lang=\"en\">\n\x20\x20\x20\x20<head>\n\x20\x20\
SF:x20\x20\x20\x20\x20\x20<meta\x20charset=\"utf-8\">\n\x20\x20\x20\x20\x2
SF:0\x20\x20\x20<title>Error\x20response</title>\n\x20\x20\x20\x20</head>\
SF:n\x20\x20\x20\x20<body>\n\x20\x20\x20\x20\x20\x20\x20\x20<h1>Error\x20r
SF:esponse</h1>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Error\x20code:\x20400<
SF:/p>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Message:\x20Bad\x20request\x20v
SF:ersion\x20\('RTSP/1\.0'\)\.</p>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Err
SF:or\x20code\x20explanation:\x20400\x20-\x20Bad\x20request\x20syntax\x20o
SF:r\x20unsupported\x20method\.</p>\n\x20\x20\x20\x20</body>\n</html>\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 95.74 seconds

```
We see SSH (OpenSSH 9.2p1 Debian 2+deb12u2), but we don't have any credentials to connect yet and recent versions of openSSH are generally well secured.\\
There is also something running on port 5000, which is identified as "Werkzeug". I already saw that running in <a href="/_walkthroughs/ScriptKiddie" target="_blank">ScriptKiddie</a>. To recall, Werkzeug is a utility framework for Python (it is often used with Flask). Here, it's the version 2.2.2 and it uses Python 3.11.2.\\
Finally, we see in the HTTP response a header "Set-Cookie" with the value **is_admin=InVzZXIi.uAlmXlTvm8vyihjNaPDWnvB_Zfs**. This could be an encrypted user session, so we have to keep that in mind.

## 2. Gaining a foothold
{:style="color:DarkRed; font-size: 170%;"}

Although we see the title "Under construction", let's visit the page to see if there is anything interesting:

<div class="img_container">
![webpage]({{https://jsom1.github.io/}}/_images/htb_headless_web.png){: height="300px" width = "400px"}
</div>

The only thing we can do is click on "For questions", which yields on the following page:

<div class="img_container">
![form]({{https://jsom1.github.io/}}/_images/htb_headless_form.png){: height="400px" width = "420px"}
</div>

Before trying anything with this formular, let's launch a *dirb* scan to search for other directories, as well as a *ffuf* scan to search for potential subdomains:

```
sudo dirb http://10.10.11.8:5000 -r
sudo ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -u http://10.10.11.8:5000 -H "Host: FUZZ.10.10.11.8:5000" -fs 2799
```
*Dirb* found 2 directories:

- http://10.10.11.8:5000/support (CODE:200\|SIZE:2363): this is the formular shown in the image above.
- http://10.10.11.8:5000/dashboard (CODE:500\|SIZE:265): it seems we can't access the dashboard, but let's head there to see what we find:

<div class="img_container">
![dashboard]({{https://jsom1.github.io/}}/_images/htb_headless_dash.png){: height="80px" width = "420px"}
</div>

This is interesting, because it is very likely linked to the cookie we saw in the header. *Ffuf* didn't find any subdomain, so we only have 3 leads now: the formular, this error message, and potentially the version of Werkzeug. Let's dig into the error message and try to modify the cookie. To do that, we could install a Firefox extension (a cookie manager), but we should also be able to do it from the menu -> Web developer -> Storage Inspector. The idea is to modify the cookie with the value we found, that is "InVzZXIi.uAlmXlTvm8vyihjNaPDWnvB_Zfs". Then, we refresh the page to see if it worked.\\
Unfortunately, as we see in the image below, it is already the cookie being used.

<div class="img_container">
![dashboard]({{https://jsom1.github.io/}}/_images/htb_headless_cookie.png){: height="300px" width = "400px"}
</div>

We see something on the right: is_admin is an array consisting of 2 strings: "InVzZXIi" and "uAlmXlTvm8vyihjNaPDWnvB_Zfs". If we decode "InVzZXIi" (Base64) with the following command:

```
echo "InVzZXIi" | base64 --decode
```

We see it translates to "user". Let's try to base64-encode "admin", and replace this part of the cookie.

```
echo -n "admin" | base64
```

This returns "YWRtaW4=", which we replace in the cookie. The new cookie is "YWRtaW4=.uAlmXlTvm8vyihjNaPDWnvB_Zfs". Let's refresh the page:

<div class="img_container">
![dashboard]({{https://jsom1.github.io/}}/_images/htb_headless_cookie2.png){: height="300px" width = "400px"}
</div>

It doesn't work... We see the parsed value on the right is not the same anymore. It was an array previously, whereas it is an object now. I tried removing the "=" from "YWRtaW4=" (it's called *URL-safe encoding* (base64URL format)), and it's recognized as an array once again. However, the error message is still the same.







Recherchez des vulnérabilités connues pour Werkzeug ou des applications Flask non sécurisées.
Vérifiez si des paramètres de débogage sont activés, ce qui pourrait permettre l'exécution de code à distance.
Injection et Manipulation de Paramètres

Testez les points d'entrée de l'application pour les vulnérabilités d'injection comme SSTI (Server-Side Template Injection) ou d'autres injections courantes.
Conclusion
Le port 5000 héberge une application web basée sur Flask avec Werkzeug comme serveur. Le cookie et la réponse HTTP initiale suggèrent qu'il pourrait y avoir des points d'entrée intéressants pour des tests de sécurité supplémentaires. Accédez à l'application, inspectez le cookie, et utilisez des outils de fuzzing pour découvrir d'autres points d'entrée possibles.




## 3. Vertical privilege escalation
{:style="color:DarkRed; font-size: 170%;"}



<ins>**My thoughts**</ins>



<ins>**Fix the vulnerabilities**</ins>

