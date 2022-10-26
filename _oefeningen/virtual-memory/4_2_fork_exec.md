---
layout: default
title: Fork en exec
nav_order: 9
parent: "Levenscyclus proces"
grand_parent: "Zitting 3: Virtual memory"
---

## fork en exec

In de [sessie over os interfaces](../../os-interfaces) hebben jullie in de permanente evaluatie ontdekt dat wanneer een proces geforked wordt, je plots twee processen hebt met *dezelfde* adressen maar toch mogelijks andere waarden op deze adressen.

* Verklaar dit aan de hand van je kennis over virtual memory
* Maak een programma dat `fork` gebruikt om een child proces te maken en vervolgens `vmprintmappings` oproept in parent en child.
  Verklaar de output.
  (Hint: gebruik `wait` in de parent om te wachten tot het child klaar is met uitvoeren om te voorkomen dat de outputs van `vmprintmappings` door elkaar geprint worden.)
* Bekijk nu het effect van `exec` op de mappings.
  Roep `vmprintmappings` voor de oproep naar `exec` en ook in het programma dat je met `exec` uitvoert.

> :information_source: Geïnteresseerden kunnen [hier](../extra/fork) een uitgebreide uitleg vinden over de xv6 implementatie van fork.


