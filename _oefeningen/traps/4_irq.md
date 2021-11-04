---
layout: default
title: "Achtergrond: Interrupts"
nav_order: 4
parent: "Zitting 4: Traps"
---

# Interrupts
{: .no_toc }

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>


Tot nog toe hebben we voornamelijk gesproken over exceptions, die traps veroorzaken waarbij de controle wordt doorgegeven aan een trap handler.
Traps kunnen echter ook veroorzaakt worden door een tweede mechanisme, net iets complexer, genaamd *interrupts*.

Op sommige momenten is het nodig om informatie te geven aan processen die actief zijn, zonder dat de programma's op dat moment zelf om die informatie vragen.

Denk bijvoorbeeld je toetsenbord.
Op het moment dat je een toets indrukt, verschijnt een letter op het scherm.
De kans is echter zeer groot dat op het moment dat je die toets indrukt,
de processor druk bezet is met het uitvoeren van andere, ongerelateerde instructies.
Er is dus een manier nodig om de processor te onderbreken.

Exceptions onderbreken de flow van een proces, maar deze worden veroorzaakt door het uitvoeren van een (al dan niet foutieve) instructie.
Toetsaanslagen kan je echter niet voorspellen en kunnen voorkomen op elk mogelijk moment gedurende de uitvoer van een proces.
Wanneer we de flow van een processor willen onderbreken op een niet-voorspelbaar tijdstip maken we gebruik van een *interrupt*.

Tussen elke instructie die een processor uitvoert zal gecontroleerd worden of er een interrupt actief is.
Indien dit het geval is, trapt de processor.
Het is mogelijk om interrupts tijdelijk volledig te disablen, om ervoor te zorgen dat bepaalde code nooit onderbroken kan worden.

## Interrupts in xv6

Interrupts zijn standaard actief gedurende de uitvoering van xv6-code.
Indien een interrupt voorkomt, zal de processor, wanneer deze klaar is met het uitvoeren van de actieve instructie, springen naar de trap handler in onze welgekende trampolinepagina, net zoals in het geval van exceptions.

Zoals we weten roept de trampoline [usertrap()][usertrap] op, waarin vervolgens de functie [`devintr()`][devintr] wordt opgeroepen.
In deze functie kunnen we zien dat xv6 drie soorten interrupts kan afhandelen: `UART` interrupts, `VIRTIO` interrupts en timer interrupts.

## Device drivers

