---
layout: default
title: Permanente evaluatie
nav_order: 4
parent: "Zitting 6: File Systems"
---

# Permanente evaluatie

Bestanden in Unix-like systemen bevatten access permissions, ook wel *modes* genoemd.
Unix-bestanden kunnen read-only, write-only of execute-only gemaakt worden, of een combinatie van deze drie
In deze permanente evaluatie voegen we een simpele variant van file permissions toe aan xv6.

File permissions zullen we voorstellen door een veld `mode`.
De `mode` van een bestand bepaalt of het leesbaar, schrijfbaar of uitvoerbaar is.
Definieer `mode` als een `uint`.
Bit 0 (de *least significant bit* of meest rechtse bit) van het `mode` veld geeft aan of een bestand readable is.
Bit 1 van het `mode` veld geeft aan of een bestand writable is.
Bit 2 van het `mode` veld geeft aan of een bestand executable is.

> :bulb: Stel dat we de waarde 5 bewaren in `mode`. Binair is `5` gelijk aan `0..00101`. Dit wil zeggen dat een bestand met mode 5 readable en executable is.

## Wijzigingen aan de base repository

Om de integratie met de automatische tests te vereenvoudigen hebben we reeds enkele zaken toegevoeged aan jullie xv6 repository.

1. In `kernel/stat.h` hebben we de constanten `M_READ`, `M_WRITE`, `M_EXECUTE` en `M_ALL` toegevoegd. Gebruik deze constanten in jullie oplossing.
2. We hebben de system call `chmod` (die je in deel 2 zal implementeren) deels toegevoegd. We hebben deze gedefinieerd in `user/user.h` en in `kernel/syscall.h`. Ook hebben we `user/usys.pl` reeds aangepast. De implementatie van `chmod` moeten jullie uiteraard zelf nog geven. Vergeet ook niet om `kernel/syscall.c` correct te bewerken zodat de system call herkend wordt.

## Deel 1: modes toevoegen

Om permissions te ondersteunen moeten we allereerst voor ieder bestand of directory op de disk de mode bewaren.
Bestanden worden voorgesteld als een `struct dinode` op de disk, een `struct inode` in-memory.
Daarnaast wordt de `struct stat` gebruikt om meta-informatie te bewaren over bestanden.

1. Voeg een `mode` veld toe aan deze verschillende representaties van een bestand of bestandmetadata in `xv6`
2. Overloop de functies in `kernel/fs.c`. Zorg waar nodig dat deze functies gebruik maken van het nieuw toegevoegde `mode` veld.
3. Het bestand `mkfs/mkfs.c` wordt gebruikt door xv6 om het initiële file system aan te maken. Voeg `mode` toe als parameter aan de functie `ialloc` in `mkfs.c`. Pas vervolgens `mkfs.c` zo aan dat alle executable bestanden die gegenereerd worden de permissies `R/-/X` (read, execute) krijgen en alle andere bestanden de permissions `R/W/-` (read, write) krijgen. Directories krijgen alle permissions (`R/W/X`) toegekend.
4. Zorg ervoor dat het commando `ls` de permissies van bestanden toont als volgt:

```console
$ ls
.              1 1 1024 rwx
..             1 1 1024 rwx
README         2 2 2226 rw-
cat            2 3 30320 r-x
echo           2 4 29152 r-x
forktest       2 5 13792 r-x
grep           2 6 33680 r-x
```

## Deel 2: chmod

1. Voeg een system call `int chmod(const char*, int);` toe aan xv6. Deze functie neemt als eerste parameter het pad van een bestand en als tweede parameter de nieuwe bestandspermissies. De system call zorgt ervoor de permissies van het meegegeven bestand aangepast worden. 
2. Voeg een userspace-programma genaamd `chmod` toe dat als eerste argument een `mode` vraagt en als tweede argument het pad naar een bestand en vervolgens de pemissies van dit bestand aanpast. Bijvoorbeeld:

```console
$ chmod 7 README
$ ls
.              1 1 1024 rwx
..             1 1 1024 rwx
README         2 2 2226 rwx
```

## Deel 3: permissions controleren

Op dit punt worden permissies bewaard voor de verschillende bestanden. We kunnen deze bekijken met `ls` en aanpassen met `chmod`. 
Deze permissions worden echter nog niet gecontroleerd.
Dat is het laatste deel van deze permanente evaluatie.

1. Bewerk `exec` om ervoor te zorgen dat enkel bestanden die executable gemarkeerd zijn uitgevoerd mogen worden.
2. Bewerk de `open` syscall. De open syscall opent bestanden met een `omode`. Achterhaal hoe deze gebruikt wordt. Zorg ervoor dat deze syscall enkel slaagt indien de meegegeven `omode` toegestaan is door de file permissions.

## Deadline

De deadline van deze permanente evaluatie valt op 13 december om 23h59.

## Testen

We hebben een paar simpele testen gegeven die jullie kunnen gebruiken om te verifiëren dat er geen grote fouten gemaakt zijn.
Je kan deze uitvoeren via het volgende commando:

```console
[ubuntu-shell]$ make test
```

Let wel: we kijken jullie code ook nog handmatig na en het feit dat de testen slagen, wilt dus niet zeggen dat je een perfecte score zal halen!.

> :warning: Zorg steeds voor het indienen dat de testen zowel lokaal als in de GitHub Actions cloud werken.

> :bulb: De testen worden automatisch uitgevoerd op GitHub wanneer je nieuwe code pusht.
> Verifieer dat alles werkt door naar de "Actions" tab te gaan op de GitHub
> webinterface van je repository (of kijk naar het groene vinkje of rode
> kruisje dat naast je commit verschijnt).

## Indienen

Dit deel van de opgave moet ingediend worden en telt mee voor de permanente evaluatie van de oefeningen.
Dien je oplossing in met behulp van GitHub classroom.

* Commit en push de bestanden naar je repository

```console
[ubuntu-shell]$ git status # Aangepaste en nieuwe bestanden zijn aangegeven in het rood onder de heading "Changes not staged for commit" en "Untracked files"
[ubuntu-shell]$ git add . # Voeg alle aangepaste bestanden toe die nodig zijn
[ubuntu-shell]$ git status # Alle bestanden die je wil committen zouden nu aangegeven moeten zijn in het groen onder de heading "Changes to be committed"
[ubuntu-shell]$ git commit -m "<vul hier een commit message in>"
[ubuntu-shell]$ git push
```

> :bulb: Controleer op de webpagina van je repository of het bestand correct gecommit is en de GitHub Actions testen slagen.
