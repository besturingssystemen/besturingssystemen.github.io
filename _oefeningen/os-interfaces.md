---
layout: default
title: OS Interfaces
nav_order: 1
nav_exclude: false
---

In deze oefenzitting leren jullie het besturingssysteem xv6 kennen.

- [Voorbereiding](#voorbereiding)
- [GitHub classroom](#github-classroom)
  - [Persoonlijke repository](#persoonlijke-repository)
- [xv6 shell](#xv6-shell)
- [User space programma bekijken](#user-space-programma-bekijken)
  - [Libraries](#libraries)
  - [Programma afsluiten](#programma-afsluiten)
- [Eigen user space programma toevoegen](#eigen-user-space-programma-toevoegen)
- [Library functie toevoegen](#library-functie-toevoegen)
- [Zelfreflecterend proces](#zelfreflecterend-proces)
- [Communicerende processen](#communicerende-processen)
  - [Testen](#testen)
  - [Indienen](#indienen)

# Voorbereiding

Ter voorbereiding van deze oefenzitting wordt je verwacht:

* Een werkende Linux-omgeving te hebben. Volg stap 1 van [deze tutorial](https://github.com/informaticawerktuigen/klaarzetten-werkomgeving) voor meer uitleg.
* Een werkende versie van xv6-riscv te hebben. Volg [deze tutorial](https://github.com/besturingssystemen/klaarzetten-werkomgeving) voor meer uitleg.
* Hoofdstuk 1 van het [xv6 boek](https://github.com/besturingssystemen/xv6-riscv-book/releases/latest/download/book.pdf) gelezen te hebben.

# GitHub classroom

## Persoonlijke repository

De submissie van de permanente evaluatie zal gebeuren via GitHub classroom.
Jullie krijgen hiervoor een kopie van de xv6 repository.
Hierin kunnen jullie met `git` zelf wijzigingen toevoegen en committen.

* Klik op [deze link](https://classroom.github.com/a/dO_SIWTY) om een persoonlijke repository aan te maken.

Wanneer je een e-mail krijg van GitHub dat je repository klaar is, moet je deze clonen naar je eigen machine. Dit kan enkele minuten duren.

> :warning: Om bij GitHub te authenticeren via de command line maken we gebruik van ssh met public/private key authentication.
> Onderstaand clone-commando zal pas werken na het uitvoeren van het commando ```ssh-add <pad-naar-je-private-key>```.
> Het genereren en gebruik van deze ssh keys wordt uitgelegd in de laatste sectie van [deze tutorial](https://github.com/informaticawerktuigen/klaarzetten-werkomgeving).
> Je zal je key ook aan GitHub moeten toevoegen, dit staat [hier](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account) uitgelegd.

* Clone je persoonlijke repository, waarbij je `<GitHubUsername>` (inclusief `<>` haakjes) vervangt door je GitHub username:

```shell
[ubuntu-shell]$ git clone git@github.com:besturingssystemen-2021-2022/os-interfaces-<GitHubUsername>.git
```

* Verifieer dat je repository correct gecloned is door `make qemu` uit te voeren.

```shell
[ubuntu-shell]$ cd os-interfaces-<GitHubUsername>
[ubuntu-shell]$ make qemu
```

Indien `make qemu` ervoor zorgt dat xv6 opstart, is je repository correct gecloned.

# xv6 shell

Wanneer je xv6 start met ```make qemu``` beland je in een simpele shell-omgeving.

* Voer het commando ``ls`` uit in de xv6 shell

    ```shell
    [xv6-shell]$ ls
    ```

Het resultaat van het ls-commando toont de root directory van het file system van xv6. In de startdirectory staan alle user space programma's. Deze programma's kan je uitvoeren vanuit de xv6 shell.
Daarnaast staat er ook een README-bestand.

* Lees `README` met behulp van het `cat`-commando

    ```shell
    [xv6-shell]$ cat README
    ```

* Maak een folder aan genaamd `testfolder` en `cd` naar die folder.
  
  ```shell
  [xv6-shell]$ mkdir testfolder
  [xv6-shell]$ cd testfolder
  ```

* Verifieer nu met `ls` dat je in de lege folder zit
  
    ```shell
    [xv6-shell]$ ls
    ```

Je krijgt nu de melding `exec ls failed`. De xv6 shell is namelijk een zeer simpele shell.

Wanneer je in `shell` (de standaard shell in de meeste Linux-distributies) een commando uitvoert, zoekt `shell` in alle directories in een variabele genaamd `$PATH` naar dit programma.
De shell van xv6 (het programma `sh`) heeft geen `$PATH`-variabele en zoekt dus enkel in de huidige directory naar uitvoerbare programma's.
Je kan alsnog `ls` uitvoeren door een relatief of absoluut pad te specifiëren.

* Voer ls uit in `sh` met een relatief pad

    ```shell
    [xv6-shell]$ ../ls
    ```

* Voer ls uit in `sh` met een absoluut pad

    ```shell
    [xv6-shell]$ /ls
    ```

# User space programma bekijken

De broncode van de user space programma's die we net hebben uitgevoerd staat in de folder `user` in je Git-repository.

* Sluit de xv6-omgeving met <kbd>CTRL</kbd>+<kbd>A</kbd> <kbd>x</kbd>.
* Bekijk de code van het programma `cat`.

  ```shell
  [ubuntu-shell]$ gedit user/cat.c
  ```

## Libraries

De libraries die je kan terugvinden in de `#include`-statements zijn niet de standaard libraries die we kennen wanneer we C programmeren.

> :information_source: De [C Standard Library](https://en.wikipedia.org/wiki/C_standard_library) bestaat uit een verzameling nuttige functies die door C-programma's gebruikt kunnen worden. De implementatie van deze functies hangt echter af van het onderliggende besturingssysteem. Op Ubuntu en vele andere Unix-distributies heet deze implementatie `glibc` of [The GNU C Library](https://www.gnu.org/software/libc/). Een andere populaire implementatie heet [`musl`](https://musl.libc.org/).

xv6 biedt maar een zeer beperkte implementatie van libc aan. Daardoor kunnen we niet zomaar `#include <stdio.h>` gebruiken om bijvoorbeeld de functie `printf` op te roepen.
De libc-functies die xv6 aanbiedt kan je terugvinden in `user/user.h`.

* Open het bestand `user/user.h`

    ```shell
    [ubuntu-shell]$ gedit user/user.h
    ```

De functies in dit bestand kunnen gebruikt worden door alle user space programma's. In plaats van de standaard C header files te includen (zoals `stdio.h`) zullen we in xv6 telkens het bestand `user/user.h` includen.

De functies in `user/user.h` maken zelf gebruik van types gedefinieerd in `kernel/types.h`. Start dus standaard een user space programma voor xv6 met

```c
#include "kernel/types.h"
#include "user/user.h"
```

## Programma afsluiten

Merk op dat in xv6 een user space programma afgesloten wordt door de oproep ```exit(0);``` in plaats van te returnen uit main. Dit is het gevolg van het feit dat xv6 geen standaard C runtime implementeert. Een C runtime zoals [crt0](https://en.wikipedia.org/wiki/Crt0) is typisch verantwoordelijk voor het oproepen van de main-functie en na een return uit main het proces correct af te sluiten. Kijk [hier](https://stackoverflow.com/questions/3463551/what-is-the-difference-between-exit-and-return) voor meer informatie.

# Eigen user space programma toevoegen

We zijn klaar om een simpel user space programma toe te voegen aan xv6.

* Maak een bestand `helloworld.c` in de directory `user`

```shell
[ubuntu-shell]$ touch user/helloworld.c
```

* Schrijf in dit bestand een simpel C-programma dat de string *Hello, world!* print naar de terminal.

```shell
[ubuntu-shell]$ gedit user/helloworld.c &
```

Bij het compileren van het besturingssysteem met `make qemu` worden ook alle programma's in de directory `user` gecompileerd naar Risc-V. Dit wordt gespecifieerd door middel van de `Makefile` in `xv6-riscv`.

We voegen nu het helloworld-programma toe aan de Makefile.

* Open het bestand `Makefile`

```shell
[ubuntu-shell]$ gedit Makefile &
```

* Zoek naar de definitie van UPROGS en voeg het programma toe

```make
UPROGS=\
    $U/_cat\
    ...
    $U/_helloworld\
    ...
    $U/_zombie\
```

> :information_source: De `UPROGS` Makefile variabele bevat alle user-space programma's die gecompileerd moeten worden.
> De Makefile zorgt ervoor dat een entry van de vorm `$U/_prog` het bestand `user/prog.c` zal compileren en installeren in de root directory als een uitvoerbaar bestand genaamd `prog`.

* Compileer `xv6` en start via qemu

```shell
[ubuntu-shell]$ make qemu
```

* Voer het programma uit

```shell
[xv6-shell]$ helloworld
Hello, World!
```

# Library functie toevoegen

Als er enkel een string afgeprint moet worden zonder interpolatie van variabelen, gebruikt men meestal de functie `puts` uit libc.
De beperkte libc versie die xv6 aanbiedt, implementeert deze functie echter niet.
Je hebt in de vorige oefening dus waarschijnlijk de (minder efficiënte) `printf` functie gebruikt.
In deze oefening moeten jullie `puts` implementeren als een library functie en de oplossing van de vorige oefening aanpassen om deze functie gebruiken.

De declaratie van `puts` is als volgt (dit is een vereenvoudigde versie zonde return-type, de versie in libc geeft een `int` terug om fouten weer te geven):

```c
void puts(const char* str);
```

`puts` schrijft de null-terminated string `str` naar stdout gevolgd door een newline (`\n`).

Om ervoor te zorgen dat alle xv6 user-space programma's `puts` kunnen gebruiken, moet de declaratie eerst toegevoegd worden aan `user/user.h`.
Maak daarna een nieuw bestand aan (`user/puts.c`) waarin `puts` geïmplementeerd zal worden.
Om ervoor te zorgen dat dit bestand gecompileerd wordt, moet je de `ULIB` variabele in de `Makefile` uitbreiden met `$U/puts.o`.
De `ULIB` variabele bevat de object files die aan alle user-space programma's worden toegevoegd.
Dit zijn dus onder andere alle bestanden die library functies implementeren.

Implementeer nu de `puts` functie in `user/puts.c`.
Het is de bedoeling om enkel de `write` system call te gebruiken (gebruik dus zeker *niet* `printf` om `puts` te implementeren).
Andere library functies de geen system calls veroorzaken (zoals `strlen`) mogen wel gebruikt worden.

> :information_source: Met behulp van een ["system call"](https://en.wikipedia.org/wiki/System_call) kan een user programma een service van het onderliggende besturingssysteem aanvragen. System calls zijn gestandardiseerd in de Portable Operating System Interface (POSIX) standaard. Tijdens het programmeren van user programma's, kan je de verwachte argumenten en return waarden voor een system call opvragen via het Linux commando `man` (sectie 2), bijvoorbeeld [`man 2 write`](https://linux.die.net/man/2/write). Let wel, in tegenstelling tot de Linux kernel, is xv6 een educationeel klein besturingssysteem dat expliciet **niet** bedoeld is om de volledige POSIX standaard te implementeren. Gebruik de Linux `man` pages dus vooral als leidraad, maar verwacht niet dat xv6 alle system calls of randgevallen steeds zal implementeren!

# Zelfreflecterend proces

Voeg nu zelf een userspace programma toe genaamd `introspection.c`.
De bedoeling is dat dit programma zijn eigen memory layout uitprint.

We zijn geïnteresseerd in de locatie van:

* de stack
* de heap
* de .text sectie (de code van je programma)
* de .data sectie (de global variables van je programma)

Gebruik onderstaande `struct memlayout` om deze waarden te bewaren.

```c
struct memlayout {
    void* text;
    int* data;
    int* stack;
    int* heap;
};
```

Onderstaande methoden kan je gebruiken om de adressen van elke sectie te vinden:

* Zoek een adres op de stack door een lokale variabele (van type `int`) te declareren in een functie. Deze variabele zal altijd gealloceerd worden op de call stack van je proces. Het adres van deze variabele is dus een adres op de stack.
* Zoek een adres op de heap door een `int` te alloceren via `sbrk(sizeof(int))`. De return-waarde van deze system call geeft de oude waarde terug van de [`program break` (meer info)](https://en.wikipedia.org/wiki/Sbrk) wat het adres is van de nieuwe allocatie.
* Zoek een adres in de .text section door het adres van een functie op te slaan. Het adres van een functie kan je verkrijgen door de naam van de functie te typen (zonder `()`)
  
    ```c
    void* function_address = (void*)function_name;
    ```

* Zoek een adres in de .data section door een globale variabele (van type `int`) te declareren en het adres van deze variabele op te vragen.

Zorg ervoor dat de main-functie van je programma een `struct memlayout` aanmaakt. Sla in elk veld van de struct een correct adres op uit de bijhorende sectie van het programma.
Voeg daarna een functie toe die de waarden in `struct memlayout` afprint en vergelijk je resultaat met Figuur 3.4 in het xv6 boek.

> :exclamation: De bovenstaande werkwijzen geven telkens een adres terug uit een bepaalde sectie van het programma.
> Deze adressen zijn niet noodzakelijk de start- of eindadressen van deze secties.
> Ze vallen wel telkens in de range [`section start addr`, `section end addr`].
> Het is voor deze opgave **niet nodig** om het startadres te geven van een sectie. Een adres in de sectie-range is voldoende.

# Communicerende processen

We weten nu hoe we user space programma's kunnen toevoegen aan xv6.
Als laatste deel van deze oefenzitting en als *permanente evaluatie* is het de bedoeling dat je het programma `introspection.c` uitbreidt _in een nieuw bestand_ genaamd `evaluation.c`.
Maak dus eerst een kopie:

```shell
cp user/introspection.c user/evaluation.c
```

en pas de `Makefile` aan om dit nieuwe bestand te compileren.

Het doel van deze oefening is de memory layout van een parent process te vergelijken met dat van een child process.
Naast informatie over de memory layout van de processen, zijn we ook geïnteresseerd in de *waarden* op deze memory locations.
Bijkomend zal het child process zelf niets mogen afprinten maar zijn layout informatie delen met de parent via een pipe.

Je programma zal de volgende stappen moeten uitvoeren:

1. Initialiseer een `struct memlayout` op dezelfde manier als in de vorige oefening;
1. Zorg ervoor dat alle gealloceerde variabelen geïnitialiseerd zijn;
1. `fork` een nieuw process en zorg dat er een `pipe` gedeeld wordt tussen parent en child;
1. In het child process:
    1. Initialiseer een `struct memlayout` op dezelfde manier als in de vorige oefening.
       Je moet hiervoor de stack en data variabelen hergebruiken maar maak wel een nieuwe heap allocatie aan;
    1. Zorg ervoor dat alle gealloceerde variabelen geïnitialiseerd zijn (gebruik een *andere waarde* dan in het parent process);
    1. Kopieer de waarden in een `struct memvalues` (zie hieronder);
    1. Zend eerst de `struct memlayout` en dan de `struct memvalues` naar de parent via de pipe;
1. In het parent process:
    1. Ontvang `struct memlayout` en `struct memvalues` van het child process;
    1. Print deze structs via `print_mem` (zie hieronder);
    1. Initialiseer een `struct memvalues` met de waarden in het parent process;
    1. Print de structs van het parent process via `print_mem`.

```c
struct memvalues {
    int data;
    int stack;
    int heap;
};

// who should be either "parent" or "child"
void print_mem(const char* who, struct memlayout* layout, struct memvalues* values) {
    printf("%s:stack:%p:%d\n", who, layout->stack, values->stack);
    printf("%s:heap:%p:%d\n", who, layout->heap, values->heap);
    printf("%s:data:%p:%d\n", who, layout->data, values->data);
    printf("%s:text:%p\n", who, layout->text);
}
```

Denk voor je begint aan de implementatie na over wat de output van je programma zal zijn.
Probeer voor jezelf te voorspellen wat de relatie gaat zijn tussen adressen en waarden van variabelen in parent en child processen.

## Testen

We hebben een paar simpele testen gegeven die jullie kunnen gebruiken om te verifiëren dat er geen grote fouten gemaakt zijn.
Je kan deze uitvoeren via het volgende commando:

```shell
make test
```

Let wel: we kijken jullie code ook nog handmatig na en het feit dat de testen slagen, wilt niet zeggen dat je een perfecte score zult halen!.

## Indienen

Dit deel van de opgave moet ingediend worden en telt mee voor de permanente evaluatie van de oefeningen.

* Commit en push het bestand `evaluation.c` naar je repository

```shell
[ubuntu-shell]$ git add user/evaluation.c # Voeg ook andere aangepaste bestanden toe die nodig zijn
[ubuntu-shell]$ git commit -m "Added introspection program"
[ubuntu-shell]$ git push
```

> :bulb: Controleer op de webpagina van je repository of het bestand correct gecommit is.

> :bulb: De testen worden automatisch uitgevoerd op GitHub wanneer je nieuwe code pusht.
> Verifieer dat alles werkt door naar de "Actions" tab te gaan.
