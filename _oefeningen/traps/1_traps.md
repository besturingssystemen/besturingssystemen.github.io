---
layout: default
title: "Achtergrond: Traps"
nav_order: 1
parent: "Zitting 4: Traps"
---

# Achtergrond: Traps
{: .no_toc }

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

De uitvoering van machinecode door een processor is meestal erg voorspelbaar:

1. Een instructie wordt geladen op de locatie waarnaar de program counter verwijst.
2. De instructie wordt uitgevoerd
3. De programmateller wordt verhoogd
4. Herhaal

Sommige machine-instructies, zoals jumps, passen expliciet de programmateller aan.
Zo implementeren we lussen, functies, enzovoort.

Het kan echter ook gebeuren dat een instructie *faalt*.
Denk bijvoorbeeld aan de volgende simpele instructie:

```asm
sd t0, (t1)
```

De waarde in register `t0` wordt opgeslagen op het adres in register `t1`.
Indien we deze instructie uitvoeren op een RISC-V processor waarop *virtual memory* enabled is, in bijvoorbeeld *user mode*, kan het zijn
dat deze instructie niet mag uitgevoerd worden.
Het kan namelijk zijn dat de pagina van het adres niet gemapt is als *writable*.
Hoe kan de processor deze instructie dan uitvoeren?

In dit specifieke scenario zal de processor een *exception* genereren.
Iedere *exception* veroorzaakt een *trap*.
In plaats van de *faulting instruction* uit te voeren, zal de processor springen naar een andere locatie in het geheugen.
Op die locatie vinden we een *trap handler*.
Dit is code, typisch een deel van het besturingssysteem, op een vaste locatie in het geheugen.
Deze code heeft de specifieke taak om te identificeren wat het probleem was en om vervolgens te beslissen of het al dan niet opgelost kan worden.

Na het lezen van bovenstaande paragraaf maak je misschien enkele terechte bedenkingen: in welke adresruimte bevindt zich de trap handler?
In de adresruimte van de kernel, of die van het user proces?
Of staat de trap handler op een fysiek adres?
Wijzigt het privilegeniveau van de processor bij het afhandelen van een trap?

Om hierop te antwoorden moeten we even uitleggen hoe exact een trap in RISC-V afgehandeld wordt.

## Traps in RISC-V

### Control and status registers

