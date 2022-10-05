---
layout: default
title: Userspace
nav_order: 2
parent: "Zitting 1: OS Interfaces"
has_children: true
has_toc: false
---

# User space programma bekijken

De broncode van user space programma's zoals `ls` en `cat` staat in de folder `user` in je Git-repository.

* Sluit de xv6-omgeving met <kbd>CTRL</kbd>+<kbd>A</kbd> <kbd>x</kbd>.
* Bekijk de code van het programma `cat`.

  ```console
  [ubuntu-shell]$ gedit user/cat.c
  ```

## Programma afsluiten

Merk op dat in xv6 een user space programma afgesloten wordt door de oproep ```exit(0);``` in plaats van te returnen uit main. Dit is het gevolg van het feit dat xv6 geen standaard C runtime implementeert. Een C runtime zoals [crt0](https://en.wikipedia.org/wiki/Crt0) is typisch verantwoordelijk voor het oproepen van de main-functie en na een return uit main het proces correct af te sluiten. Kijk [hier](https://stackoverflow.com/questions/3463551/what-is-the-difference-between-exit-and-return) voor meer informatie.
