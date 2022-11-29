---
layout: default
title: "Zitting 6: File Systems"
nav_order: 6
nav_exclude: false
search_exclude: false
has_children: true
has_toc: true
---
 
## Voorbereiding

Ter voorbereiding van deze zitting worden jullie verwacht:

* De oefenzitting [synchronizatie](../locks) te hebben voltoold;
* Hoofdstuk 8 van het [xv6 boek](https://github.com/besturingssystemen/xv6-riscv-book/releases/latest/download/book.pdf) te hebben te hebben gelezen.

## Introductie

Besturingssystemen werken met een geheugenhiërarchie.
Op het moment dat een OS actief is, zit er bijvoorbeeld code en data in het RAM, in caches en registers.
Al dit geheugen is echter vluchtig.
Wanneer je de stroom van de machine uittrekt, is al deze data plots weg.

Code en data die we willen bewaren voor een lange tijd, ook na het heropstarten van een machine, bewaren we op een opslagmedium.
Op dit opslagmedium staat de code van het besturingssysteem, alle data, de bestanden die een gebruiker op zijn machine heeft staan, enzovoort.

Er zijn vele soorten opslagmediums, elk met hun eigen capaciteit, snelheid en eventueel andere voor- en nadelen.
Al deze mediums kunnen gekoppeld worden aan een computer.
Zo heb je [Hard disk drives](https://en.wikipedia.org/wiki/Hard_disk_drive), [Solid-state drives](https://en.wikipedia.org/wiki/Solid-state_drive), [USB flash drives](https://en.wikipedia.org/wiki/USB_flash_drive), [Floppy disk drives](https://en.wikipedia.org/wiki/Floppy_disk), enzovoort.
Dit soort apparaten worden ook welk [*mass storage devices*](https://en.wikipedia.org/wiki/Mass_storage) genoemd.

> :information_source: Bij het opstart van een PC zal typisch de [BIOS](https://en.wikipedia.org/wiki/BIOS) de opstartcode (boot loader) lezen uit een drive en in het RAM-geheugen plaatsen.
> Vanuit RAM kan de processor deze code uitvoeren.
> Deze boot loader zal het RAM geheugen van de machine initialiseren via een [bootstrap-proces](https://en.wikipedia.org/wiki/Bootstrapping).
> Gedurende dit proces kunnen er eventueel ook nog andere nodige delen van het OS uit de drive gelezen worden.

Elk van deze mediums maken het mogelijk grote hoeveelheden data te bewaren.
Deze data moet georganiseerd worden.
Er moet een manier zijn om te weten welke data bij welk bestand hoort.
Deze organisatie gebeurt met behulp van *file systems*.
File systems kunnen op [vele verschillende manieren](https://en.wikipedia.org/wiki/Comparison_of_file_systems) geïmplementeerd worden.
In deze zitting zullen we dieper ingaan op het custom file system van xv6.

Het eerste deel van deze oefenzitting bekijkt het on-disk file system in detail.
In het tweede deel bekijken we hoe xv6 gebruik maakt van dit formaat om te werken met bestanden.
