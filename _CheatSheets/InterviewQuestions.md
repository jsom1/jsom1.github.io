---
title: "Interview questions"
author: "Me"
date: "June 26, 2024"
output: html_document
---

# Interview questions
{:.no_toc}

The purpose of this document is to prepare for technical questions that a recruiter might ask for a junior cybersecurity job.

<!---1. TOC
{:toc}
{:style="color:black; font-size: 150%;"}
--->


- Qu'est-ce qu'un SIEM ?
{:style="color:DarkRed; font-size: 120%;"}

Un SIEM (Security Information and Event Management) est un outil logiciel qui collecte et analyse les données de sécurité provenant de diverses sources (logs de systèmes, applications, réseaux) pour détecter des activités suspectes et des menaces potentielles en temps réels.\\
Il joue donc un rôle crucial en fournissant des alertes, en centralisant les logs, et en permettant des réponses rapides aux attaques.
Un SIEM peut également automatiser certaines réponses aux menaces, comme bloquer une adresse IP suspecte ou isoler un segment de réseau compromis. Toutefois, pour des incidents plus complexes, une intervention humaine est généralement nécessaire pour évaluer la situation et prendre des mesures appropriées.\\
Exemple de SIEM: **Splunk**.


- Qu'est-ce qu'un SOC ?
{:style="color:DarkRed; font-size: 120%;"}

Un SOC (Security Operations Center) est une équipe centralisée de sécurité chargée de surveiller, détecter, analyser et répondre aux incidents de sécurité en temps réel.\\
Il est composé d'analystes de sécurité et de technologies de surveillance qui travaillent ensemble pour protéger les systèmes informatiques d'une organisation.
Les SOC utilisent généralement des SIEM pour détecter les menaces et les anomalies.\\
En résumé, le SOC est l'équipe et l'infrastructure qui gère la sécurité, tandis que le SIEM est un outil clé utilisé par le SOC pour accomplir cette mission.


- Qu'est-ce qu'un IDS et un IPS ?
{:style="color:DarkRed; font-size: 120%;"}

**IDS** (Intrusion Detection System) : outil qui surveille le trafic pour détecter des activités suspectes en générant des alertes pour permettre une réponse appropriée (qui est manuelle).

**IPS** (Intrusion Prevention System) : similaire à l'IDS, mais va plus loin que en bloquant activement les menaces détectées pour empêcher les attaques de réussir.


- Qu'est-ce qu'un EDR, un XDR, et un MDR ?
{:style="color:DarkRed; font-size: 120%;"}

