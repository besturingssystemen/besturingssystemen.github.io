---
layout: default
title: "Oefening: xv6 system call toevoegen"
nav_order: 4
parent: "Zitting 2: System calls"
---

In de vorige sessie hebben we geleerd hoe je een user-space programma kan toevoegen aan xv6.
Ondertussen zijn we klaar om onze eerste aanpassing te maken aan de kernel zelf.
We zullen onze eigen system call toevoegen.
Na deze sessie kan je jezelf dus officieel een OS programmeur noemen.

De system call die we gaan toevoegen is `getnumsyscalls`.
Wanneer een user-space programma deze system call uitvoert, krijgt deze als resultaat het totaal aantal uitgevoerde system calls in het huidige proces.

### Voorbereiding system call

Om dit mogelijk te maken moeten we in de process state in `struct proc` een teller bijhouden.

* Voeg `uint64 numsyscalls` toe aan de [`struct proc`][struct proc] in `kernel/proc.h`. Doe dit in de private sectie. (In de sessie over synchronizatie zal duidelijk worden waarom.)

Wanneer we een veld toevoegen aan de struct moeten we er uiteraard ook voor zorgen dat dit veld een initiële waarde toegewezen krijgt bij de aanmaak van een proces.

* Initialiseer het veld `numsyscalls` in de functie [`allocproc`][allocproc] in [`kernel/proc.c`][proc].
  De `allocproc` functie wordt door `fork` gebruikt om een nieuwe `struct proc` aan te maken.

Om ervoor te zorgen dat de teller het aantal system calls correct telt kunnen we deze waarde verhogen telkens wanneer de system call handler wordt opgeroepen.

* Verhoog `numsyscalls` in [`syscall`][syscall] in [`kernel/syscall.c`][syscall]

### Implementatie system call

Nu onze teller correct werkt, resteert ons enkel de effectieve implementatie van de system call.

* Voeg `SYS_getnumsyscalls` toe aan [`kernel/syscall.h`][syscall.h].
* Implementeer `sys_getnumsyscalls` in [`kernel/sysproc.c`][sysproc.c].
  Je kan een pointer naar het huidige proces krijgen via de [`myproc`][myproc] functie.
  (In een latere oefenzitting zal duidelijk worden hoe deze functie precies werkt.)
* Zorg ervoor dat de system call handler [`syscall`][syscall] onze nieuwe system call correct doorstuurt.

> :bulb: Om `sys_getnumsyscalls` te kunnen oproepen vanuit `syscall` zal je de C-compiler moeten vertellen dat deze functie bestaat, en deze dus moeten declareren. Volg het voorbeeld van de declaraties van de andere system calls in het bestand.

### System call exposen naar user-space

Onze system call werkt nu. We kunnen hem alleen nog niet eenvoudig oproepen vanuit user-space.
Herinner je dat system calls opgeroepen worden via een C-wrapper.
Laten we allereerst onze C-wrapper declareren:

* Voeg `int getnumsyscalls()` toe aan [`user/user.h`][user.h]

De implementatie van de wrapper-functie gebeurt in assembly. In principe hebben jullie ondertussen genoeg kennis om deze wrapper zelf te implementeren.
xv6 biedt echter een script aan waarmee dit volledig automatisch kan verlopen.
Het [`user/usys.pl`][usys.pl] script genereert automatisch een assembly-bestand (`user/usys.S`) met daarin de implementatie van de wrappers.
Bekijk dit gegenereerde bestand en zorg dat je de wrappers begrijpt.

- voeg `entry("getnumsyscalls")` toe aan [`user/usys.pl`][usys.pl]

### System call oproepen

De system call is klaar.
Tijd om  deze uit te testen.
We zullen ervoor zorgen dat onze C-runtime na afloop van ons programma print hoeveel system calls uitgevoerd werden.

* Bewerk `crt0.c` zodat `getnumsyscalls` opgeroepen wordt na de return uit `main` en print het resultaat uit.

Indien dit correct werkt zullen de user-space programma's die returnen uit main het aantal uitgevoerde system calls printen.

* Voer nu het programma `hi_asm_write` uit. Welk resultaat krijg je? Is dit het resultaat dat je verwacht had? Kan je achterhalen welke system calls werden uitgevoerd in het proces die je niet zelf expliciet hebt opgeroepen?

[struct proc]: https://github.com/besturingssystemen/xv6-riscv/blob/2b5934300a404514ee8bb2f91731cd7ec17ea61c/kernel/proc.h#L94
[allocproc]: https://github.com/besturingssystemen/xv6-riscv/blob/2b5934300a404514ee8bb2f91731cd7ec17ea61c/kernel/proc.c#L100
[proc]: https://github.com/besturingssystemen/xv6-riscv/blob/2b5934300a404514ee8bb2f91731cd7ec17ea61c/kernel/proc.c
[syscall]: https://github.com/besturingssystemen/xv6-riscv/blob/2b5934300a404514ee8bb2f91731cd7ec17ea61c/kernel/syscall.c#L133
[syscall.h]: https://github.com/besturingssystemen/xv6-riscv/blob/2b5934300a404514ee8bb2f91731cd7ec17ea61c/kernel/syscall.h
[myproc]: https://github.com/besturingssystemen/xv6-riscv/blob/2b5934300a404514ee8bb2f91731cd7ec17ea61c/kernel/proc.c#L75
[usys.pl]: https://github.com/besturingssystemen/xv6-riscv/blob/2b5934300a404514ee8bb2f91731cd7ec17ea61c/user/usys.pl
[sysproc.c]: https://github.com/besturingssystemen/xv6-riscv/blob/2b5934300a404514ee8bb2f91731cd7ec17ea61c/kernel/sysproc.c
[user.h]: https://github.com/besturingssystemen/xv6-riscv/blob/2b5934300a404514ee8bb2f91731cd7ec17ea61c/user/user.h
