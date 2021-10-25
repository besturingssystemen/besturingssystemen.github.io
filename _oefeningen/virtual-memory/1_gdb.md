---
layout: default
title: Debuggen met GDB
nav_order: 1
parent: "Zitting 3: Virtual memory"
---

# Debuggen met GDB

Het programma `gdb`, de GNU debugger, kan gebruikt worden om stap voor stap de machinecode van een gecompileerd programma uit te voeren. 
Helaas is er heeft Ubuntu standaard geen packages voor de RISC-V versie van GDB en zullen we deze handmatig moeten installeren.
Zie [hier](../../../tutorials/gdb) voor uitgebreide installatie instructies.

> :warning: We geven deze informatie ter referentie mee, voor als je `gdb` later nodig zou hebben om een concreet probleem te debuggen. Begin nu niet meteen tijdens te oefenzitting kostbare tijd te besteden met `gdb` installeren en uit te proberen.