**EDR** (Endpoint Detection and Response) : technologie de sécurité axée sur les terminaux, c'est-à-dire les ordinateurs, les smartphones, les tablettes et autres dispositifs qui se connectent au réseau d'une entreprise.
L'objectif principal de l'EDR est de détecter, d'analyser, de répondre et de remédier aux menaces de sécurité directement sur ces dispositifs.\\
On distingue 2 phases : la pré-infection (gérée par un antivirus (AV) qui regarde la signature d'un malware -> précis à 50-60%) et la post-infection (on utilise l'AI pour regarder non plus la signature, mais le comportement d'un malware -> on essaie d'attraper les 40-50% qui ont passé l'AV).\\
L'EDR peut ensuite prendre des actions telles que mettre en quarantaine des fichiers ou bloquer des processus suspectés d'être malveillants.
Tout ça se passe donc au niveau d'un endpoint, alors que les bonnes attaques n'en visent pas qu’un seul mais sont plus générales ; dans les architectures d’aujourd’hui, il y a le cloud, des devices IoT, des applications, des firewalls (FW), le réseau, des endpoints, etc…
Donc un EDR ne suffit pas.\\
Exemple d'EDR : **Crowdstrike**.

**XDR** (Extended Detection and Response) : complémentaire et similaire à l’EDR, mais à une plus grande échelle.\\
Un XDR regroupe des données de différentes sources, et utilise l’AI pour détecter des patterns entre ces sources, ce qui permet de détecter des attaques plus complexes. 
Tout comme un EDR, un XDR peut ensuite appliquer des actions prédéfinies comme bloquer une IP au FW, mettre un utilisateur en quarantaine au switch, etc…\\
Exemple d'XDR : **Crowdstrike**, **Palo Alto Networks Cortex**, **Cisco XDR**.

**MDR** (Managed Detection and Response) : les EDR et XDR se concentrent sur des technologies spécifiques.\\
Un MDR est un service géré par un tiers qui fait du monitoring 24/7, mais qui contient aussi des experts spécialisés qui font aussi du threat hunting (donc pas seulement du monitoring, mais de la cyberdéfense proactive), de l'incident response, etc... 
C’est donc un service complet de gestion des menaces.


- Qu'est-ce qu'un MSS ?
{:style="color:DarkRed; font-size: 120%;"}

Un MSS (Managed Security Services) désigne des services de sécurité externalisés fournis par une entreprise tierce pour gérer et surveiller la sécurité des systèmes et des réseaux des clients.\\
Un MSSP (Managed Security Services Provider) offre généralement des services comme la gestion des firewalls, la surveillance des événements de sécurité, la détection et la réponse aux menaces, et la gestion des vulnérabilités 24/7. 
Les MSS incluent souvent :

  - Surveillance continue des réseaux et systèmes pour détecter les menaces en temps réel.
  - Gestion des incidents de sécurité et réponse rapide en cas de détection de menaces.
  - Mises à jour et gestion des appareils de sécurité comme les pare-feux, les IDS/IPS, etc.
  - Services de conseil pour améliorer la posture de sécurité globale des clients.
  - Reporting et analyse pour aider les clients à comprendre les menaces auxquelles ils sont confrontés.

Donc les avantages des MSS sont les suivants:

  - Externalisation des responsabilités de sécurité, permettant aux entreprises de se concentrer sur leurs activités principales.
  - Accès à des experts en sécurité et à des technologies avancées.
  - Surveillance continue sans la nécessité d'une infrastructure interne dédiée.


- Qu'est-ce que le threat hunting ?
{:style="color:DarkRed; font-size: 120%;"}

Il s'agit d'une démarche proactive : on n’attend pas des alertes ou autres, ce qui permet de détecter des choses que les IDS ou IPS sont susceptibles de louper (lateral movement, etc…). 
Les informations recueillies permettent aussi d’améliorer ensuite les SIEM ou EDR. Étapes du threat hunting :

  - Formuler une hypothèse de chasse (d’après les tendances récentes ou anomalies observées) : on définit ce qu’on cherche à découvrir ou à confirmer dans la chasse.
  -	Collecte et analyse des données : récolte de données de différentes sources : logs, données réseau, données d’endpoints, etc… Analyse des données à l’aide d’outils pour détecter des anomalies.
  -	Investigation et recherche de menaces : recherche d'indicateurs de compromission (**IoC**) ou de signes d’intrusion (fichiers suspects, connexions inhabituelles, comportements étranges, etc…). Ensuite, on investigue pour comprendre la nature, l’origine, et l’étendue de la menace.
  -	Réponse aux menaces : on isole les systèmes affectés pour éviter la propagation de la menace. Ensuite, on prend des mesures pour l'éliminer et réparer les systèmes affectés.
  -	Documentation et apprentissage : on documente les découvertes, méthodes, leçons apprises. On utilise ces informations pour améliorer les processus de sécurité et affiner les mécanismes de détection.


- Qu'est-ce que le modèle OSI ?
{:style="color:DarkRed; font-size: 120%;"}

Le modèle OSI (Open System Intercommunication) est un modèle théorique décrivant des standards pour rendre possible la communication entre 2 systèmes, quelle que soit leur architecture.
Concrètement, on peut par exemple envoyer un mail depuis gmail sur Windows à quelqu'un qui va l'ouvrir sur Protonmail sur un Mac, sans se poser de questions.\\
Le modèle est composé de 7 couches. Les 3 premières sont des couches hardware, les 3 dernières sont des couches software, et la 4ème au milieu fait le lien entre les deux:

  - Physical layer : son rôle est de transmettre des données (bits) sur un support (câble, air (wifi), etc...). Il s'agit donc de spécifications mécanique et électriques. L'équipement hardware associé serait un *hub*.
  - Data link layer : son rôle est d'organiser les bits en frame, et de les transférer entre des machines sur un **même réseau**. Cette couche utilise l'adresse MAC (Media Access Control), qui est l'adresse physique de la carte réseau de la machine. L'équipement hardware associé serait un *switch*.
  - Network layer : son rôle est d'envoyer des paquets d'une source à une destination, mais aussi sur des **réseaux différents** en n'utilisant non plus l'adresse MAC, mais l'adresse IP (Internet Protocol). Cette couche gère donc l'adressage et le routage. L'équipement hardware associé serait le *routeur*.
  - Transport layer : son rôle est de s'assurer que les données transitent correctement à travers les réseaux. Les 2 protocoles sont UDP (User Datagram Protocol) et TCP (Transmission Control Protocol).
  - Session layer : son rôle est de fournir des services pour établir, maintenir et terminer des sessions ou connexion entre applications. Cette couche facilite donc la communication entre applications des des réseaux différents (ex : RPC (Remote Procedure Call).
  - Presentation layer : son rôle est de présenter et formatter les données entre systèmes (encryption/décryption, compression/décompression, etc...)
  - Application layer : son rôle est de gérer l'interface entre les utilisateurs et les applications.

Il faut noter qu'il s'agit surtout d'un modèle théorique. En pratique, on utilise plutôt le modèle TCP/IP (qui est semblable au modèle OSI dans les principes).


- Qu'est-ce que le modèle TCP/IP ?
{:style="color:DarkRed; font-size: 120%;"}

Le modèle TCP/IP est "la vraie implémentation" du modèle OSI. 
Il s'agit d'une suite de protocoles de communication utilisée pour interconnecter les dispositifs sur Internet. 
Contrairement au modèle OSI, il est composé de quatre couches : la couche d'accès réseau, la couche Internet, la couche transport et la couche application.


- Quelle est la différence entre les protocoles TCP et UDP ?
{:style="color:DarkRed; font-size: 120%;"}

Les 2 sont des protocoles utilisés dans la couche transport du modèle OSI.

**TCP** (Transmission Control Protocol) : TCP est plus utilisé, car il vérifie que les packets aient bien été reçus, et et qu’ils soient dans le bon ordre. Si ce n’est pas le cas, ça redemande l’information. 

**UDP** (User Datagram Protocol) : contrairement à TCP, il n'y a pas de vérification de bonne réception des paquets donc c'est un peu moins fiable. En revanche, l'avantage est que c'est plus rapide. Donc UDP est utilisé pour le streaming vidéo par exemple).


- Qu'est-ce qu'un firewall (FW) et quelles sont les étapes à suivre pour le configurer ?
{:style="color:DarkRed; font-size: 120%;"}

Un FW est un dispositif qui contrôle le trafic entrant et sortant en fonction de règles de sécurité prédéfinies. 
Il peut être configuré pour bloquer ou autoriser le trafic en fonction de critères comme l’adresse IP, le port ou le protocole.
Pour configurer un firewall, on suit généralement les étapes suivantes :

1. Modifier le username / password par défaut
2. Désactiver l'administration à distance
3. Configurer le port forwarding pour certaines applications (web server ou serveur FTP)
4. Désactiver le serveur DHCP du FW s’il y en a déjà un dans le réseau (sinon il pourrait y avoir des conflits)
5. S’assurer que les logs soient activés
6. S'assurer que le FW est configuré pour respecter les policies


- Qu'est-ce qu'un IVS ?
{:style="color:DarkRed; font-size: 120%;"}

Un Internal Vulnerability Scanner (IVS) analyse les systèmes internes pour identifier les failles de sécurité, les configurations incorrectes et les logiciels obsolètes qui pourraient être exploités. 
Il joue donc un rôle crucial dans la sécurité proactive en permettant de corriger les vulnérabilités avant qu’elles ne soient exploitées. 
Les IVS effectuent des scans réguliers pour détecter les vulnérabilités **connues**.


- Quelle est la différence entre OT et IT ?
{:style="color:DarkRed; font-size: 120%;"}

**IT** (Information Technology) : les systèmes IT collectent, traitent et stockent les données qui aident au business decision-making et à la communication.
**OT** (Operational Technology) : les système OT contrôlent et surveillent les équipement physiques ainsi que les processus de fabrication dans des industries. Ils se concentrent sur des données en temps réel.


- Comment sécuriser un serveur ?
{:style="color:DarkRed; font-size: 120%;"}

Il faut suivre les étapes suivantes :

  - Mettre à jour la propriété du fichier
  - Garder le serveur web à jour en faisant les mises à jour
  - Désactiver les modules supplémentaires et inutilisés sur le serveur
  - Supprimer les scripts par défaut


- Qu'est-ce qu'AD ?
{:style="color:DarkRed; font-size: 120%;"}

Active Directory (AD) est un service de gestion des identités et des accès développé par Microsoft pour les environnements Windows. 
Il permet aux administrateurs de gérer les utilisateurs, les ordinateurs, les groupes et les ressources au sein d'un réseau d'entreprise. 
AD organise ces éléments en un annuaire structuré, facilitant l'authentification, l'autorisation et la gestion centralisée des ressources et des politiques de sécurité. 
Il est couramment utilisé pour gérer les droits d'accès et les permissions, ainsi que pour déployer des logiciels et des mises à jour à l'échelle d'une organisation.


- Quelles sont quelques cyberattaques courantes ?
{:style="color:DarkRed; font-size: 120%;"}

  - Malware
  - Phishing
  - Attaques brute-force sur des mots de passe
  - DDoS
  - MitM (man in the middle)


- Qu'est-ce que SSH ?
{:style="color:DarkRed; font-size: 120%;"}

SSH (Secure Shell) est un "shell sécurisé" (ou terminal) et c'est la méthode la plus courante pour de gestion de serveurs Linux.
Ce protocole permet de dialoguer avec une machine ou un serveur à distance. 


- Qu'est-ce que SSL, TLS, et quelle est la différence entre les deux ?
{:style="color:DarkRed; font-size: 120%;"}

Il s'agit de protocoles de sécurité utilisés pour sécuriser les communications sur internet.

**SSL** (Secure Sockets Layer) : vérifie l’identité de l’expéditeur et aide à suivre la personne avec laquelle on communique.
**TLS** (Transport Layer Security) : version plus récente et plus sécurisée de SSL, qui offre un canal sécurisé entre 2 clients.


- Le protocole SSL est-il suffisant pour assurer la sécurité du réseau ?
{:style="color:DarkRed; font-size: 120%;"}

SSL vérifie l'identité de l'expéditeur, mais n'assure pas la sécurité une fois les données transférées vers le serveur. 
Il est donc recommandé d'encrypter les données côté serveur pour le protéger.


- Qu'est-ce que le processus de salage ?
{:style="color:DarkRed; font-size: 120%;"}

Le processus de salage (ou salting en anglais) est une technique utilisée pour renforcer la sécurité des mots de passe stockés dans les bases de données. 
Il consiste en :

  - L'ajout d'une valeur aléatoire : Lorsqu'un utilisateur crée un mot de passe, une valeur aléatoire unique appelée "sel" (ou "salt") est générée et ajoutée au mot de passe avant son hachage.
  - Le hachage du mot de passe salé : Le mot de passe, combiné avec le sel, est ensuite passé à travers une fonction de hachage pour produire un hachage salé. Par exemple, si le mot de passe est "password123" et le sel est "abc123", le hachage serait calculé sur "password123abc123".
  - Le stockage du hachage et du sel : Le hachage résultant et le sel utilisé sont stockés dans la base de données. Chaque mot de passe a son propre sel unique, ce qui signifie que même si deux utilisateurs ont le même mot de passe, les hachages stockés seront différents en raison des sels différents.

Le salage permet donc de se protéger contre les attaques par rainbow tables : les rainbow tables sont des pré-calculs de hachages de mots de passe courants. En ajoutant un sel unique, même les mots de passe communs produisent des hachages uniques, rendant ces tables inefficaces.
Aussi, ça complexifie les attaques par force brute car l'attaquant doit recalculer le hachage pour chaque combinaison de mot de passe et de sel.
En résumé, le salage améliore considérablement la sécurité des mots de passe en garantissant que même les mots de passe identiques produisent des hachages différents, compliquant ainsi les tentatives de compromission des mots de passe.


- Qu'est-ce qu'une attaque CSRF ?
{:style="color:DarkRed; font-size: 120%;"}

Un CSRF (Cross-Site Request Forgery) est une attaque où un attaquant trompe un utilisateur authentifié pour qu'il effectue une action non désirée sur une application web à laquelle il est connecté. 
Cela se fait généralement en incitant l'utilisateur à cliquer sur un lien malveillant ou à visiter une page contenant un script malveillant (email de phishing, lien sur un forum, etc...).
La page malveillante contient un code (comme une balise HTML, un formulaire automatique ou un script) qui envoie une requête HTTP à l'application web cible. 
Cette requête est exécutée avec les mêmes droits que ceux de l'utilisateur authentifié, car elle inclut automatiquement les cookies de session de l'utilisateur.


- Qu’est-ce que ARP et l’ARP poisoning ?
{:style="color:DarkRed; font-size: 120%;"}

**ARP** (Address Resolution Protocol) : protocole utilisé pour associer une adresse IP à une adresse MAC sur un réseau local. Protocole qui fonctionne comme une interface entre le réseau OSI et la couche liaison OSI.

**ARP poisoning** : attaque où un attaquant envoie de fausses informations ARP sur un réseau local pour détourner le trafic vers sa propre machine.


- Qu'est-ce qu'un jeton d'accès (token) ?
{:style="color:DarkRed; font-size: 120%;"}

Un jeton d'accès est une chaîne de caractères utilisée pour authentifier un utilisateur sur un réseau ou une application. 
Il permet l'accès aux ressources sans nécessiter de fournir des identifiants à chaque requête.


- Qu’est-ce que le cracking WEP ?
{:style="color:DarkRed; font-size: 120%;"}

Le cracking WEP (Wired Equivalent Privacy) est le processus de décryptage de la clé de sécurité utilisée dans les réseaux Wi-Fi protégés par WEP. 
Exemple d'outil de cracking WEP : Aircrack-ng.


- Qu'est-ce que la règle 80/20 dans un réseau ?
{:style="color:DarkRed; font-size: 120%;"}

La règle 80/20 stipule que 80 % du trafic réseau se produit localement (à l'intérieur du réseau local) et seulement 20 % va à l'extérieur (à travers le WAN).


- Qu'est-ce qu'une adresse IP ?
{:style="color:DarkRed; font-size: 120%;"}

L'adresse IP, associée à un masque, donne deux informations importantes : le réseau dans lequel se trouve la machine à joindre, et l'adresse de la machine dans ce réseau.
Une adresse IP est codée sur 32 bits, ou 4 bytes. On utilise la notation décimale.
Le masque permet donc de séparer l'adresse IP en deux adresses. Les bits à 1 dans le masque représentent la partie de l'adresse réseau, et les bits à 0 représentent la partie de l'adresse machine.
Exemple: 192.168.0.1 avec le masque 255.255.0.0. En binaire, cela donne:\\
192.168.0.1 --> 11000000.10101000.00000000.00000001\\
255.255.0.0 --> 11111111.11111111.00000000.00000000\\
La partie réseau de l'adresse est donc 192.168, et la partie machine est 0.1.


- Qu'est-ce que le protocole Ethernet ?
{:style="color:DarkRed; font-size: 120%;"}

Ethernet est le langage utilisé par les machines pour communiquer entre elles. Puisqu'elles ont des architectures différentes (Mac OS, Windows, Linux), il leur faut un langage commun pour se comprendre.
Dans le support (air ou câbles), ce qui circule n'est qu'une suite de 0 et de 1, par exemple "00110011001010101010000101001111" ; sans protocole qui définisse le sens de cette information (l'ordre des bits), cela ne veut absolument rien dire.
Il faut au moins 3 choses dans un message : l'adresse de la personne qui l'envoie, celle de celui qui le reçoit, et le message lui-même.
Le protocole Ethernet définit le format des messages envoyés sur le réseau (les frames). Il spécifie l'ordre des éléments, les protocoles à utiliser dans les différentes couches du modèle OSI, etc...).


- Qu'est-ce qu'un honeypot ?
{:style="color:DarkRed; font-size: 120%;"}

Un honeypot est un système informatique configuré pour attirer et piéger les attaquants en simulant des failles de sécurité afin de les observer et de comprendre leurs méthodes d'attaque.


- Quelle est la différence entre HIDS et NIDS ?
{:style="color:DarkRed; font-size: 120%;"}

**HIDS** (Host-based Intrusion Detection System) : système de détection d'intrusion installé sur un seul hôte ou dispositif pour surveiller et analyser l'activité suspecte.

**NIDS** (Network-based Intrusion Detection System) : système de détection d'intrusion qui surveille et analyse le trafic réseau pour détecter des activités malveillantes.


- Qu'est-ce que l'OWASP ?
{:style="color:DarkRed; font-size: 120%;"}

L'OWASP (Open Web Application Security Project) est une organisation mondiale qui se consacre à améliorer la sécurité des logiciels. Ils fournissent des ressources et des outils pour aider à sécuriser les applications web.
Ils listent aussi le top 10 des vulnérabilités les plus courantes (injections, XXE (XML External Entities), XSS, Insecure deserialization, SSRF, Misconfigurations, etc...)


- Qu'est-ce qu'un cheval de troie ?
{:style="color:DarkRed; font-size: 120%;"}

Un cheval de troie est un type de logiciel malveillant qui se fait passer pour un programme légitime tout en effectuant des actions malveillantes à l'insu de l'utilisateur.


- Quelle est la différence entre Diffie-Hellman et RSA ?
{:style="color:DarkRed; font-size: 120%;"}

Diffie-Hellman est un protocole de chiffrement utilisé pour échanger des clés de manière sécurisée (permet à deux parties de générer une clé secrète partagée sur un canal de communication non sécurisé sans échanger la clé elle-même), tandis que RSA est un algorithme de cryptographie asymétrique utilisé pour le chiffrement et la signature numérique (RSA utilise une paire de clés, une clé publique pour le chiffrement et une clé privée pour le déchiffrement. Les clés ne sont pas éphémères et peuvent être utilisées à long terme).
Diffie-Hellman est souvent utilisé en conjonction avec d'autres algorithmes pour sécuriser les communications, comme TLS.\\
RSA est utilisé dans de nombreux protocoles de sécurité pour le chiffrement des données, la signature numérique, et l'authentification, tels que TLS, SSH, et PGP


- Qu'est-ce que SOAR ?
{:style="color:DarkRed; font-size: 120%;"}

Un SOAR (Security Orchestration, Automation, and Response) est une technologie qui permet de collecter des données sur les menaces et d'automatiser les réponses aux incidents de sécurité. 
Il aide donc les équipes de sécurité à gérer et à coordonner les réponses aux incidents de manière plus efficace et rapide.


- Qu'est-ce qu'un WAF ?
{:style="color:DarkRed; font-size: 120%;"}

Un Web Application Firewall (WAF) est une solution de sécurité qui surveille, filtre et bloque les requêtes HTTP/HTTPS malveillantes vers une application web.\\
Il permet donc de protéger l'application contre des attaques web classiques telles que des injections SQL, du cross-site scripting, etc...


- Qu'entend-on par red team, blue team, purple team ?
{:style="color:DarkRed; font-size: 120%;"}

**Red Team** : Équipe qui simule des attaques pour tester la sécurité d'une organisation.
**Blue Team** : Équipe qui défend contre les attaques et surveille les menaces de sécurité.
**Purple Team** : Équipe qui combine les efforts des équipes red et blue pour améliorer les défenses de sécurité grâce à une collaboration accrue.


- Qu'est-ce qu'un attaque DDoS et comment l'empâcher ?
{:style="color:DarkRed; font-size: 120%;"}

Une attaque DDos (Distributed Denial of Service) est une attaque qui met hors service un serveur. Il y a 2 types de DDoS :

  - Flooding attacks : le hacker envoie un énorme volume de données au serveur qu’il ne peut pas gérer et donc le fait crasher. Généralement menée en utilisant des programmes automatisés (botnets) qui envoient des paquets non-stop au serveur.
  - Crash attacks : le hacker exploite un bug sur le serveur qui fait qu’il crash et ne peut plus répondre aux autres client.

On peut éviter des attaques DDoS en utilisant des logiciels anti DDoS, en configurant les FW et routeurs, en utilisant du load-balancing, etc...


- Qu'est-ce qu'un botnet ?
{:style="color:DarkRed; font-size: 120%;"}

C’est un réseau de dispositifs informatiques infectés par des logiciels malveillants, contrôlés à distance par un attaquant pour mener des activités malveillantes comme des DDoS, spam, vol d’informations.


- Quelle est la différence entre une menace, une vulnérabilité, et un risque ?
{:style="color:DarkRed; font-size: 120%;"}

**Menace** : potentiel événement ou agent capable de causer des dommages à un système.
**Vulnérabilité** : faiblesse ou faille dans un système ou une app qui peut être exploitée par une menance.
**Risque** : probabilité qu’une menace exploite une vulnérabilité pour causer un dommage, combinée à l’impact potentiel de ce dommage.


- Qu'est-ce que l'encryption SSL ?
{:style="color:DarkRed; font-size: 120%;"}

SSL (Secure Sockets Layer) est un standard de sécurité pour créer des connexions encryptées entre un serveur web et un browser, permettant de garantir la data privacy et pour protéger les données.
Les étapes pour établir une connexion SSL sont les suivantes :

  - Le browser essaie de se connecter à un serveur qui utilise SSL
  - Le serveur envoie au browser une copie de son certificat SSL
  - Le browser vérifie l'authenticité de ce certficat. Si il est ok, le browser demande au serveur d'établir une connexion encryptée
  - Le serveur web répond avec un "ACK" pour commencer une connexion encryptée
  - La communication encryptée entre le serveur et le browser commence


- Qu'est-ce qu'un VPN ?
{:style="color:DarkRed; font-size: 120%;"}

Un VPN (Virtual Private Network) permet de créer une connexion sécurisée et encryptée. Lorsqu'on utilise un VPN, les données du client sont envoyées vers un serveur qui les encrypte et les envoie sur internet vers un autre serveur qui les décrypte et les transmets au récipient final, et vice-versa.
En résumé, un VPN permet d'envoyer des données encryptées.


- Quels sont les différentes réponses d'un serveur ?
{:style="color:DarkRed; font-size: 120%;"}

  - 1xx - Réponse informatives
  - 2xx - Succès
  - 3xx - Redirection
  - 4xx - client-side error
  - 5xx - server-side error


- Qu'est-ce qu'une attaque MitM et comment la prévenir ?
{:style="color:DarkRed; font-size: 120%;"}

Une attaque MitM (Man-in-the-Middle) est un type d'attaque où le hacker se place entre deux personnes qui communiquent et vole les informations.\\
Si 2 personnes A et B communiquent, le hacker se joint à la discussion, et il se fait passer pour la personne B auprès de A, et pour la personne A auprès de B.
Les données de A et B transitent donc par le hacker, qui les redirige comme si de rien n'était. Les personnes A et B ne se rendent compte de rien.

On peut empêcher cette attaque en utilisant un VPN.


- Qu'est-ce que la CIA triad ?
{:style="color:DarkRed; font-size: 120%;"}

Il s'agit d'un modèle utilisé pour mettre en place des bonnes pratiques de sécurité.
**Confidentiality** : les données doivent être accessibles et lisibles que par le personnel autorisé. Elles doivent être encryptées au cas où quelqu’un arriverait à y accéder.\\
**Integrity** : les données ne doivent pas avoir été modifiées par quelqu'un de pas autorisé. Donc l’intégrité s’assure que les données ne sont pas corrompues. S’il y a une tentative, elles doivent doivent être restaurées dans un état précédent leur corruption.\\ 
**Availability** : les données doivent être accessibles dès qu’un utilisateur autorisé en a besoin. Il faut maintenir le hardware, faire les mises à jour, faire des backups, éviter les network bottlenecks, etc..


- Qu'est-ce que la cryptographie ?
{:style="color:DarkRed; font-size: 120%;"}

La cryptographie est la pratique et l'étude des techniques permettant de sécuriser les communications et les données en les transformant en un format illisible, uniquement déchiffrable par ceux qui possèdent la clé appropriée.


- Quels sont les signes d'intrusion sur un système ?
{:style="color:DarkRed; font-size: 120%;"}

**Activité inhabituelle sur le réseau** : connexion innattendue d’adresse ip externes, gros volumes de données vers des destinations inconnues ou des protocoles rarement utilisés, traffic à des heures étranges pendant lesquelles le réseau est généralement calme.\\
**Changement inattendus du système** : changement de taille de fichiers, timestamps, permissions, création de nouveaux comptes utilisateurs, changement de comptes existants, modifications de fichiers système ou de configuration.\\
**Présence de malware ou de fichier suspicieux** : noms étranges, dans des répertoires étranges, avec des extensions étranges, des fichiers exécutables dans les répertoires tmp, détection de signatures de malware connues par un AV, etc...\\
**Comportement utilisateur étrange** : tentatives de login échouées.\\
**Problèmes de performances** : ralentissements/crashes inattendus, haute utilisation du CPU ou de la mémoire.\\


- Quelle est la différence entry l'encryption symétrique et asymétrique ?
{:style="color:DarkRed; font-size: 120%;"}

Dans l'encryption symétrique, on utilise la même clé pour encrypter / décrypter (donc une seule clé).\\
Dans l'encryption asymétrique, on utilise des clés différentes (clé privée et clé publique).\\

On peut utiliser l'encryption asymétrique pour initialiser une conversation, puis symétrique ensuite car c'est plus rapide.


- Qu'est-ce que traceroute ?
{:style="color:DarkRed; font-size: 120%;"}

Traceroute est un outil qui suit la route que prend un paquet pour rejoindre sa destination, et ça identifie les routeurs par lesquels il passe. 
Ça permet de voir là où il y a un problème de routage et de corriger le problème.


- Quelles méthodes utiliser pour renforcer l'authentification ?
{:style="color:DarkRed; font-size: 120%;"}

Il faut des mots de passe fort (mélangeant caractères spéciaux, lettres et chiffres), ainsi qu'utiliser la 2FA ou MFA.


- Qu'est-ce que le "three-way handshake" ?
{:style="color:DarkRed; font-size: 120%;"}

Il s'agit de la méthode utilisée dans les réseaux TCP/IP pour créer une connexion entre un hôte et un client. Il consiste en 3 étapes :
  - Le client envoie un packet SYN (Synchronize) au serveur pour voir s'il est disponible
  - Le serveur renvoie un packet SYN-ACK pour confirmer qu'il est prêt à discuter
  - Le client accuse bonne réception et envoie un packet ACK (acknowledgement) au serveur.

A ce stade, la connexion est établie et les deux parties peuvent s'envoyer des données.


- Qu'est qu'une attaque XXE ?
{:style="color:DarkRed; font-size: 120%;"}

Une attaque XXE (XML External Entitites) est une attaque visant les applications qui parse des input XML.
Elle se produit lorsqu'une entrée XML contenant une référence à une entité externe est traitée par un analyseur XML faiblement configuré.

- Qu'est qu'une injection SQL ?
{:style="color:DarkRed; font-size: 120%;"}

Une injection SQL est une technique d'attaque qui exploite des vulnérabilités dans une application web en manipulant les requêtes SQL pour accéder, modifier ou supprimer des données dans une base de données non autorisée.\\
Si une telle vulnérabilité existe, l'attaquant peut faire ce qu'il veut. Par exemple sur un site de vente en ligne, il pourrait modifier le prix d'un produit.\\
Exemple d'injection: admin' or 1=1;\\
Lorsqu'on entre un username et password sur un portail de login, la requête SQL derrière ressemble à ceci:\\

SELECT * FROM users WHERE username = 'username' AND password = 'password';

En insérant "admin' or 1=1;" dans le champ username, la requête devient:

SELECT * FROM users WHERE username = 'username' OR 1=1; AND password = 'password';

La condition "OR 1=1" est toujours vraie, quel que soit le mot de passe. La partie "AND password='password" est donc ignorée, puisque la condition précédente (username = 'admin' OR 1=1) est satisfaite.\\
En résumé, on contourne la vérification du mot de passe.\\
On peut prévenir ce type d'injections en validant les entrées des utilisateurs, en échappant les caractères spéciaux et en utilisant des requêtes préparées.\\


- Quelles est la différence entre un malware, un virus, et un ver ?
{:style="color:DarkRed; font-size: 120%;"}

**Malware** : Les logiciels malveillants sont un terme général qui englobe tous les logiciels conçus pour nuire (virus, vers, ransomware, etc...)

**Virus** : les virus peuvent se propager d’un ordinateur à un autre à l’intérieur des fichiers. 
Pour activer le virus, quelqu’un doit le déclencher par une action externe. 
Par exemple, si un virus est intégré dans une feuille de calcul, il faut qu'une personne l'ouvre pour déclencher le virus. Tant qu'il n'a pas été ouvert, l'ordinateur n'est pas forcément infecté.

**Ver** : avec un ver, il n’est pas nécessaire que la victime ouvre des fichiers ou même clique sur quoi que ce soit. 
Le ver peut à la fois s’exécuter et se propager à d’autres ordinateurs. 
Comme un ver a la capacité de se propager automatiquement, il est possible qu'un ordinateur soit infecté simplement parce qu’il se trouve sur le même réseau qu’un autre appareil infecté.


- Qu'est-ce que le NAT ?
{:style="color:DarkRed; font-size: 120%;"}

Le NAT (Network Address Translation) est un mécanisme permettant à un routeur de faire correspondre des adresses IP à d'autres adresses IP.\\
Ceci est utilisé par exemple pour permettre à des machines ayant des adresses privées dans un intranet (ces adresses ne sont ni uniques ni routables en dehors de l'intranet) de communiquer avec le reste d'internet.
Le routeur leur donne des adresses externes publiques, uniques et routables.\\
Ainsi, on peut faire correspondre une seule adresse externe publique visible surinternet à toutes les adresses d'un réseau privé.\\
Concrètement, le routeur modifie les adresses IP dans les en-têtes des paquets. En plus de permettre la communication avec le reste d'internet, le NAT permet de :
  - Economiser les adresses IP publiques (puisque plusieurs machines sur un réseau local utilisent une seule adresse IP publique pour accéder à internet)
  - Améliorer la sécurité en masquant les adresses IP internes


- Pourquoi un attaquant créerait-il une machine virtuelle ?
{:style="color:DarkRed; font-size: 120%;"}

Il y a plusieurs raisons possibles :

  - Evasion : si un attaquant se doute qu'il y a un EDR, il peut créer une VM pour le contourner. En effet, l'EDR installé sur le système hôte ne voit pas ce qu'il se passe dans la VM.
  - Persistence : l'attaquant peut configurer une backdoor dans la VM pour maintenir un accès persistant. Par exemple, il peut ajouter un cronjob pour exécuter automatiquement des scripts malveillants au démarrage de la VM ou à intervalles réguliers.
  - Isolation : utiliser une VM permet de bénéficier des ressources de la machine hôte tout en isolant les activités, évitant ainsi de perturber l'hôte et d'attirer l'attention.
  - Test : une VM offre un environnement isolé pour installer et tester des malwares sans risque de détection.
  - Attaque : une VM peut être utilisée pour lancer des attaques internes contre d'autres machines du même réseau. 


- Quels sont quelques ports/protocoles communs ?
{:style="color:DarkRed; font-size: 120%;"}

Voici quelques services fréquents :

  - 21 - FTP
  - 22 - SSH
  - 23 - Telnet
  - 25 - SMTP
  - 53 - DNS
  - 80 - HTTP, 443 - HTTPS (http over SSL)
  - 3306 - MySQL
  - 3389 - RDP
  - 5432 - PostgreSQL


- Comment restez-vous informé de l'actualité en cybersécurité ?
{:style="color:DarkRed; font-size: 120%;"}

Il y a 3 moyens de rester informé :

  - En lisant des blogs/sites, par exemple comme *The Hacker News* ou *Dark Reading*.
  - En lisant des forums ou en faisant partie de communautés, par exemple le subreddit *netsec*
  - En allant à des conférences ou en suivant des webinaires, par exemple la *DEF CON*.



