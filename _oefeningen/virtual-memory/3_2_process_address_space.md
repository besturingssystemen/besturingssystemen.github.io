---
layout: default
title: Process address space
nav_order: 7
parent: "Address spaces in xv6"
grand_parent: "Zitting 3: Virtual memory"
---

## Process address space

Op het moment dat een proces in user-mode uitvoert zorgt de kernel ervoor dat het `satp`-register wijst naar de top-level page table van het huidige proces.
Elk proces heeft zo een eigen virtuele adresruimte.

Ook voor processen kiest xv6 een vaste layout om deze adresruimte op te delen:

![xv6 process address space](../../../../img/xv6-process-address-space.png)

Ieder proces krijgt een eigen pagina gealloceerd om de *call stack* van het programma in te bewaren.
Onder deze stackpagina wordt, net zoals in de kernel, een guard pagina geplaatst die niet gemapt wordt in het geheugen, zodat stack overflows gededecteerd kunnen worden.

> :warning: Merk op dat *boven* de stack in user space geen guard pagina te vinden is. Dat wil zeggen dat een xv6 user-space proces wel kan underflowen in het heap-geheugen. Het zou beter zijn ook hier een guard pagina te plaatsen.

De code van een proces wordt in xv6 gemapt op virtueel adres 0, dus op de eerste pagina in de virtuele adresruimte.
Deze code kan één of meerdere pagina's groot zijn.
Boven de code (op hogere addressen) wordt de global data van het proces gemapt.

### Security

Indien een user-space proces zou kunnen schrijven naar het geheugen van de kernel, zou dit rampzalig zijn.
Eender welk onschuldig, geïnstalleerd programma zou je besturingssysteem kunnen aanpassen en vervolgens je volledige machine kunnen overnemen.

Het zou voldoende zijn om eigen code te schrijven naar een pagina van de kernel, vervolgens `ecall` uit te voeren (waardoor de processor naar supervisor mode switcht) en ervoor te zorgen dat de trap handler jouw nieuwe code uitvoert in plaats van de oude kernel code.
Op dat moment wordt je code uitgevoerd met de volle rechten van het OS. Het onschuldige programma heeft dus volledige controle over alles wat er verder gebeurt op jouw machine.

Door gebruik te maken van een beschermde trampolinepagina en een aparte virtuele adresruimte voor de kernel, zorgt xv6 ervoor dat applicaties geen rechtstreekse toegang hebben tot het geheugen van de kernel.
Virtual memory zorgt er verder ook voor dat je geen toegang hebt tot het geheugen van andere processen.
Zo kan je browser niet aan de inhoud van je password manager, enzovoort.
