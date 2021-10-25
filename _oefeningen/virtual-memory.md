---
layout: default
title: "Zitting 3: Virtual memory"
nav_order: 1
nav_exclude: true
search_exclude: true
has_children: true
has_toc: false
---

In deze sessie gaan we dieper in op het concept virtual memory. We kijken hoe paging werkt en hoe dit gebruikt wordt om virtual memory te implementeren. Vervolgens bekijken we enkele wijdverspreide toepassingen van virtual memory.

# Voorbereiding

Ter voorbereiding van deze oefenzitting word je verwacht:
  * De oefenzitting [system calls](../system-calls) te hebben voltooid.
  * Hoofdstuk 3 van het [xv6 boek](https://github.com/besturingssystemen/xv6-riscv-book/releases/latest/download/book.pdf) te hebben gelezen.
  * Begrip te hebben van de theorie rond *virtual memory*, *paging* en *page tables*
    * Weten hoe een *virtual address* vertaald kan worden naar een *physical address* via page tables


Onderstaande video kan je bekijken om meer vertrouwd te geraken met het concept paging:

[![Bekijk de video](https://img.youtube.com/vi/JgTXJ-ZV5Zw/hqdefault.jpg)](https://www.youtube.com/watch?v=JgTXJ-ZV5Zw)

Daarnaast kan je:
* Hoofdstuk 8 in Silberschatz raadplegen
* De online les over Hoofdstuk 8, deel 3 bekijken op Toledo
<!-- TODO online les h8 niet meer beschikbaar op toledo dit jaar? -->
