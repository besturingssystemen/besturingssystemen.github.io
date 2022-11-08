---
layout: default
title: "Permanente evaluatie"
nav_order: 6
parent: "Zitting 4: Traps"
---

# Permanente Evaluatie
{: .no_toc }

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

## Lazy allocation

Tot nu toe hebben we in voorbeeldcode altijd `sbrk` gebruikt om geheugen op de heap te alloceren.
C-programma's gebruiken typisch echter de functies [`malloc`][malloc ref] en [`free`][free ref] om heap geheugen te alloceren en te dealloceren.
Deze functies zijn in user space geïmplementeerd en gebruiken intern syscalls zoals `sbrk` om geheugen te krijgen van de kernel.
Om het aantal syscalls laag te houden, zullen deze functie typisch meer geheugen van de kernel vragen dan ze nodig hebben.

1. Schrijf een programma die 1 byte op de heap alloceert via `malloc(1)` en ga na hoeveel geheugen er via `sbrk` gevraagd wordt.
   Je kan dit bijvoorbeeld doen door in GDB een breakpoint te zetten in de [`sys_sbrk`][sys_sbrk] functie of door daar een print toe te voegen.

   Zoals je gezien zou moeten hebben, alloceert de [`malloc` implementatie van xv6][umalloc.c] een stuk [meer geheugen][over alloc] dan er gevraagd wordt.
   Alhoewel dit inderdaad het aantal syscalls zal verlagen, kan het er ook voor zorgen dat er veel meer geheugen gebruikt wordt dan nodig.

   In deze oefening gaan we [_demand paging_][demand paging] implementeren voor het heap geheugen.
   Het idee is het volgende: wanneer er via `sbrk` extra geheugen voor het proces gevraagd wordt, wordt dit geheugen niet onmiddellijk in het proces gemapt.
   Pas wanneer het process dit geheugen probeert te gebruiken, en er dus een page fault gebeurt, zal de kernel een frame alloceren en mappen op het adres dat het process probeerde te gebruiken.

2. Voeg een functie `void pagefault(uint64 va)` toe aan [`vm.c`][vm.c].
   Het `va` argument zal aangeven welk virtueel adres werd aangesproken toen de page fault gebeurde.
   Print hier voorlopig een boodschap af en stop het huidige process door de [`struct proc::killed`][proc killed] variabele op `1` te zetten voor het huidige proces.

   Ons volgende doel is om ervoor te zorgen dat, wanneer er een load of store page fault optreedt, onze handler wordt opgeroepen.
   Dit vervangt het normale gedrag waarbij deze exceptions er gewoon voor zorgt dat een error geprint wordt. Op dit moment print je eigen handler uiteraard ook enkel nog maar een boodschap, maar dat zullen we aanpassen.

   > :bulb: Instruction page faults vangen we niet op. De heap is namelijk niet executable. Als een user space programma springt naar een gedeelte lazy heap willen we dit niet mappen, enkel indien er gelezen of geschreven wordt.

3. Zorg ervoor dat `void pagefault(uint64 va)` opgeroepen wordt wanneer er een page fault in user mode voorkomt.
   Bekijk hiervoor de [`usertrap`][usertrap] functie.
   Het virtuele adres dat de page fault veroorzaakte, vind je in het `stval` CSR ([hint][stval hint]).
   Je moet er ook voor zorgen dat interrupts terug enabled worden vóór de uitvoering van je handler, kijk hoe dit gebeurt voor system calls.

   > :warning: Zorg ervoor dat eventuele waarden die je nodig hebt uit CSRs al in een variabele zitten alvorens je interrupts terug enabled, anders kunnen deze waarden verloren gaan (zie comments bij het enablen van interrupts in het system call gedeelte).

   > :information_source: Het system call gedeelte verhoogt `epc` in het trapframe (de waarde van de program counter op het moment van de exception) met 4. Dit zorgt ervoor dat wanneer de trap handler returnt naar user space, we niet terugspringen naar de `ecall` instructie maar wel naar de instructie erna. In het geval van de page fault handler willen we echter de instructie die de page fault veroorzaakt heeft opnieuw laten uitvoeren (na mapping van de page), dus willen we deze instructie zeker niet overslaan.

   Als het goed is, heb je nu een werkende page fault handler die een boodschap print en het proces killt.
   Dit is een goed moment om eens te controleren of de kernel nog naar behoren werkt.
   Je kan hiervoor bijvoorbeeld de `usertests` executable gebruiken.
   Dit is een user space programma dat standaard met xv6 wordt geleverd en een hele hoop testen uitvoert.
   Je kan dit uitvoeren zoals elk ander user space programma.

   > :warning: De `usertests` voert een test (bigwrite) uit die zeer veel data schrijft naar het file system. Indien je zelf enkele programma's aan xv6 toevoegt, faalt deze test omdat het file system daar vol komt te zitten. Los dit op door in `kernel/params.h` de constante `FSSIZE` op een grotere waarde (bvb 2000) te zetten.

   De volgende stap is om `sbrk` _lazy_ te laten werken.
   Zoals eerder beschreven, wilt dit zeggen dat geheugen niet direct in het proces gemapt wordt tijdens een oproep van `sbrk`.

