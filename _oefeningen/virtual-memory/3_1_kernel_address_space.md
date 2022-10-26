---
layout: default
title: Kernel address space
nav_order: 7
parent: "Address spaces in xv6"
grand_parent: "Zitting 3: Virtual memory"
---

## Kernel address space

In hoofdstuk 3 van het xv6 boek kwamen we de volgende figuur tegen:

![kernel-address-space](../../../../img/xv6-kernel-address-space.png)

De functie [`kvmmake`][kvmmake] roept `mappages` op (via `kvmmap`) om de address space van de kernel op te bouwen.

* Bekijk de functie [`kvmmake`][kvmmake]. Deze code zou ondertussen begrijpbaar moeten zijn.

Dankzij `kvmmake` zijn de page tables van de kernel geïnitialiseerd zodat de virtuele adresruimte van de kernel bovenstaande structuur volgt.
Op het moment dat code in de kernel uitvoert, wijst het `satp`-register naar de top-level page table van de kernel.
Hierdoor worden adressen vertaald zoals afgebeeld.

Zo zie je dat de code (`text`-sectie) van de kernel ingeladen is op adres `0x80000000`.
De pagina's met code zijn gemapt als read/execute.
Net boven de code wordt de `data`-sectie (globale variabelen) van de kernel gemapt.

### Identity mapping

De mapping van de kernel heeft een speciale structuur.
De kernel code en data volgen een identity mapping.
Elk fysisch adres wordt gemapt op hetzelfde overeenkomstige virtueel adres.
Hiermee bedoelen we: virtueel adres `0x80000000` wordt gemapt op fysisch adres `0x80000000`.
Virtueel adres `0x80000001` wordt gemapt of fysisch adres `0x80000001`, enzovoort.

Er is een belangrijke reden om deze mapping op deze manier uit te voeren.
De code om paginatabellen te bewerken bevindt zich in de text section van de kernel.
Deze code moet voortdurend kunnen schrijven naar fysieke adressen.
Door de identity map werk je in feite rechtstreeks met fysieke adressen, waardoor page table code veel eenvoudiger geschreven kan worden.
De code die page tables bewerkt in xv6 zou niet werken zonder deze identity map.

### Kernel stacks

De kernel reserveert voor ieder proces een eigen *kernel stack*.
Deze stack wordt gebruikt als *call stack* op het moment dat een proces switcht naar supervisor-mode, bijvoorbeeld als gevolg van een `ecall` of een *exception*.

Rond elke kernel stack vind je *guard pages*.
Dit is meteen een interessante toepassing van virtual memory.
De grootte van de process kernel stack wordt door xv6 gelimiteerd tot 1 pagina.
Indien een stack groter wordt dan het gealloceerde geheugen spreken we over een *stack overflow*.
Zonder bescherming tegen een *stack overflow* zou dit ervoor kunnen zorgen dat de stack kritische kerneldata overschrijft.
Een stack kan ook *underflowen*.
Wanneer je bijvoorbeeld een element popt van een lege stack zal de stack pointer buiten het stackgeheugen wijzen.

Om te detecteren wanneer een xv6 kernel stack overflowt of underflowt, worden de pagina's rond de stack niet gemapt.
Wanneer je probeert te schrijven naar een unmapped pagina krijg je een page fault.
Op die manier kunnen stack overflows en stack underflows automatisch gedetecteerd en vermeden worden.

[kvmmake]: https://github.com/besturingssystemen/xv6-riscv/blob/d4cecb269f2acc61cc1adc11fec2aa690b9c553b/kernel/vm.c#L20