RISC-V heeft een verzameling speciale registers genaamd *control and status registers* (CSR).
Deze registers worden specifiek gebruikt om bepaalde speciale functionaliteiten te kunnen implementeren, zoals exceptions.
In onderstaande afbeelding uit de [RISC-V specificaties](https://riscv.org/technical/specifications/) worden enkele van deze CSRs gerelateerd aan exception handling beschreven:

![CSR](../../../img/CSR.png)

Merk op dat elk van deze registers de letter `s` als prefix heeft.
Deze `s` staat voor supervisor-mode.
Deze specifieke registers kunnen alleen gebruikt worden door de processor, wanneer deze in supervisor mode (of een hoger privilegeniveau) uitvoert.
Sommige van deze registers hebben ook een `u`- en `m`-equivalent (bijvoorbeeld `ustatus` en `mstatus`).
`ustatus` kan gelezen worden in alle modes, `mstatus` enkel in machine mode.

### Delegation register

Op het moment dat een trap optreedt, moet beslist worden in welk privilegeniveau deze afgehandeld moet worden.
Standaard is dit in machine mode, het hoogste privilegeniveau.
Er is echter een mogelijkheid voorzien om traps te forwarden naar supervisor mode of user mode.

Om traps te forwarden wordt gebruik gemaakt van speciale delegatieregisters (dit zijn ook CSRs).
In totaal zijn er vier verschilende delegatieregisters: `medeleg` en `sedeleg` voor exceptions, en `mideleg` en `sideleg` voor interrupts.
Voorlopig beschouwen we alleen exceptions, we komen later in de sessie terug op interrupts.

Indien de oorzaak van de trap een exception was, wordt gekeken in het `medeleg` register.
Elke bit van dit register is gelinkt aan een specifieke exception.
Indien de bit van een specifieke exception op `1` staat, wordt deze exception doorgestuurd naar het volgende (lagere) privilegeniveau.
Indien de specifieke processor een supervisor mode heeft, zal de exception dus in supervisor mode afgehandeld worden.
Via `sedeleg` kunnen vervolgens exceptions verder doorgestuurd worden naar user mode, op dezelfde wijze.

### Trap vectors

Eenmaal bepaald is in welke modus een exception moet afgehandeld worden, worden de overeenkomstige CSRs geconsulteerd.
De registers `mtvec`, `stvec` en `utvec` bevatten respectievelijk de adressen van de machine mode trap handler, supervisor mode trap handler en user mode trap handler.

Stel dat de exception in machine mode wordt afgehandeld.
De oorzaak van de exception wordt geschreven naar het CSR `mcause`.
De waarde van de program counter op het moment dat de exception zich voordeed, wordt geschreven naar `mepc`.
Indien de exception veroorzaakt werd door een adres aan te spreken, zal de waarde van dit adres in `mtval` geplaatst worden.
In de andere privilegeniveaus gebeurt exact hetzelfde, maar dan met de registers van dat privilegeniveau.

De tabellen uit de specs voor [mcause](../../../img/mcause.png) en [scause](../../../img/scause.png) kunnen gebruikt worden om te bepalen welke waarde in deze registers geplaatst worden per interrupt of exception.

> :question: **Oefening** Een programma in user-mode voert de instructie op adres `0x4B1D` uit: `sd t0, 0x0FF1C0`. Het adres `0x0FF1C0` bevindt zich in een pagina die niet schrijfbaar is. De waarde van het `medeleg` register staat op `0xffffffff`, de waarde van het `sedeleg` register op `0x0`. In `mtvec` staat het adres `0xdeadbeef`, in `stvec` staat het adres `0xcafebabe` en in `utvec` staat het adres `0xBAADF00D`. Op welk adres bevindt zich de trap handler die de exception zal afhandelen? In welk privilegeniveau wordt de exception afgehandeld? Welke waarden zullen er in de `xcause`, `xepc` en `xtval` registers geplaatst worden (en welke letter is *x*)?

> :bulb: Een pagina aanspreken op een niet-toegelaten manier veroorzaakt altijd een page fault. Het soort instructie dat je uitvoert bepaalt de soort page fault die zal optreden. Instructies die schrijven naar geheugen zonder toegang veroorzaken store page faults, instructies die lezen uit geheugen zonder toegang load page faults. Een instruction page fault treedt op wanneer de instructie die uitgevoerd moet worden op een pagina staat zonder de correcte permissies.

## Virtual memory

We weten nu het hardware-mechanisme waarmee traps afgehandeld kunnen worden.
Er rest ons echter nog een belangrijke vraag om te beantwoorden: worden virtuele of fysieke adressen gebruikt bij het afhandelen van traps?
Dit hangt volledig af van de huidige configuratie van de processor en van het privilegeniveau waarin de trap afgehandeld wordt.
Machine-mode ondersteunt geen virtual memory dus wanneer een trap in die mode afgehandeld wordt door naar `mtvec` te springen, zullen er fysieke adressen gebruikt worden.
In de andere modes hangt het af van de huidige configuratie van virtual memory.
Indien paging aanstond op het moment dat de trap zich voordoet, zal de waarde in de `xtvec` registers verwijzen naar virtuele adressen via de huidige page table in `satp`.
Het adresvertalingsmechanisme blijft dus actief.

Zeer attentieve studenten merken hierbij misschien een probleem op.
Stel dat we van privilegeniveau wijzigen door de exception.
Neem bijvoorbeeld aan dat we een exception hebben die zich voordoet in user-mode en afgehandeld wordt in supervisor-mode.
Op het moment dat de processor springt naar de waarde in `stvec` zal `satp` nog steeds verwijzen naar de page table van het user proces.
De adresvertaling gebeurt dus nog steeds volgens de mapping van het user proces. De processor zal dus springen naar code in de user adresruimte met een hoger privilegeniveau (in dit geval supervisor mode).

Een besturingssysteem zal er dus voor zorgen dat er kernel code geschreven wordt op het adres `stval` en dat het proces zelf deze code niet kan bewerken (door de pagina niet schrijfbaar te maken en de `U`-bit te deactiveren).
De code op dat adres kan vervolgens de waarde van `satp` wijzigen, en zo de page tables van de kernel activeren, om ervoor te zorgen dat we verder kunnen uitvoeren in de adresruimte van de kernel zelf.

De code die verantwoordelijk is voor het opvangen van traps in xv6 bevindt zich in de trampolinepagina.
De trampolinepagina is gemapt op het hoogste adres in de virtuele adresruimte van *elk* user space proces.

### Trampoline

Het is je misschien opgevallen dat de *trampoline* niet enkel gemapt is in de adresruimte van elk user space proces maar ook in de adresruimte van de kernel zelf.
Ook in de kernel is deze pagina gemapt op het hoogst mogelijke virtuele adres.

Eerder hebben we vermeld dat de trampolinepagina `satp` zal wijzigen.
`satp` wijzigen zorgt er echter voor dat plotseling alle virtuele adressen volgens een volledig andere mapping vertaald worden.
De program counter zelf bevat het virtuele adres van de huidige locatie in de code!
Indien, bij een wisseling van user space naar kernel space, de trampolinepagina niet op hetzelfde virtuele adres gemapt is in beide adresruimten, zal de volgende instructie die geladen wordt door de processor zich niet bevinden in de trampolinepagina.
Om dat te vermijden wordt deze pagina dus op hetzelfde virtuele adres gemapt in iedere adresruimte.

* Bekijk nu de code van [`kernel/trampoline.S`][trampoline]

Zoals je kan zien voert de trampoline twee taken uit:

1. Het [bewaren][trampoline save regs] (en [herstellen][trampoline restore regs]) van alle registerwaarden in de [`trapframe`][trapframe]. Dit is nodig om te verzekeren dat bij terugkeer uit de trap, het actieve programma kan voortgezet worden met correcte registerwaarden.
  Merk op dat xv6 het `sscratch` CSR gebruikt om het adres van de trapframe van het huidige proces op te slaan.
  Dit is een speciaal register waar de processor zelf nooit iets mee doet en dus door de kernel gebruikt mag worden als _scratch_ register.
2. Het wijzigen van de `satp` om zo te wisselen van page tables bij transities tussen user space en kernel space ([naar kernel][satp kernel], [naar user][satp user]).
  Na het switchen van de page table wordt de `sfence.vma` instructie gebruikt om de TLB te flushen.

Bij de overgang van user space naar kernel space wordt, aan het einde van de trampolinepagina, de kernel-functie [`usertrap()`][usertrap] opgeroepen in `kernel/trap.c`.
Om terug te keren van kernel space naar user space definieert de trampolinepagina de functie `userret`.

> :question: **Oefening** De trampolinepagina staat gemapt met `R` (read) en `X` (execute) permissies. Daarnaast is de pagina enkel toegankelijk in supervisor mode (de `U`-bit is inactief). Stel dat de trampolinepagina ook `W` (write) permissies zou hebben en user-mode access zou enabled zijn. Wat voor probleem zou dit kunnen opleveren?
 
[trampoline]: https://github.com/besturingssystemen/xv6-riscv/blob/1f555198d61d1c447e874ae7e5a0868513822023/kernel/trampoline.S
[trapframe]: https://github.com/besturingssystemen/xv6-riscv/blob/2b5934300a404514ee8bb2f91731cd7ec17ea61c/kernel/proc.h#L52
[trampoline save regs]: https://github.com/besturingssystemen/xv6-riscv/blob/103d9df6ce3154febadcf9a67791d526ec6b07ac/kernel/trampoline.S#L27-L65
[trampoline restore regs]: https://github.com/besturingssystemen/xv6-riscv/blob/103d9df6ce3154febadcf9a67791d526ec6b07ac/kernel/trampoline.S#L104-L134
[satp kernel]: https://github.com/besturingssystemen/xv6-riscv/blob/103d9df6ce3154febadcf9a67791d526ec6b07ac/kernel/trampoline.S#L76-L79
[satp user]: https://github.com/besturingssystemen/xv6-riscv/blob/103d9df6ce3154febadcf9a67791d526ec6b07ac/kernel/trampoline.S#L95-L97
[usertrap]: https://github.com/besturingssystemen/xv6-riscv/blob/27057bc9b467db64a3de600f27d6fa3239a04c88/kernel/trap.c#L32