4. Zoek uit hoe `sbrk` precies werkt, begin hiervoor met het lezen van de [`sys_sbrk`][sys_sbrk] functie.
   Als er een positief getal aan `sbrk` wordt gegeven, zal het geheugen van het proces vergroot worden.
   In plaats van dit direct te doen, moet je ervoor zorgen dat de aanvraag enkel geregistreerd wordt zonder geheugen te mappen.
   De [`struct proc::sz`][proc sz] variabele geeft aan hoeveel geheugen een proces gebruikt.
   Een negatief argument voor `sbrk` zorgt ervoor dat het geheugen _verkleind_ wordt; dit moet _wel_ blijven gebeuren (waarom?).

   Na deze stap zullen user space processen die `sbrk` gebruiken uiteraard niet meer juist werken.
   Wat verwacht je dat er gebeurt met zulke processen?
   Verifieer dit ook.

5. Implementeer nu de logica in de page fault handler om pages _on demand_ te mappen.
   De functie [`kalloc`][kalloc] kan je gebruiken om nieuwe fysieke frames to alloceren en in de vorige oefenzitting hebben jullie al geleerd hoe je mappings kan maken via [`mappages`][mappages].
   Er zijn een aantal zaken waar je op moet letten:
    - Voeg enkel mappings toe voor adressen die eerder via `sbrk` gealloceerd waren (denk aan de [`struct proc::sz`][proc sz] variabele);
    - Controleer of er niet al een mapping bestaat voor het adres dat de page fault genereerde (wat wilt dit zeggen?);
    - Als de page fault handler faalt _na_ het alloceren van een frame, moet dit frame weer vrijgegeven worden via [`kfree`][kfree] (waarom?).

   Nu is het grootste deel van de benodigde logica voor een lazy `sbrk` geïmplementeerd.
   Controleer door bijvoorbeeld de `vmprintmappings` syscall te gebruiken dat mappings inderdaad on demand aangemaakt worden.

   Je zal echter merken dat programma's die `sbrk` gebruiken meestal een [`panic`][panic] veroorzaken, bijvoorbeeld wanneer ze afgesloten worden.
   Er zijn een aantal functies in xv6 die afdwingen dat alle pages tussen `[0, struct proc::sz)` gemapt zijn.
   Dit is begrijpelijk aangezien het (zonder demand paging) een bug zou zijn als dit niet het geval is.
   Nu we demand paging hebben toegevoegd, klopt deze invariant echter niet meer.

6. Vind de functies de een `panic` veroorzaken.
   Je kan dit bijvoorbeeld doen door [GDB][gdb] te gebruiken en een breakpoint te zetten in de [`panic`][panic] functie.
   Als je dan een backtrace afprint (via het `backtrace` commando), kan je zien waar `panic` opgeroepen werd.
   Los de `panic`s op door op de juiste plekken unmapped pages te negeren.

   Als het goed is, zullen de meeste user space programma's nu weer uit kunnen voeren zonder problemen.
   Verifieer dit en controleer je implementatie via de `vmprintmappings` syscall.

## Bonusoefening

> :warning: When you start this bonus exercise, open the file `exercises.json` and set the variable `bonus_exercise` to `true`. This will make sure that the
> automatic tests now also test the bonus exercise (and we have a clear marker that you want the bonus exercise to be graded!)
> After setting `bonus_exercise` to true, make sure to check that your solution passes the additional automatic tests on Github Actions!
> In the case were you started the bonus exercise, but could not finish it to the point where it passes the additional tests, you can unset `bonus_exercise` and set `want_bonus_graded` in `exercises.json`, so that we see that you started on the bonus exercise but the test ignores it.
> We will then still do a manual check of your partial bonus solution and take it into account for grading.
> You can also let us know if you maybe put the bonus in another branch than the default `main` branch by means of the `optional_bonus_branch` variable.

Als je het `usertests` programma runt, zul je merken dat er toch nog een paar problemen zijn met onze lazy heap allocatie.

