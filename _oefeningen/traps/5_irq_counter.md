---
layout: default
title: "Oefening: Interrupt counters"
nav_order: 5
parent: "Zitting 4: Traps"
---

# Oefening: Interrupt counters

Interrupts hebben een belangrijke invloed op performantie en de correcte
werking van het systeem. Moderne besturingssystemen bieden daarom typisch
gedetailleerde "interrupt accounting" functionaliteit aan.
In deze oefening kijken we eerst hoe we inzicht kunnen krijgen in het
interruptgedrag op een modern Linux systeem. Daarna gaan we zulke
basisfunctionaliteit toevoegen aan xv6.

## Interrupt accounting in Linux 

Voer op je Linux distributie het volgende commando uit (je kan dit commando afsluiten met CTRL-C):

```shell
[ubuntu-shell]$ watch -n 1 cat /proc/interrupts
```

Wat zie je nu? Wat betekenen deze getallen volgens jou? Waarom variëren deze
getallen over tijd?

> :information_source: Het is niet belangrijk de exacte betekenis van elke
> interrupt source die hier opgelijst is, te begrijpen (dit is afhankelijk van
> de onderliggende computerarchitectuur en kan snel erg technisch worden!). De
> bedoeling van deze stap is slechts je een idee te geven van interruptfrequentie
> en accounting in een real-world besturingssysteem.

## Interrupt accounting in xv6

xv6 houdt momenteel niet bij hoe vaak interrupts voorkomen.
We gaan nu een syscall toevoegen die ons informatie geeft over het aantal interrupts dat er zijn gebeurd per CPU sinds xv6 opgestart is.

Jouw taak is de output van Linux's `/proc/interrupts` zo goed mogelijk na te bootsen.
Meer bepaald, zijn er zijn drie types interrupts die xv6 afhandelt: timer, UART
en disk (VIRTIO). We willen aparte informatie krijgen over alle types en voor
elke afzonderlijke CPU. Voorbeeldoutput is als volgt:

```shell
[xv6-shell]$ dumpirq
	CPU0	CPU1	CPU2
 TMR:	30	30	30
UART:	24	21	6
DISK:	20	3	1
```

1. Eerst moeten we ervoor zorgen dat er interrupt tellers per individuele CPU
   kunnen worden bijgehouden.

   > :bulb: Meer over multitasking volgt in de volgende oefenzitting. Voorlopig
   > is het voldoende te weten dat xv6 een maximum van `NCPU=8` processors
   > ondersteunt, waarvoor belangrijke metadata wordt bijgehouden in een
   > [globale array][cpus] van het type `struct cpu cpus[NCPU]`.

   1. Breid het datatype `struct cpu` in [proc.h][struct cpu] uit met een aparte
   teller (van het type integer) per interruptbron (timer, UART, disk) dat we
   gaan tellen voor elke CPU.
   2. Initialiseer de toegevoegde tellers op nul wanneer de CPUs voor het
   eerst beginnen uit te voeren, i.e., aan het begin van de [scheduler()][scheduler] functie.

2. Momenteel registreert xv6 niet expliciet tijdens het opstarten welke van de
   maximaal 8 ondersteunde CPUs effectief bestaan en welke niet.
   We willen interrupts echter alleen afprinten voor CPUs die effectief bestaan!

   > :bulb: Standaard worden er slechts 3 (van de maximaal 8) processors
   > geëmuleerd in QEMU. Je kan dit optioneel overriden door xv6 als volgt op
   > te starten:
   > ```shell
   > [ubuntu-shell]$ CPUS=4 make qemu
   > ```

   1. Breid het datatype `struct cpu` in [proc.h][struct cpu] uit met een expliciet
   `init` veld dat aangeeft of deze CPU bestaat, i.e., opgestart is.
   2. Initialiseer het `init` veld wanneer de CPUs voor het eerst beginnen uit
   te voeren, i.e., aan het begin van de [scheduler()][scheduler] functie.

3. Nu moeten we logica toevoegen in de [`devintr()`][devintr] functie om de
   verschillende soorten interrupts effectief te tellen. Gebruik hiervoor
   de functie `mycpu()` als volgt: `struct cpu *cpu = mycpu();`.
   Met behulp van deze pointer kan nu dan de juiste teller in `cpu` ophogen
   afhankelijk van welke interrupt `devintr()` juist behandelt.

   > :bulb: Timer interrupts worden naar _alle_ CPUs gestuurd maar er is er maar één die de klok bijhoudt.
   > Waar gebeurt dit?
   > Zorg ervoor dat je elke timer interrupt één keer telt per CPU.

4. Voeg een nieuwe syscall `printinterrupts()` toe die deze tellers afprint. Kijk
   hiervoor zo nodig nog eens naar de relevante oefening van de [vorige
   oefenzitting](../../system-calls/4_adding_syscall/). Je kan in
   `printinterrupts()` de globale `struct cpu cpus[NCPU]` array overlopen, maar
   zorg er wel voor dat je alleen tellers print voor CPUs die effectief bestaan
   (i.e., waarvan het `init` veld gezet is).

3. Schrijf ten slotte een simpel user-space programma `dumpirq` dat de nieuwe
   syscall gebruikt.

Run `dumpirq` een aantal keer en probeer de resultaten te interpreteren, bijvoorbeeld als volgt:

```shell
[ubuntu-shell] $ make qemu
qemu-system-riscv64 -machine virt -bios none -kernel kernel/kernel -m 128M -smp 3 -nographic -drive file=fs.img,if=none,format=raw,id=x0 -device virtio-blk-device,drive=x0,bus=virtio-mmio-bus.0

xv6 kernel is booting

hart 1 starting
hart 2 starting
init: starting sh
$ dumpirq
	CPU0	CPU1	CPU2
 TMR:	30	30	30
UART:	24	21	6
DISK:	20	3	1
$ dumpirq
	CPU0	CPU1	CPU2
 TMR:	62	62	62
UART:	38	30	15
DISK:	20	3	1
$ echo "hello, world!"
"hello, world!"
$ dumpirq
	CPU0	CPU1	CPU2
 TMR:	143	143	143
UART:	70	67	49
DISK:	24	3	1
$ halt
```

[cpus]: https://github.com/besturingssystemen/xv6-riscv/blob/485e428c26eeeb49fd530427ad30c3ad9ffd7216/kernel/proc.c#L9
[struct cpu]: https://github.com/besturingssystemen/xv6-riscv/blob/485e428c26eeeb49fd530427ad30c3ad9ffd7216/kernel/proc.h#L30
[scheduler]: https://github.com/besturingssystemen/xv6-riscv/blob/485e428c26eeeb49fd530427ad30c3ad9ffd7216/kernel/proc.c#L443
[devintr]: https://github.com/besturingssystemen/xv6-riscv/blob/27057bc9b467db64a3de600f27d6fa3239a04c88/kernel/trap.c#L177

