---
layout: default
title: Klarzetten Werkomgeving
nav_order: 1
nav_exclude: false
---
Deze tutorial geeft je de nodige informatie om je eigen machine klaar te maken voor het vak [Informatica Werktuigen](https://onderwijsaanbod.kuleuven.be/syllabi/n/G0Q30EN.htm). Volg deze instructies vóór de start van de eerste oefenzitting.

<!-- TOC -->

- [Stap 1: Virtuele Machine met Ubuntu](#stap-1-virtuele-machine-met-ubuntu)
- [Stap 2: Klaarzetten Linux-omgeving](#stap-2-klaarzetten-linux-omgeving)
- [Stap 3: SSH-keys opvragen](#stap-3-ssh-keys-opvragen)
  - [1. Aanvragen SSH-keys](#1-aanvragen-ssh-keys)
  - [2. Installeren SSH-keys](#2-installeren-ssh-keys)

<!-- /TOC -->

Een videoversie van deze instructiepagina is beschikbaar:

[![Bekijk de video](https://img.youtube.com/vi/kzrfRC0TaYg/hqdefault.jpg)](https://www.youtube.com/watch?v=kzrfRC0TaYg)

# Stap 1: Virtuele Machine met Ubuntu

In dit vak zullen we meestal werken in een Linux-omgeving. Indien je reeds een Linuxdistributie geïnstalleerd hebt, kan je deze stappen overslaan.


1. Installeer het programma [VirtualBox](https://www.virtualbox.org/wiki/Downloads).
2. Download de laatste versie van het besturingssysteem [Ubuntu](https://ubuntu.com/download/desktop). Dit zou een ISO-bestand moeten zijn.
3. Gebruik VirtualBox om een nieuwe virtuele machine aan te maken. Het type van de machine is Linux, de versie Ubuntu. De naam mag je zelf kiezen.
4. Je kan zelf kiezen hoeveel RAM-geheugen en schijfruimte je alloceert aan de machine. Indien je zelf geen voorkeur hebt, gebruik dan de standaard instellingen.
5. Selecteer de ISO-file uit stap 2 wanneer de disk image gevraagd wordt.
6. Doorloop de installatie van Ubuntu.

Een meer gedetailleerde handleiding om Ubuntu te installeren in VirtualBox kan je hier vinden: [Windows](https://itsfoss.com/install-linux-in-virtualbox/),
[Mac](https://www.dev2qa.com/how-to-install-ubuntu-on-virtualbox-mac/).



# Stap 2: Klaarzetten Linux-omgeving

In dit vak zullen we gebruik maken van de volgende Ubuntu-packages:

`build-essential` `git` `texlive-latex-base` `texlive-latex-recommended` `texlive-fonts-recommended` `texlive-fonts-extra` `texlive-latex-extra` `texlive-lang-european` `ddd`

Een editor om programmacode te schrijven (C en JavaScript) zal ook nuttig zijn. Indien je reeds een voorkeur hebt voor een specifieke editor kan je die blijven gebruiken. Zo niet raden wij [Visual Studio Code](https://code.visualstudio.com) aan.

Volg onderstaande stappen om de nodige packages te installeren

1. Gebruik de toetsencombinatie <kbd>CTRL</kbd> + <kbd>ALT</kbd> + <kbd>T</kbd> in Ubuntu om een nieuwe terminal te openen.
2. Voer onderstaande commando's uit in volgorde. Let op dat je het $-teken niet mee kopieert.
   
``` shell 
$ sudo apt update
$ sudo apt upgrade
$ sudo apt install build-essential dkms linux-headers-$(uname -r)
```
3. Klik in het VirtualBox-menu op Device -> Insert Guest Additions CD Image. Voer vervolgens de installatie van de Guest Additions uit door op Run te klikken en je wachtwoord in te geven.
4. Zet na deze installatie de virtuele machine uit
5. Ga in de VirtualBox manager naar General -> Advanced en zet Shared Clipboard en Drag'n'Drop op bidirectional. Dit laat je toe om te copy-pasten naar de Virtuele Machine en om bestanden te slepen.
6. Start de machine terug op en voer onderstaand commando uit. Vanaf nu kan je commando's kopieren en plakken.
  
``` shell
$ sudo apt install -y git texlive-latex-base texlive-latex-recommended texlive-fonts-recommended texlive-fonts-extra texlive-latex-extra texlive-lang-european ddd
```
7. (Optioneel) Installeer Visual Studio Code met het commando

``` shell
$ sudo snap install --classic code
```

# Stap 3: SSH-keys opvragen

Gedurende je gehele opleiding heb je als informaticastudent 24/7 toegang tot de departmentale studentenmachines. Deze machines staan in de volgende lokalen:

* 200A 00.25 (SOL Z)
* 200A 00.24 (SOL N)
* 200A 00.124

Op deze machines staat Ubuntu geïnstalleerd met alle nodige software voor dit vak.

Je kan de machines op twee manieren gebruiken:

1. Fysiek inloggen met je studentennummer en wachtwoord (je Toledo-wachtwoord).
2. Vanop afstand inloggen via SSH.

## 1. Aanvragen SSH-keys

Inloggen vanop afstand via SSH zullen jullie aangeleerd krijgen in dit vak. Dit zal echter niet gebeuren via de traditionele username/password log-in. In plaats daarvan worden sleutelbestanden gebruikt. **Deze sleutelbestanden moet je op voorhand aanvragen**. Dat gebeurt via [deze link](https://www.cs.kuleuven.be/restricted/computerklas/).

Je sleutelbestand wordt beveiligd met een wachtwoord dat je op bovenstaande pagina moet instellen. **Vergeet dit wachtwoord niet!**

Indien je de instructies op de webpagina correct gevolgd hebt zou je normaal gezien een link moeten krijgen waarop je je sleutels kan downloaden. Deze link heeft de vorm https://www.cs.kuleuven.be/restricted/computerklas/keys/rXXXXXXX/ waarbij de X-tekens vervangen moeten worden door je eigen r-nummer.

## 2. Installeren SSH-keys

Volg nu onderstaande stappen om je sleutelbestanden toe te voegen aan je virtuele machine. Dit zal het mogelijk maken zeer eenvoudig verbinding te maken met de machines van het departement.

1. Surf in Ubuntu naar de link https://www.cs.kuleuven.be/restricted/computerklas/keys/rXXXXXXX/ waarbij je je eigen studentennummer invult op de plaats van de X-tekens.
2. Download het bestand genaamd *id_rsa* (rechtsklik -> save link as ...). Sla het bestand onder die naam op in de folder *Downloads*.
3. Open een terminal met <kbd>CTRL</kbd> + <kbd>ALT</kbd> + <kbd>T</kbd> en voer onderstaande commando's uit (zonder $-teken):
``` shell
$ mkdir ~/.ssh
$ mv ~/Downloads/id_rsa ~/.ssh
$ chmod 600 ~/.ssh/id_rsa 
```
Na deze stappen zijn je sleutels correct geïnstalleerd. De sleutels moeten wel nog door de [systeemgroep](https://system.cs.kuleuven.be/cs/system/wegwijs/systeemgroep/) geactiveerd worden. Dit gebeurt enkel op werkdagen.

In een toekomstige oefenzitting zullen jullie deze sleutels gebruiken om verbinding te maken met de departmentale computers.