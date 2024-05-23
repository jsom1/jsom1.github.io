---
title: "Perfection"
author: "Me"
date: "May 20, 2024"
output: html_document
---

# Perfection

 <div id="boxinfo">
 <div id="textbox">
 <p class="alignleft">**Difficulty**: Easy</p>
 <p class="aligncenter">**Type**: CTF</p>
 <p class="alignright">**OS**: Linux</p>
 </div>
 <div style="clear: both;"></div>
 </div> 

<div class="img_container">
<!---
![desc]({{https://jsom1.github.io/}}/_images/htb_squashed_desc.png){: height="300px" width = 320px"}
</div>
-->
  
**Ports/services exploited:** 80/http\\
**Tools:** chatGPT\\
**Techniques:** SSTI\\
**Keywords:** 

**TL;DR**: 


## 1. Services enumeration
{:style="color:DarkRed; font-size: 170%;"}

Let's start by enumerating the running services with nmap:

<!---
<div class="img_container">
![nmap]({{https://jsom1.github.io/}}/_images/htb_squashed_nmap.png)
</div>
-->

SSH et application web, "wight-calc...". Visite la page, on voit ce qu'on peut entrer. 1ère col texte, 2ème et 3ème nombre. Try manuellement 2-3 payloads, mais marche pas.
need burp. Explications intruder et attaque, fuzzing du formulaire.
En parallèle, dirb et gobuster. Plein de faux positifs, filtre --> grosse erreur. Faux positif car en fait, il y a bien quelque chose quand on se rend sur une adresse random...
L'app derrière est en ruby (voit sur nmap). 
On dirait donc que l'app est vulnérable a une injection de code ruby... L'application a donc executé le code injecté.
Need comprendre la vulnerabilité. Ne marche pas à tous les coups ?! C'est quoi cette commande ? En fait le get '/ss' définit une route dans l'app web. 
Ca spécifie que quand l'app reçoit une requête HTTP GET pour le chemin /ss, elle doit exécuter le code "Hello World". 
Sur burp, on voit où il cherche l'image localement. Semble indiquer que l'app utilise le framework sinatra, un DSL pour créer des apps en ruby. Donc l'app sinatra fonctionne localement sur le serv.
Le port 3000 n'écoute que sur localhost et n'est pas exposé à l'extérieur. C'est pour ça que nmap ne le montre pas.
Bcp de tests avec burp, différents payloads et encodage.

Hint : SSTI. Vuln qui se produit quand l'app web permet à l'utilisateur de souettres des données qui sont ensuite incorp dans un templat côté serveur. Si l'entrée user n'est pas correctement filtrée ou échapée, un attaquant peut injecter du code qui sera executé par le moteur de template du serveur.


Need reproduire... Testé pleins de payloads et d'encodage. Certains passent, comme l'encodage URL standard... l'encodafe hexadécimal ou base64 ne passe pas (Malicious input blocked). Surtout, certains payloads passent dans burp ou curl, mais pas dans le formulaire en ligne. Ex : get%20%27/tst%27%20do%0A%20%20%22Hello%20World%22%0Aend.
Ex. de curl :

```
curl -X POST -d "category1=%3C%25%20%60ls%60%20%25%3E&grade1=90&weight1=20&category2=Math&grade2=80&weight2=20&category3=English&grade3=85&weight3=20&category4=Science&grade4=70&weight4=20&category5=History&grade5=75&weight5=20" http://10.10.11.253/weighted-grade-calc
```

Check .txt de searchsploit webrick. La version 1.7.0 est p-e vuln. 

## 2. Gaining a foothold
{:style="color:DarkRed; font-size: 170%;"}

## 4. Vertical privilege escalation
{:style="color:DarkRed; font-size: 170%;"}

p-e check le script ruby de searchsploit webrick. Condition : être sur le même subnet que le serveur.


<ins>**My thoughts**</ins>


<ins>**Fix the vulnerabilities**</ins>



