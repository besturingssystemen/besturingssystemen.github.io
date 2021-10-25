---
layout: default
title: Permanente evaluatie
nav_order: 10
parent: "Zitting 3: Virtual memory"
---

# Permanente evaluatie

Bekijk de implementatie van de [`uptime`][sys_uptime] syscall.
Als we de locking even buiten beschouwing laten (hier komen we in een later sessie op terug), doet de `sys_uptime` functie niets anders dan de globale variabele [`ticks`][ticks] terug te geven.
Een volledige syscall uitvoeren (met twee context switches tot gevolg) lijkt bijzonder inefficiënt voor het lezen van een variabele.
Voor deze oefening zullen we hier een oplossing voor zoeken die slim gebruik maakt van memory mappings.

De Linux kernel gebruikt een mechanisme genaamd [vDSO][vDSO] om bepaalde syscalls volledig in user space uit te kunnen voeren.
De kernel groepeert een aantal veel gebruikte variabelen in een page en mapt deze page read-only in elk proces.
Dit zorgt ervoor dat processen de waarden van deze variabelen kunnen lezen (maar niet schrijven!) zonder een syscall uit te voeren.

We gaan nu een gelijkaardig mechanisme implementeren in xv6 om de `ticks` variabele leesbaar te maken in user space.
Aangezien mappings per pagina gemaakt worden, moeten we ervoor zorgen dat de `ticks` variabele alleen op een pagina staat (zodat we niet _meer dan nodig_ leesbaar maken in user space).
We doen dit in twee stappen:
1. We zorgen ervoor dat `ticks` in een specifieke linker sectie genaam `.vdso` geplaatst wordt.
   GCC biedt hiervoor het [`section`][GCC section] attribuut aan dat op de volgende manier gebruikt kan worden:
   ```c
   uint ticks __attribute__((section(".vdso")));
   ```
   Er zal dus een linker sectie `.vdso` gegenereerd worden waar enkel de `ticks` variabele in zit.
1. We passen het linkerscript ([`kernel/kernel.ld`][kernel.ld]) aan dat gebruikt wordt om de kernel te linken.
   Een linkerscript is een beschrijving van hoe de executable die de linker gaat maken er uit moet zien.
   Voeg de volgende code toe net _voor_ de lijn met `PROVIDE(end = .);`:
    ```c
    .vdso : { /* Begin een *output* sectie genaamd .vdso */
      . = ALIGN(0x1000); /* Align het huidige adres (.) op een page boundary */
      _vdso_start = .; /* Maak een nieuw symbol _vdso_start met het huidige adres */
      *(.vdso); /* Plaats alle *input* secties genaamd .vdso in de huidige output sectie  */
      . = ALIGN(0x1000); /* Align het huidige adres op een page boundary */
      ASSERT(. - _vdso_start == 0x1000, "error: vdso larger than one page");
    }
    ```
    Jullie hoeven hier zeker niet alles van te begrijpen!
    (De geïnteresseerden kunnen voor meer informatie de [manual][ld scripts] over linkerscripts lezen.)
    Het resultaat is dat de kernel executable een output sectie heeft die:
    1. Begint en eindigt op een page boundary (en exact 1 page groot is);
    1. Alle secties genaamd `.vdso` bevat (in ons geval dus enkel de `ticks` variabele);
    1. Een symbool genaamd `_vdso_start` definieert dat verwijst naar het begin van de sectie.

Dit is een goed moment om eens te controleren of de kernel nog werkt.
Om zeker te zijn dat de kernel volledig opnieuw gecompileerd en gelinkt wordt, voer je best eerst `make clean` uit.

Jullie opdracht bestaat nu uit het volgende:
1. Map de page die de `.vdso` sectie bevat in elk user proces op dezelfde locatie.
    1. Het adres van het symbool `_vdso_start` kan je in C krijgen door een variabele op de volgende manier te declareren:
       ```c
       extern char _vdso_start[];
       ```
    1. Een mogelijke plek om dit te implementeren, is in de functie [`proc_pagetable`][proc_pagetable] die de initiële mappings voor een proces aanmaakt.
       Kijk zeker naar hoe de mapping voor de trampoline aangemaakt wordt.
       Vergeet ook niet te kijken hoe de trampolinepagina op het einde van het proces unmapped wordt.
    1. Kies zelf een logisch virtueel adres voor de mapping.
    1. Zorg ervoor dat de page read-only gemapt wordt.
