---
layout: default
title: Pagetables inspecteren
nav_order: 9
parent: "Levenscyclus proces"
grand_parent: "Zitting 3: Virtual memory"
---

## Pagetables inspecteren

Om het gemakkelijk te maken de pagetables van processen te bekijken, hebben we een syscall toegevoegd: [`vmprintmappings`][sys_vmprintmappings].
Deze syscall zal alle geldige mappings van het oproepende proces afprinten in het volgende formaat:

<pre>
{va} -> {pa}, mode={U|S}, perms={r|-}{w|-}{x|-}
</pre>

Hier is `va` het virtuele adres van een page, `pa` het fysieke adres van het overeenkomende frame.
`mode` is `U` voor een user page of `S` voor een supervisor (kernel) page.
`perms` toont de permissie flags voor de pagina: readable (`r`), writable (`w`), en executable (`x`).
Elk veldje bevat een `-` als de permissie niet gezet is.

De implementatie van `vmprintmappings` vind je in [`vm.c`][sys_vmprintmappings impl].
Alhoewel je zeker niet elk detail hoeft te begrijpen, is het nuttig om de implementatie eens te bekijken.

- Roep `vmprintmappings` in je hello world programma en vergelijk het resultaat met figuur 3.4 in het xv6 boek.
  Probeer elke mapping te begrijpen en kijk zeker naar de `mode` en `perms` velden.
- Maak een programma dat `vmprintmappings` oproept voor en na een oproep naar `sbrk(1)`.
  Verklaar het verschil in de outputs.

[sys_vmprintmappings]: https://github.com/besturingssystemen/xv6-riscv/blob/b26f9c647c1b8d27b7a7b3b374422c87591a8e1a/kernel/sysproc.c#L99
[sys_vmprintmappings impl]: https://github.com/besturingssystemen/xv6-riscv/blob/b26f9c647c1b8d27b7a7b3b374422c87591a8e1a/kernel/vm.c#L433-L460
