---
layout: default
title: "Achtergrond: Exceptions"
nav_order: 3
parent: "Zitting 4: Traps"
---

# Achtergrond: Toepassingen van exceptions
{: .no_toc }

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

We kennen nu het mechanisme waarmee traps worden afgehandeld en hebben reeds onze eerste ervaringen met exceptions gehad.
Laten we nu eens kijken naar enkele toepassingen van exceptions.
Nadien bespreken we interrupts, die ook door traps afgehandeld worden.

## Error handling

De meest standaard toepassing waaraan gedacht wordt bij exceptions is het afhandelen van fouten.

Zo is er bijvoorbeeld een regel in RISC-V dat een `lw` (load word) instructie enkel mag opgeroepen worden met een adres dat een veelvoud is van 4. Een word is 4 bytes groot, als we ons geheugen opdelen in words is het byte-adres van ieder word namelijk een veelvoud van 4.
Wanneer we een `lw` instructie proberen uitvoeren met een adres dat geen veelvoud is van 4, wordt de exception *load address misaligned* gesmeten.

Zoals we weten, wordt vervolgens de controle doorgegeven aan de trap handler in één van de verschillende privilegeniveau's, afhankelijk van de waarde van de delegatieregisters.
Laten we even aannemen dat de exception in de kernel wordt afgehandeld, in supervisor mode.

Wat kan een kernel doen om dit probleem op te lossen?
De handler zou kunnen beslissen om de `lw` instructie over te slaan, maar dan zou het uitvoerende proces incorrect werken.
In de meeste gevallen is het antwoord dus simpelweg, het proces vroegtijdig beeïndigen en een boodschap geven dat het proces een exception heeft veroorzaakt met daarbij informatie over de specifieke exception.

De boodschap *segmentation fault* die je krijgt in vele Linux-distributies is een voorbeeld van een exception die niet opgelost kan worden door het besturingssysteem als gevolg van een fout in het programma.

In xv6 kennen jullie ondertussen reeds de foutboodschap `usertrap` en `kerneltrap`, die getoond wordt wanneer xv6 een exception niet kan oplossen.

<!-- 
* **Oefening:** **TODO** In jullie xv6 repository hebben we enkele simpele programma's toegevoegd, die elk exceptions veroorzaken.
Voer deze programma's uit en gebruik de tabel [scause](img/scause.png) om te bepalen wat er misloopt.
Gebruik het commando `risc64-linux-gnu-objdump -d <executable>` om te achterhalen naar welke regel code de programmateller verwees op het moment dat de exception zich voordeed. Probeer ten slotte het probleem op te lossen door het `.S` bronbestand te wijzigen.
-->

## System calls

Misschien is het je opgevallen dat environment calls (system calls) ook in de scause tabel stonden.
Zoals eerder reeds vermeld in de oefenzitting over system calls, worden deze geïmplementeerd met behulp van exceptions.

De `ecall` instructie genereert expliciet een exception.
De code van de exception verschilt afhankelijk van het privilegeniveau waarin de `ecall` uitgevoerd wordt.
Zo kan je ook vanuit de kernel een `ecall` uitvoeren, die dan in machine mode opgevangen zou kunnen worden.

* Bekijk de code in [`usertrap()`][usertrap]. Herinner je dat deze functie opgeroepen wordt door de trampoline. Je zou ondertussen de code in usertrap moeten begrijpen. Merk op dat deze code ofwel een system call oproept, ofwel een interrupt afhandelt (hierover meer in volgende sectie) ofwel de error message print die we in vorige oefening hebben gedecodeerd.

In het geval van een system call wordt dus [`syscall()`][syscall] opgeroepen, een methode die jullie ondertussen goed zouden moeten kennen.

## Demand paging

Exceptions worden dus niet enkel gebruikt om fouten op te lossen, ook voor system calls.
Een andere interessante toepassing van exceptions is de implementatie van demand paging.
Demand paging in het algemeen zorgt ervoor dat een pagina van een proces pas gealloceerd en gemapt worden de eerste keer dat deze pagina wordt aangesproken.

Neem het voorbeeld van een ELF-bestand.
Stel dat we geen enkel segment van het ELF-bestand in het geheugen laden. We starten met een page table waarin enkel de trampolinepagina en het trapframe gemapt staat en springen naar het entry point van het ELF bestand (het virtueel adres waar de eerste instructie van het programma zich bevindt).

Op dat moment wordt een page fault exception gegenereerd.
Het entry point zal namelijk niet gemapt zijn.
De trap handler vangt deze page fault exception op en kan op dat moment de kernel vragen om enkel deze specifieke pagina te mappen.
Vervolgens kan de code op dat adres wel ingeladen worden en verder uitgevoerd worden.
Pagina's worden dus pas gemapt op het moment dat ze worden aangesproken.
Om demand paging te implementeren maken we dus gebruik van exceptions.

## Debugging

Debuggers worden ook mogelijk gemaakt dankzij exceptions.
Net zoals de `ecall`-instructie een exception genereert en de controle doorgeeft aan de trap handler, zo zal ook de `ebreak`-instructie een exception genereren, met een specifieke exception code.

Wanneer je met behulp van een debugger een breakpoint plaatst in een programma, wordt de instructie waarop je een breakpoint plaatst vervangen door de `ebreak`-instructie.
Op het moment dat de instructie normaal gezien zou uitvoeren, zal je dus in plaats daarvan een exception genereren.
Deze kan dan opgevangen worden door een trap handler, die vervolgens de controle door kan geven aan de debugomgeving.
De executie van het programma is op dat moment gepauzeerd, de debugomgeving kan functionaliteit aanbieden die het mogelijk maakt de toestand van het geheugen en de registers te inspecteren.

<!--
In komende oefening zullen we een simpele debugger implementeren voor xv6, gebruik makend van exceptions.

* **TODO**
* **Oefening** (misschien?): basic debugger schrijven voor xv6:
  * `./dump_registers <symbol> <executable> <args>`
  * Instructie op locatie `<symbol>` vervangen door `ebreak`
  * User-mode handler in dump_registers.S
  * In geval van ebreak exception
  * Dump alle registers naar stdout
  * Vervang `ebreak` terug door originele instructie
  * continue

-->

[usertrap]: https://github.com/besturingssystemen/xv6-riscv/blob/27057bc9b467db64a3de600f27d6fa3239a04c88/kernel/trap.c#L32
[syscall]: https://github.com/besturingssystemen/xv6-riscv/blob/2b5934300a404514ee8bb2f91731cd7ec17ea61c/kernel/syscall.c#L133