1. Maak nu een functie in user space genaamd `fastuptime` die de waarde van de variabele `ticks` uitleest uit de gemapte page en returnt.
    1. Hint: de variabele `ticks` staat op het eerste adres van deze page en heeft het type `uint`.

# Bonus: Null pointer exception

* Compileer en voer in je Linuxdistributie (niet in xv6) het volgende simpele programma uit:

```c
#include <stdio.h> //#include "user/user.h" in xv6! 
int main(){
  int *p = 0; //p is a pointer to the address 0
  printf("The value at address 0 is %d", *p);
  return 0;
}
```

* Wat is de output van je programma? *(hint: een welbekende foutmelding)*
* Compileer nu hetzelfde programma in xv6 en voer dit uit. Wat is daar je output?

De fout die we krijgen in onze eigen Linuxdistributie krijgen we niet in xv6.
Tijd om eens na te denken over wat bovenstaande code net doet.
In feite niet veel meer dan het adres 0 uitlezen.

In de meeste Linux-distributies wordt het adres 0 gebruikt om aan te geven dat een pointer nergens naar verwijst, of niet geïnitialiseerd is.
Denk bijvoorbeeld aan een simpele linked list.
Indien het *next* veld van een lijstelement 0 is, hebben we het laatste element van de lijst bereikt.
Kortom, het is niet de bedoeling dat er nuttige informatie op adres 0 te vinden is.

Het lezen van informatie uit adres 0 wijst dus bijna zeker op een fout in je code.
Om ervoor te zorgen dat deze fouten gedetecteerd kunnen worden, zullen Linux-distributies ervoor zorgen dat het adres 0 ontoegankelijk is.

xv6 doet dit echter niet. In xv6 staat er code op adres 0. Adres 0 is gewoon toegankelijk. Dat is in feite een zeer slechte beslissing, want zo is het veel lastiger fouten te vinden in C-code (in plaats van een exception krijg je willekeurge data, waarschijnlijk de bytecode van een functie, bij het uitlezen van adres 0).

* **Bonusoefening (moeilijk):** Pas xv6 aan zodat je een exception krijgt wanneer het adres 0 wordt gedereferenced.
  Let op, adres 0 mag ook niet uitvoerbaar zijn (om problemen met null functionpointers te detecteren).
  Met andere woorden, virtueel adres 0 mag helemaal niet gemapt zijn.

  Hint: bekijk de [lus die ELF secties inlaadt][exec load loop] in de [`exec`][exec] functie.

> :information_source: Gebruik het forum voor hulp!


[sys_uptime]: https://github.com/besturingssystemen/xv6-riscv/blob/720a130ceafcc55ec3624b47e8a1368f3f5f00ae/kernel/sysproc.c#L86
[ticks]: https://github.com/besturingssystemen/xv6-riscv/blob/720a130ceafcc55ec3624b47e8a1368f3f5f00ae/kernel/trap.c#L10
[vDSO]: https://en.wikipedia.org/wiki/VDSO
[GCC section]: https://gcc.gnu.org/onlinedocs/gcc/Common-Variable-Attributes.html#index-section-variable-attribute
[kernel.ld]: https://github.com/besturingssystemen/xv6-riscv/blob/720a130ceafcc55ec3624b47e8a1368f3f5f00ae/kernel/kernel.ld
[ld scripts]: https://sourceware.org/binutils/docs/ld/Scripts.html
[proc_pagetable]: https://github.com/besturingssystemen/xv6-riscv/blob/720a130ceafcc55ec3624b47e8a1368f3f5f00ae/kernel/proc.c#L162
[exec]: https://github.com/besturingssystemen/xv6-riscv/blob/720a130ceafcc55ec3624b47e8a1368f3f5f00ae/kernel/exec.c#L12
[exec load loop]: https://github.com/besturingssystemen/xv6-riscv/blob/720a130ceafcc55ec3624b47e8a1368f3f5f00ae/kernel/exec.c#L41-L59
