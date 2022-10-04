---
layout: default
title: GitHub SSH Authenticatie
nav_order: 2
nav_exclude: false
---

# GitHub

De oefenzitting en permanente evaluatie zullen via [GitHub](https://github.com/) verspreid worden.
Maak een account op GitHub indien je er nog geen hebt.

Om je te authenticeren bij GitHub via de command line zullen we gebruik maken van [public key authentication](https://www.ssh.com/academy/ssh/public-key-authentication).
Hiervoor genereren we een public en private key pair.

De private key is je *persoonlijke sleutel*.
Dit bestand zal je nodig hebben op elke machine of virtuele machine waarmee je jezelf wil authenticeren bij GitHub via de command line.
Bewaar dit bestand goed.

De public key is in feite het bijhorende sleutelgat.
Je zal de public key naar GitHub uploaden.
Enkel jouw private key zal passen op de bijhorende public key.

## Keys genereren

> :bulb: Indien je reeds een public/private key pair gebruikt op andere plaatsen kan je je public key hergebruiken voor GitHub.
> Je moet dan geen nieuwe keys genereren.

* Open je terminal en voer het onderstaande commando uit

```console
$ ssh-keygen -t ed25519
```

* ```ssh-keygen``` zal voorstellen om een private key aan te maken in de .ssh folder. Accepteer dit met <kbd>Enter</kbd>.
* Geef vervolgens een wachtwoord in. Telkens wanneer je in de toekomst Git zal gebruiken met deze ssh-key zal je dit wachtwoord moeten ingeven.
* Je zou nu onderstaande output moeten zien:

```console
$ ssh-keygen -t ed25519
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/u0111663/.ssh/id_ed25519): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/u0111663/.ssh/id_ed25519
Your public key has been saved in /home/u0111663/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:dcFDhs8w06YATKsu7NtcxwDvIxV0CiRFgl4DiEBNFXo u0111663@ham
The key's randomart image is:
+--[ED25519 256]--+
|*+*=*==..  =+    |
|+  *.o.+. =.=.   |
|. ..oE+  ..O..   |
| .  .+ . ...o    |
|    . + S        |
| . . o o         |
|  o o + o        |
| . + o o         |
|  o.o            |
+----[SHA256]-----+
```

* Onderstaande lijnen geven de locatie aan van respectievelijk je private en public key:

```console
Your identification has been saved in /home/u0111663/.ssh/id_ed25519
Your public key has been saved in /home/u0111663/.ssh/id_ed25519.pub
```

## Keys toevoegen aan GitHub

Je hebt nu een public/private key pair.

* Voer onderstaand commando uit:

```console
$ cat ~/.ssh/id_ed25519.pub
```

Je krijgt nu een output te zien die er uitziet als volgt (de exacte output met jouw public key zal natuurlijk verschillen):

```console
$ ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDS/KnrnRBLvxMt5gOoA2vqcY6mzaa1ww3QUX2ufQa1w u0111663@ham
```

* Selecteer de volledige output en kopieer deze naar je clipboard. Gebruik om te kopieren uit de terminal rechtsklik en copy, de toetsencombinatie <kbd>CTRL</kbd> + <kbd>C</kbd> wordt in een console voor andere doeleinden gebruikt.
* Bezoek vervolgens [deze link](https://github.com/settings/keys) en klik op ```New SSH key```.
* Plak de gekopieerde public key in het key-veld
* Geef de sleutel een naam naar keuze en voeg hem toe

## Connectie testen

Om je private key te gebruiken is het gemakkelijk deze in je console eerst toe te voegen aan je [ssh agent](https://en.wikipedia.org/wiki/Ssh-agent).

* Voeg je *private key* toe met onderstaand commando

```console
$ ssh-add ~/.ssh/id_ed25519
```

* Test je connectie met het commando

```console
$ ssh -T git@github.com
Hi <Username>! You've successfully authenticated, but GitHub does not provide shell access.
```