[*UART*](https://en.wikipedia.org/wiki/Universal_asynchronous_receiver-transmitter) staat voor *universal asynchronous receiver-transmitter*.
Dit is een apparaat dat gebruikt kan worden om (seriële, dus bit per bit) communicatie mogelijk te maken tussen een processor en externe apparaten.
Vroeger verliep de communicatie tussen een toetsenbord en een processor bijvoorbeeld typisch via een UART-apparaat.
Vandaag verloopt dit meestal via een *universal serial bus*, beter gekend als [*USB*](https://en.wikipedia.org/wiki/USB).

Wanneer we een toetsenbord aansluiten, bijvoorbeeld via een UART of via USB, kan informatie verstuurd worden naar de processor, zoals de ingedrukte toetsaanslagen. Om deze informatie echter te ontvangen, hebben we nood aan een programma dat weet hoe te communiceren met het apparaat.
Dat soort programma noemen we een *driver*.

De `qemu` emulator emuleert een UART-apparaat.
Wanneer je in je Ubuntu-console xv6 opstart en vervolgens in de terminal typt, wordt deze informatie via de geëmuleerde UART doorgestuurd.
In xv6 lijkt het dus vervolgens alsof er rechtstreeks via een UART informatie binnenkomt.

xv6 heeft dus ook een UART driver. Deze kan je terugvinden in [`kernel/uart.c`][uart].
Driver code is complex en zeer gebonden aan specifieke apparaten, en dus ook weinig interessant om te bestuderen.
Wel interessant is om te kijken hoe de samenwerking met interrupts net werkt.

Wanneer we drukken op een toets van een toetsenbord aangesloten via UART op xv6, gebeuren de volgende stappen:

1. De ingedrukte toets gestuurd van het toetsenbord naar de UART
2. De UART genereert vervolgens een IRQ (interrupt request) op de processor
3. Indien interrupts actief zijn zal de processor na afloop van de huidige instructie springen naar de correcte trap handler (ook voor interrupts zijn er delegatieregisters)
4. De trap handler determineert dat de reden van de trap een interrupt was en geeft de controle door aan [`devintr()`][devintr]
5. [`devintr()`][devintr] leest de waarde van de IRQ, bepaalt dat het een UART-interrupt was en geeft de controle door aan [`uartintr()`][uartintr] in de UART driver.
6. [`uartintr()`][uartintr] leest een karakter uit de UART en stuurt dit door naar de console van xv6

UART communicatie kan in twee richtingen werken.
De console van xv6 stuurt de output van de console ook via UART terug naar `qemu`, die het toont in je Ubuntu-terminal.

Merk op dat de essentie van deze device driver dus gebaseerd is op interrupts.
UART drivers kunnen ook geïmplementeerd worden zonder interrupts.
Dit kan door met behulp van een proces dat voortdurend actief is in een lus kijkt of er een karakter in de UART buffer staat.
Deze techniek noemen we *polling*.
Voor apparaten die weinig invoer sturen op onregelmatige momenten zijn interrupts meestal efficiënter.
Indien een apparaat voortdurend informatie verstuurt kan polling de betere oplossing zijn, om interrupt overhead te vermijden.

De `VIRTIO` driver ten slotte wordt gebruikt om een harde schijf te ondersteunen. Hier gaan we in deze oefenzitting verder niet op in.

## Timer interrupts

Wanneer meerdere processen op hetzelfde moment actief zijn binnen een besturingssysteem, zal de processor in vele gevallen deze processen afwisselend uitvoeringstijd toekennen.
Indien dit snel genoeg verloopt, krijg je de illusie dat de programma's effectief in parallel draaien.
Wat dit in feite wil zeggen, is dat er een manier nodig is om ervoor te zorgen dat programma's om de zoveel tijd onderbroken worden.
Het besturingssysteem wil dus in feite een proces opstarten en een timer instellen.
Wanneer deze timer afloopt, is een nieuw proces aan de beurt.

Om dit mogelijk te maken wordt vaak een hardwarematige timer gebruikt.
Deze timer kan geprogrammeerd worden om een interrupt te sturen na een instelbare tijd.
Dit soort interrupt noemen we een timer interrupt.

Deze timer wordt in xv6 opgestart bij het booten van het besturingssysteem.
Met alle informatie die we in deze oefenzitting verzameld hebben is het interessant om eens een kijkje te nemen in de boot code.

* Lees de code in [kernel/start.c][start]. Probeer de code te begrijpen aan de hand van de commentaren en de kennis die je in deze sessie hebt opgedaan.

Deze code wordt opgeroepen als deel van de boot van xv6 en bevat de code waarin de processor geconfigureerd wordt. 
Aan het einde van de configuratie wordt (via `mret`) overgegaan naar supervisor mode en de controle doorgegeven aan de main-functie in de kernel.
Eerder hebben jullie hier reeds de floating point unit van de processor aangezet.
Merk op dat hier de timer interrupts ook geactiveerd worden.

* Hoeveel clockcycli zal een proces in xv6 ongeveer kunnen uitvoeren alvorens het wordt onderbroken door een timer interrupt?

Timer interrupts in RISC-V worden altijd opgevangen door machine mode en worden dus niet gedelegeerd naar supervisor mode of user mode via delegatieregisters.
In xv6 zijn dit de enige interrupts en exceptions die door machine mode worden afgehandeld.
Je kan in [`start()`][start] zien dat alle andere exceptions gedelegeerd worden naar supervisor mode.
In de functie [`timerinit()`][timerinit] wordt om die reden de machine mode trap handler geconfigueerd zodat deze verwijst naar een trap handler specifiek geschreven voor timer interrupts, namelijk de functie [`timervec`][timervec] in `kernel/kernelvec.S`.

Om ervoor te zorgen dat timer interrupts uiteindelijk toch in supervisor mode afgehandeld kunnen worden, zal deze functie, na het [instellen van de volgende time interrupt][schedule next timer irq], een _supervisor software interrupt_ [triggeren][raise ssi].
Zoals de naam doet uitschijnen, is een software interrupt een interrupt die niet door externe hardware, maar door software expliciet getriggered wordt.
In RISC-V kunnen interrupts door software getriggered worden door naar een _interrupt pending_ CSR te schrijven, in dit geval het `sip` CSR.
Dit zorgt ervoor dat een timer interrupt indirect toch in supervisor mode [opgevangen kan worden][handle ssi].



[usertrap]: https://github.com/besturingssystemen/xv6-riscv/blob/27057bc9b467db64a3de600f27d6fa3239a04c88/kernel/trap.c#L32
[devintr]: https://github.com/besturingssystemen/xv6-riscv/blob/27057bc9b467db64a3de600f27d6fa3239a04c88/kernel/trap.c#L177
[uart]: https://github.com/besturingssystemen/xv6-riscv/blob/6781ac00366e2c46c0a4ed18dfd60e41a3fa4ae6/kernel/uart.c
[uartintr]: https://github.com/besturingssystemen/xv6-riscv/blob/6781ac00366e2c46c0a4ed18dfd60e41a3fa4ae6/kernel/uart.c#L180
[start]: https://github.com/besturingssystemen/xv6-riscv/blob/103d9df6ce3154febadcf9a67791d526ec6b07ac/kernel/start.c#L19
[timerinit]: https://github.com/besturingssystemen/xv6-riscv/blob/103d9df6ce3154febadcf9a67791d526ec6b07ac/kernel/start.c#L57
[timervec]: https://github.com/besturingssystemen/xv6-riscv/blob/bebecfd6fd449fb86f73b81982f8c90e5b6bbf90/kernel/kernelvec.S#L93
[schedule next timer irq]: https://github.com/besturingssystemen/xv6-riscv/blob/103d9df6ce3154febadcf9a67791d526ec6b07ac/kernel/kernelvec.S#L104-L110
[raise ssi]: https://github.com/besturingssystemen/xv6-riscv/blob/103d9df6ce3154febadcf9a67791d526ec6b07ac/kernel/kernelvec.S#L112-L114
[handle ssi]: https://github.com/besturingssystemen/xv6-riscv/blob/103d9df6ce3154febadcf9a67791d526ec6b07ac/kernel/trap.c#L203-L215

