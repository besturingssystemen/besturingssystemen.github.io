---
layout: default
title: "Permanente evaluatie"
nav_order: 5
parent: "Zitting 2: System calls"
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

# Deadline

De deadline van deze permanente evaluatie valt op 26 oktober om 23h59.

# Testen

We hebben een paar simpele testen gegeven die jullie kunnen gebruiken om te verifiëren dat er geen grote fouten gemaakt zijn.
Je kan deze uitvoeren via het volgende commando:

```console
[ubuntu-shell]$ make test
```

Let wel: we kijken jullie code ook nog handmatig na en het feit dat de testen slagen, wilt dus niet zeggen dat je een perfecte score zal halen!.

> :warning: Zorg steeds voor het indienen dat de testen zowel lokaal als in de GitHub Actions cloud werken.

> :bulb: De testen worden automatisch uitgevoerd op GitHub wanneer je nieuwe code pusht.
> Verifieer dat alles werkt door naar de "Actions" tab te gaan op de GitHub
> webinterface van je repository (of kijk naar het groene vinkje of rode
> kruisje dat naast je commit verschijnt).

# Indienen

Dit deel van de opgave moet ingediend worden en telt mee voor de permanente evaluatie van de oefeningen.
Dien je oplossing in met behulp van GitHub classroom.

* Commit en push de bestanden naar je repository

```console
[ubuntu-shell]$ git status # Aangepaste en nieuwe bestanden zijn aangegeven in het rood onder de heading "Changes not staged for commit" en "Untracked files"
[ubuntu-shell]$ git add . # Voeg alle aangepaste bestanden toe die nodig zijn
[ubuntu-shell]$ git status # Alle bestanden die je wil committen zouden nu aangegeven moeten zijn in het groen onder de heading "Changes to be committed"
[ubuntu-shell]$ git commit -m "Permanente evaluatie system calls"
[ubuntu-shell]$ git push
```

> :bulb: Controleer op de webpagina van je repository of het bestand correct gecommit is en de GitHub Actions testen slagen.

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