1. Schrijf een user space programma dat de `read` en `write` syscalls gebruikt met buffers in nog niet gemapte pages.
   Roep dus eerst `sbrk` op en gebruik een pointer naar dit nieuwe geheugen als buffer _zonder_ dit geheugen eerst te gebruiken (want dan wordt het gemapt).
   Wat is het resultaat en hoe verklaar je dit?

   Wanneer syscalls zoals `read` en `write` een pointer krijgen naar user space geheugen, kunnen ze deze niet zomaar gebruiken.
   Herinner je uit de vorige oefenzitting dat de kernel code runt in de kernel address space en het user space geheugen dus niet gemapt is.
   Kernel code gebruikt daarom de volgende functies om aan user space geheugen te kunnen:
   - [`copyout`][copyout]: Kopieert data in de kernel address space naar een user address space;
   - [`copyin`][copyin]: Kopieert data in een user address space naar de kernel address space;
   - [`copyinstr`][copyinstr]: Hetzelfde als `copyin` maar specifiek om strings te kopiëren.

   Al deze functies werken op dezelfde manier: gegeven een page table van een user proces en een virtueel adres in dit proces, gebruik eerst de [`walkaddr`][walkaddr] functie om dit adres om te zetten naar een fysiek adres.
   Aangezien dit fysieke adres wel in de kernel gemapt is, kan de data gekopieerd worden naar/van dit adres.
   Als `walkaddr` faalt omdat het adres niet gemapt is, zullen de verschillende copy functies een error teruggeven en faalt de syscall.
   Er zal dus geen page fault gebeuren maar het user space programma krijgt een error code terug van de syscall.

2. Pas `copyin`, `copyinstr` en `copyout` aan zodat on demand pages gemapt worden wanneer nodig.

   Als dit gebeurd is, zou het `usertests` programma zonder fouten moeten runnen.


[usertrap]: https://github.com/besturingssystemen/xv6-riscv/blob/103d9df6ce3154febadcf9a67791d526ec6b07ac/kernel/trap.c#L37
[malloc ref]: https://en.cppreference.com/w/c/memory/malloc
[free ref]: https://en.cppreference.com/w/c/memory/free
[sys_sbrk]: https://github.com/besturingssystemen/xv6-riscv/blob/103d9df6ce3154febadcf9a67791d526ec6b07ac/kernel/sysproc.c#L41
[umalloc.c]: https://github.com/besturingssystemen/xv6-riscv/blob/103d9df6ce3154febadcf9a67791d526ec6b07ac/user/umalloc.c
[over alloc]: https://github.com/besturingssystemen/xv6-riscv/blob/103d9df6ce3154febadcf9a67791d526ec6b07ac/user/umalloc.c#L54
[demand paging]: https://en.wikipedia.org/wiki/Demand_paging
[vm.c]: https://github.com/besturingssystemen/xv6-riscv/blob/103d9df6ce3154febadcf9a67791d526ec6b07ac/kernel/vm.c
[proc killed]: https://github.com/besturingssystemen/xv6-riscv/blob/103d9df6ce3154febadcf9a67791d526ec6b07ac/kernel/proc.h#L101
[proc sz]: https://github.com/besturingssystemen/xv6-riscv/blob/103d9df6ce3154febadcf9a67791d526ec6b07ac/kernel/proc.h#L107
[stval hint]: https://github.com/besturingssystemen/xv6-riscv/blob/103d9df6ce3154febadcf9a67791d526ec6b07ac/kernel/trap.c#L71-L72
[kalloc]: https://github.com/besturingssystemen/xv6-riscv/blob/103d9df6ce3154febadcf9a67791d526ec6b07ac/kernel/kalloc.c#L65
[kfree]: https://github.com/besturingssystemen/xv6-riscv/blob/103d9df6ce3154febadcf9a67791d526ec6b07ac/kernel/kalloc.c#L42
[mappages]: https://github.com/besturingssystemen/xv6-riscv/blob/103d9df6ce3154febadcf9a67791d526ec6b07ac/kernel/vm.c#L133
[panic]: https://github.com/besturingssystemen/xv6-riscv/blob/103d9df6ce3154febadcf9a67791d526ec6b07ac/kernel/printf.c#L120
[uvmunmap]: https://github.com/besturingssystemen/xv6-riscv/blob/103d9df6ce3154febadcf9a67791d526ec6b07ac/kernel/vm.c#L159
[gdb]: https://github.com/besturingssystemen/klaarzetten-werkomgeving#gdb
[copyout]: https://github.com/besturingssystemen/xv6-riscv/blob/103d9df6ce3154febadcf9a67791d526ec6b07ac/kernel/vm.c#L340
[copyin]: https://github.com/besturingssystemen/xv6-riscv/blob/103d9df6ce3154febadcf9a67791d526ec6b07ac/kernel/vm.c#L365
[copyinstr]: https://github.com/besturingssystemen/xv6-riscv/blob/103d9df6ce3154febadcf9a67791d526ec6b07ac/kernel/vm.c#L390
[walkaddr]: https://github.com/besturingssystemen/xv6-riscv/blob/103d9df6ce3154febadcf9a67791d526ec6b07ac/kernel/vm.c#L100

