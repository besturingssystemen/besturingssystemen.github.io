---
layout: default
title: "Oefening: Interrupt counters"
nav_order: 5
parent: "Zitting 4: Traps"
---

# Oefening: Interrupt counters

We gaan nu een syscall toevoegen die ons informatie geeft over het aantal interrupts dat er zijn gebeurd sinds xv6 opgestart is.
Er zijn drie types interrupts die xv6 afhandelt: timer, UART en disk (VIRTIO) en we willen aparte informatie krijgen over alle types.
xv6 houdt momenteel niet bij hoe vaak interrupts voorkomen.

1. Pas [`devintr`][devintr] aan om per type interrupt een aparte teller te verhogen.
  > :bulb: Timer interrupts worden naar _alle_ CPUs gestuurd maar er is er maar één die het uitendelijk afhandeld.
  > Waar gebeurt dit?
  > Zorg ervoor dat je teller elke timer interrupt maar één keer telt.
2. Voeg een nieuwe syscall `printinterrupts` toe die deze tellers afprint.
3. Schrijf een user space programma dat de nieuwe syscall gebruikt.
  Run het een aantal keer en probeer de resultaten te interpreteren.

[devintr]: https://github.com/besturingssystemen/xv6-riscv/blob/27057bc9b467db64a3de600f27d6fa3239a04c88/kernel/trap.c#L177
