---
layout: default
title: "Permanente evaluatie"
nav_order: 5
parent: "Zitting 2: System calls"
search_exclude: true
---

# Traceme

Permanente evaluatie
{: .label .label-yellow}

Als permanente evaluatie van deze oefenzitting is het de bedoeling om zelfstandig een system call toe te voegen.

* Voeg een nieuwe system call `void traceme(int enable)` met nummer 25 toe die ervoor zorgt dat (als `enable` truthy is) elke system call van het proces geprint wordt naar de console, samen met de `pid` van het proces.
  * Gebruik de kernel-versie van [`printf`][kernel printf]
  * Volg het printformaat `[{pid}] syscall {num}`
  * Gebruik [`argint`][argint] om een syscall argument op te vragen.
* Maak een user-space programma `trace` dat een executable als argument krijgt en deze executable oproept met de `traceme` functionaliteit aangezet.
  * Hint: gebruik de system calls `traceme` en `exec`

# Testen

TODO

# Deadline

De deadline van deze permanente evaluatie valt op 26 oktober om 23h59.

# Indienen

Dien je oplossing in met behulp van GitHub classroom.

> :bulb: De testen worden automatisch uitgevoerd op GitHub wanneer je nieuwe code pusht.
> Verifieer dat alles werkt door naar de "Actions" tab te gaan.

# Bonusoefening

- Voeg een nieuwe system call `void tracemefd(int fd)` met nummer 26 toe. Let op: dit moet een *nieuwe* system call zijn, dus de `traceme` system call hierboven moet ongemodificeerd blijven werken.
- Wanneer `tracemefd` opgeroepen wordt, valideer je eerst de `fd` (geldige open file, is een pipe, is writable)
  * Zo ja, sla op in `struct proc::tracefd`.
  * Zo nee, zet dit veld op -1
- Wanneer er een syscall gebeurt en `tracefd` niet gelijk is aan -1, schrijf een `struct tracemsg` naar `tracefd` via [`pipewrite`][pipewrite] (zet `user_src` op 0 om aan te geven dat de input in kernel space zit)
- Schrijf een user space programma `tracef` dat gebruik maakt van deze nieuwe syscall

> :information_source: Deze oefening is niet verplicht. Werk eerst je permanente evaluatie af voor je hier aan zou beginnen. Eventuele code die je extra schrijft voor de bonusoefening mag nooit de testen van je verplichte oefening breken.

[kernel printf]: https://github.com/besturingssystemen/xv6-riscv/blob/2b5934300a404514ee8bb2f91731cd7ec17ea61c/kernel/printf.c#L64
[pipewrite]: https://github.com/besturingssystemen/xv6-riscv/blob/2b5934300a404514ee8bb2f91731cd7ec17ea61c/kernel/pipe.c#L77
[argint]: https://github.com/besturingssystemen/xv6-riscv/blob/2b5934300a404514ee8bb2f91731cd7ec17ea61c/kernel/syscall.c#L58
