---
layout: default
title: "Aanmaak processen"
nav_order: 1
parent: "Zitting 2: System calls"
---

# Aanmaak processen

We starten onze bespreking van system calls door twee zeer belangrijke system calls te bespreken: `fork` en `exec`.
Hiervoor is het belangrijk om te begrijpen wat we bedoelen wanneer we spreken over een proces.

## De proces-abstractie

Een proces is een abstractie op niveau van het besturingssysteem.
Besturingssystemen zijn verantwoordelijk voor de aanmaak, het beheer en de correcte afsluiting van processen.

De proces-abstractie is enorm krachtig.
Ze laat ons toe om complexe systemen op te bouwen als een verzameling programma's die elk een eigen specifieke rol vervullen.
Deze programma's kunnen in parallel worden uitgevoerd, *geïsoleerd* van elkaar met elk een eigen geheugenruimte.

Om te vermijden dat processen in user space de correcte werking van het besturingssysteem in gedrang kunnen brengen, zal de kernel (de core van het besturingssysteem) ook zichzelf isoleren van deze processen.

Isolatie via processen zorgt er dus enerzijds voor dat programma's geen toegang hebben tot het geheugen van andere programma's, maar ook dat deze programma's geen toegang hebben tot het geheugen van de kernel.
Een proces kan niet lezen of schrijven naar dit kernelgeheugen.
Het is zelfs niet mogelijk om met een jump of call instructie naar kernelcode te springen.

Om processen toch de mogelijkheid aan te bieden om diensten van het besturingssysteem te gebruiken, wordt gebruik gemaakt van *system calls*.
System calls vormen de interface van de kernel naar processen.
Ze laten het toe om de code in de kernel op een gecontroleerde manier uit te voeren.

## De `fork` system call

In [UNIX](https://en.wikipedia.org/wiki/Unix) (Linux, xv6, ...) besturingssystemen worden nieuwe processen aangemaakt door middel van de `fork` system call.
Deze system call maakt een kopie van het huidige proces.

### Process state

In detail begrijpen hoe een proces gekopieerd wordt, is op dit punt in de oefenzittingen nog te vroeg. We kunnen wel al tonen welke state een besturingssysteem bewaart per proces.

* Bekijk [struct proc][struct proc] gedefinieerd in `proc.h` in xv6.
* Lees de comments bij elk veld van de `struct`.
  
Het nut van de velden `parent`, `pid`, `sz`, `ofile`, `cwd` en `name` zou duidelijk moeten zijn. Vraag verduidelijking aan een assistent indien dit niet het geval is.

> :bulb: Een file descriptor (`int fd`) indexeert de `ofile` array. Wanneer we dus met de `write` system call schrijven naar een bestand geven we als eerste argument aan `write` een index mee in de tabel met open bestanden van het proces.

Om het veld `pagetable` te begrijpen hebben we kennis nodig van virtual memory, een concept dat we in de volgende oefenzitting zullen bekijken.

De velden `state` en `killed` bevatten informatie voor de scheduler en zullen dus in detail worden bekeken in een toekomstige oefenzitting over scheduling.

De velden `lock`, `chan` en `xstate` hebben te maken met synchronizatie en worden in een latere oefenzitting over synchronizatie in detail bekeken.

### Trapframe

Begrip van het veld `trapframe` is belangrijk voor deze oefenzitting. Wanneer een proces de controle doorgeeft aan het besturingssysteem bij het uitvoeren van een *system call* of wanneer een proces onderbroken wordt door bijvoorbeeld een interrupt, gebeurt dit via een *trap*.

Een *trap* in user-mode zorgt ervoor dat de processor schakelt naar supervisor-mode en vervolgens de *trap handler* begint uit te voeren. Deze handler is een stuk machinecode op een vaste locatie in het geheugen.

Het uitvoerende proces, dat de trap heeft veroorzaakt, wil na afloop van de trap verder kunnen uitvoeren. Het kan echter zijn dat machinecode in het besturingssysteem bepaalde registers nodig heeft die reeds in gebruik zijn door het uitvoerende proces. Om ervoor te zorgen dat deze registerwaarden niet verloren gaan, bewaart de trap handler deze in de *trap frame*. Bij terugkeer uit de trap kunnen de registerwaarden hersteld worden.

* Bekijk de velden van [`struct trapframe`][trapframe] in kernel/proc.h.

Het veld `kernel_satp` is gerelateerd aan het veld `pagetable` in `struct proc` en zal pas in detail bekeken worden in de oefenzitting over Virtual Memory.

In xv6 heeft de kernel een eigen call stack, gesplitst van de call stack van het proces. Het veld `kstack` in `struct proc` bevat het adres van deze stack, het veld `kernel_sp` bevat een pointer naar de top van deze stack.
Merk op dat xv6 dus een kernel stack heeft _per proces_.
Waarom dit nodig is, zal duidelijk worden in latere oefenzittingen.

Het veld `kernel_hartid` bewaart de identifier van de hardware thread (CPU core) waarop het proces actief is. Dit wordt later besproken in de sessie over synchronizatie.

Het veld `ra`, en alle daarop volgende velden, bevatten de waarden van de registers op het moment dat de trap gebeurde.

Op dit moment zou je een idee moeten hebben van de state die per proces bewaard wordt, en de state die bewaard en hersteld moet worden bij het uitvoeren van een trap.

### Executable Files

De `fork` system call zal als een van zijn taken de process state kopiëren voor het nieuwe proces. Nadat een proces gekopieerd is, is het in vele gevallen de bedoeling dat het nieuwe proces een eigen taak toegewezen krijgt.
De meest gangbare manier om het proces een nieuwe taak toe te wijzen, maakt gebruik van de system call `exec`.

`exec` neemt als invoer een uitvoerbaar bestand (programma) en vervangt de code en data in het huidige proces door deze in het uitvoerbare bestand. Vervolgens wordt gesprongen naar het *entry point* van het programma. Op deze manier krijgt het geforkte proces een nieuwe taak toegewezen.

Om samen te vatten wordt een proces in UNIX aangemaakt door
een combinatie van de system call `fork`, dat de processtructuur kopieert, en `exec`, dat een nieuw programma
in het programma laadt.

[struct proc]: https://github.com/besturingssystemen/xv6-riscv/blob/2b5934300a404514ee8bb2f91731cd7ec17ea61c/kernel/proc.h#L94
[trapframe]: https://github.com/besturingssystemen/xv6-riscv/blob/2b5934300a404514ee8bb2f91731cd7ec17ea61c/kernel/proc.h#L52