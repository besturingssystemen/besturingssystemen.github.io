---
layout: default
title: "Oefening: RISC-V System call"
nav_order: 3
parent: "System calls vs function calls"
grand_parent: "Zitting 2: System calls"
---

# RISC-V System Call

Nu functie-oproepen in assembly weer vers in het geheugen zit, is het tijd om te kijken naar de werking van een system call.

Alle system calls in RISC-V worden uitgevoerd met behulp van de `ecall` instructie. Deze instructie, wanneer uitgevoerd vanuit user mode, zal ervoor zorgen dat de processor overgaat naar *supervisor mode* en vervolgens (via de trampoline, hierover meer in de sessie over Virtual Memory) springt naar de eerder besproken trap handler. Herinner je dat in die trap handler de registers van het user-space programma bewaard worden.

De trap handler zal vervolgens de oorzaak van de trap bepalen. Indien de trap veroorzaakt was door een `ecall` weten we dat de gebruiker een system call probeerde op te roepen.
Er wordt dus gesprongen naar de system call handler, met name de functie `syscall(void)`.

* Bekijk de functie [`syscall`][syscall] in de code van xv6. Welk register wordt hier gebruikt om te bepalen welke system call opgeroepen moet worden?
* Voeg nu een bestand `user/hello_asm_write.S` toe (let op de hoofdletter `S` in de extensie). Maak gebruik van de `ecall` instructie om de system call `write` uit te voeren. Je zal het register uit bovenstaande vraag moeten gebruiken om de correcte system call op te vragen. Je kan starten vanuit onderstaande code:

```s
#include "kernel/syscall.h"

.text
.global main
main:
    #TODO implement

    #HINT: you can use the symbols defined in the header
    #file kernel/syscall.h
    #e.g. li t0, SYS_fork loads the value 1 into t0

.section .rodata
hello_str:
    .string "Hello, world!\n"
```

> :bulb: Naast de selectie van de system call zal het ook nodig zijn om parameters door te geven aan `write`. Dit gebeurt volgens dezelfde conventies als bij een gewone functie-oproep met `jal`. De signature van `write` is `int write(int fd, const void *buf, int nbytes)`. Gebruik dus de correcte registers om deze parameters door te geven.  

We weten nu hoe we een system call opgeroepen wordt vanuit user-space, hoe de trap handler en system call handler ervoor zorgen dat de juiste system call wordt uitgevoerd en we begrijpen hoe een C-wrapper functie dit voor een programmeur erg eenvoudig kan maken.
Tijd om eens te kijken naar de implementatie van zo'n system call.

* Bekijk de implementatie van de eenvoudige system call [`getpid`][sys_getpid]. Zo eenvoudig kan het zijn.  
* Kijk eens naar de andere system calls die in hetzelfde bestand zijn geïmplementeerd. Het is nog niet nodig om deze allemaal in detail te begrijpen.

[syscall]: https://github.com/besturingssystemen/xv6-riscv/blob/2b5934300a404514ee8bb2f91731cd7ec17ea61c/kernel/syscall.c#L133
[sys_getpid]: https://github.com/besturingssystemen/xv6-riscv/blob/2b5934300a404514ee8bb2f91731cd7ec17ea61c/kernel/sysproc.c#L21
