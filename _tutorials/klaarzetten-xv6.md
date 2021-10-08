---
layout: default
title: Klaarzetten Xv6
nav_order: 1
nav_exclude: false
---

# Inhoudsopgave

- [Inhoudsopgave](#inhoudsopgave)
- [Leesmateriaal](#leesmateriaal)
- [Video](#video)
- [Linux-omgeving](#linux-omgeving)
- [Packages](#packages)
- [xv6](#xv6)
- [GitHub](#github)
  - [Keys genereren](#keys-genereren)
  - [Keys toevoegen aan GitHub](#keys-toevoegen-aan-github)
  - [Connectie testen](#connectie-testen)

# Leesmateriaal

In deze oefenzittingenreeks werken we met het besturingsssysteem xv6.
Ter voorbereiding van de oefenzitting vragen we om het eerste hoofdstuk (*Operating System interfaces*) van het [xv6 boek](https://github.com/besturingssystemen/xv6-riscv-book) te lezen.
[Hier](https://github.com/besturingssystemen/xv6-riscv-book/releases/latest/download/book.pdf) kan je een recente PDF editie terugvinden.

> :bulb: In de komende stappen van de voorbereiding kan er veel wachttijd zijn bij het downloaden en installeren van Ubuntu en de packages. Lees dit materiaal terwijl je moet wachten op de installaties.

# Video

In onderstaande video wordt deze tutorial overlopen.

[![Bekijk de video](https://img.youtube.com/vi/vjJW36_q_sg/hqdefault.jpg)](https://youtu.be/vjJW36_q_sg)

# Linux-omgeving

Om de oefenzittingen van dit vak succesvol af te kunnen leggen moeten jullie een werkende Linux-omgeving ter beschikking hebben.

Indien je nog geen werkende Linux-omgeving geïnstalleerd hebt kan je stap 1 van [deze tutorial](https://github.com/informaticawerktuigen/klaarzetten-werkomgeving) volgen.

> :warning: Indien je niet kiest voor Ubuntu is het de bedoeling dat je zelf uitzoekt hoe je aan alle nodige packages geraakt. De [Risc-V GNU Toolchain](https://github.com/riscv/riscv-gnu-toolchain) moet je dan waarschijnlijk zelf compileren.

# Packages

Open een terminal met <kbd>CTRL</kbd> + <kbd>ALT</kbd> + <kbd>T</kbd>.

* Update je Linux-omgeving:

    ```PowerShell
    sudo apt update
    sudo apt upgrade
    ```

* Installeer alle nodige packages in Ubuntu:

    ```PowerShell
    sudo apt install build-essential git qemu-system-misc gcc-riscv64-linux-gnu 
    ```

> :warning: Indien je een oude release van Ubuntu hebt (<20.04) zal je niet de laatste versie van de RISC-V compiler kunnen installeren. In dat geval moet je je distributie upgraden met ```sudo do-release-upgrade``` of de compiler zelf builden vanuit de RISC-V GNU Toolchain Git repository.

# xv6

Ten slotte willen we het besturingssysteem [xv6](https://github.com/besturingssystemen/xv6-riscv) clonen en compileren naar [RISC-V](https://riscv.org/):

* Clone de xv6 git-repository

    ```PowerShell
    git clone https://github.com/besturingssystemen/xv6-riscv
    ```

* Compileer xv6

    ```PowerShell
    cd xv6-riscv
    make qemu
    ```

Indien de compilatie succesvol is verlopen zal xv6 automatisch gestart worden in de huidige shell. Je krijgt nu onderstaande boodschap te zien:

```PowerShell
xv6 kernel is booting

hart 2 starting
hart 1 starting
init: starting sh
$ 
```

Sluit xv6 af met de toetsencombinatie <kbd>CTRL</kbd> + <kbd>A</kbd> gevolgd door <kbd>x</kbd>.

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

```PowerShell
ssh-keygen -t ed25519
```

* ```ssh-keygen``` zal voorstellen om een private key aan te maken in de .ssh folder. Accepteer dit met <kbd>Enter</kbd>.
* Geef vervolgens een wachtwoord in. Telkens wanneer je in de toekomst Git zal gebruiken met deze ssh-key zal je dit wachtwoord moeten ingeven.
* Je zou nu onderstaande output moeten zien:

```PowerShell
ssh-keygen -t ed25519
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

```PowerShell
Your identification has been saved in /home/u0111663/.ssh/id_ed25519
Your public key has been saved in /home/u0111663/.ssh/id_ed25519.pub
```

## Keys toevoegen aan GitHub

Je hebt nu een public/private key pair.

* Voer onderstaand commando uit:

```PowerShell
cat ~/.ssh/id_ed25519.pub
```
Je krijgt nu een output te zien die er uitziet als volgt (de exacte output met jouw public key zal natuurlijk verschillen):

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDS/KnrnRBLvxMt5gOoA2vqcY6mzaa1ww3QUX2ufQa1w u0111663@ham
```

* Selecteer de volledige output en kopieer deze naar je clipboard. Gebruik om te kopieren uit de terminal rechtsklik en copy, de toetsencombinatie <kbd>CTRL</kbd> + <kbd>C</kbd> wordt in een console voor andere doeleinden gebruikt.
* Bezoek vervolgens [deze link](https://github.com/settings/keys) en klik op ```New SSH key```.
* Plak de gekopieerde public key in het key-veld
* Geef de sleutel een naam naar keuze en voeg hem toe

## Connectie testen

Om je private key te gebruiken is het gemakkelijk deze in je console eerst toe te voegen aan je [ssh agent](https://en.wikipedia.org/wiki/Ssh-agent).

* Voeg je *private key* toe met onderstaand commando

```PowerShell
ssh-add ~/.ssh/id_ed25519
```

* Test je connectie met het commando

```PowerShell
ssh -T git@github.com
Hi <Username>! You've successfully authenticated, but GitHub does not provide shell access.
```
