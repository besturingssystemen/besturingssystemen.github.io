---
layout: default
title: "Achtergrond: Runtime File System"
nav_order: 3
parent: "Zitting 6: File Systems"
---

# Runtime file-system

In het eerste deel van de sessie hebben we ons voornamelijk geconcentreerd op de voorstelling en layout van bestanden en directories op de disk.
Er komt echter veel kijken bij het gebruik van dit soort on-disk file system at runtime, door een OS.
In dit deel zullen we ons vooral bezig houden met de runtime informatie die nodig is om efficiënt te werken met het file system.

Er zijn nog twee on-disk secties die we niet besproken hebben: de `log` en  `bitmap` sectie.
Deze hebben voornamelijk betrekking tot de runtime structuur van het file system en worden daarom pas geïntroduceerd in deze sectie.

## Disk space management

Herinner je dat de [`kalloc`][kalloc] en [`kfree`][kfree] functies gebruikt werden om fysiek geheugen te beheren.
Vrije frames worden in een gelinkte lijst bewaard, bij allocatie wordt een frame uit deze lijst gehaald en gealloceerd.

Zoals fysiek geheugen opgedeeld is in frames, zo wordt diskgeheugen opgedeeld in blokken.
Om blokken diskgeheugen te alloceren hebben we gelijkaardige functies [`balloc`][balloc] en [`bfree`][bfree].

### Bitmap

`balloc` en `bfree` werken echter niet met een gelinkte lijst om vrije blokken te beheren.
In de plaats daarvan wordt een [*bitmap*](https://en.wikipedia.org/wiki/Bit_array) gebruikt.

De bitmap bestaat uit één (of meerdere) blokken geheugen op de disk.
Elke bit van deze blokken verwijst naar een specifiek diskblok.
Indien de bit van een blok in de bitmap `1` is, is deze diskblok gealloceerd.
Indien de bit van een blok in de bitmap `0` is, is deze diskblok vrij om gebruikt te worden.

Om te kijken of een blok vrij is moet je dus kijken in de bitmap naar de bit die verwijst naar dit blok.
Indien de bit `0` is, is de blok vrij.
Om de blok te alloceren moet je deze bit op `1` zetten.

> **:question: Waarom maakt xv6 gebruik van een bitmap voor `balloc` en een gelinkte lijst voor `kalloc`? Zou een bitmap voor `kalloc` ook mogelijk zijn? Zou een gelinkte lijst voor `balloc` mogelijk zijn? Kan je voor- of nadelen bedenken? Zou jij dezelfde keuze maken als xv6?**

## Performantie

Lezen en schrijven naar mass storage kan erg traag zijn.
Indien je bijvoorbeeld gebruik maakt van een harde schijf, zal deze schijf eerst fysiek moeten roteren en eventueel van track verwisselen om de juiste sector te selecteren.
Dat kan allemaal lang duren.
Hoewel een Solid State Drive al een stuk sneller is, is een leesoperatie uit een SSD schijf nog steeds enkele grootte-ordes trager dan een leesoperatie uit RAM geheugen.

> :information_source: De tijd tussen een leesoperatie en het moment dat de data beschikbaar is vanuit RAM geheugen [wordt gemeten in nanoseconden](https://en.wikipedia.org/wiki/CAS_latency).
> Voor een SSD [wordt dit gemeten in microseconden](https://blocksandfiles.com/2020/09/08/seven-attempts-to-speed-processing-with-faster-storage/) (1 microseconde = 1000 nanoseconden). 
> Voor een HDD [gaat dit zelfs over miliseconden](https://en.wikipedia.org/wiki/Hard_disk_drive_performance_characteristics) (1 miliseconde = 1000 microseconden).

### Buffer cache

Om ervoor te zorgen dat lees- en schrijfoperaties naar bestanden veel sneller kunnen verlopen, maakt het file system van xv6 gebruik van een [*cache*](https://en.wikipedia.org/wiki/Cache_(computing)).
Wanneer een geheugenblok gelezen wordt, wordt het in het RAM-geheugen geplaatst, in de *buffer cache*.
Vanaf dan kunnen alle lees- en schrijfoperaties rechtstreeks via het RAM-geheugen verlopen, in plaats van via de harde schijf.

De buffer cache wordt voorgesteld door een *doubly linked list* van buffers (gecachte blokken), gesorteerd zodat de *least recently used* buffer achteraan (`head->prev`) in de lijst staat en de *most recently used* buffer dus vooraan (`head->next`) staat.

* Bekijk de functie [`bget`][bget] in `kernel/bio.c`. Deze functie wordt gebruikt door [`bread`][bread] en [`bwrite`][bwrite] om een buffer overeenkomstig met een gegeven bloknummer op te vragen. Indien de blok niet gecacht is wordt er gezocht naar een ongebruikte buffer in de buffer cache en wordt deze buffer toegewezen aan een specifieke blok.
* Bekijk de functies [`bread`][bread] en [`bwrite`][bwrite]. `bread` vraagt aan `bget` een buffer voor een specifiek bloknummer. Indien deze buffer zonet door `bget` hergebruikt werd bevatte deze nog foute data van een andere blok (`b->valid == 0`), en wordt de juiste blokdata van de disk gelezen. `bwrite` zorgt ervoor dat de data in een buffer naar de correcte blok in de mass storage wordt geschreven.

Het gevolg van het invoeren van een cachelaag is dat lees- en schrijfoperaties niet meer rechtstreeks naar de harde schijf worden gestuurd.
Pas wanneer buffers expliciet weggeschreven worden met `bwrite` zal een aanpassing van de inhoud van een blok effectief bewaard worden in de long term storage.

## Consistency

### Transaction log

Een besturingssysteem kan op ieder moment crashen, ter gevolg van een fout in de code of gewoon door stroom die wegvalt.
Stel dat je echter besturingssysteemcode hebt die een gegevensstructuur aanpast op de disk, bijvoorbeeld code die een bestand uit het file system verwijderd
Deze operatie zal bestaan uit meerdere schrijfoperaties.

Stel dat schrijfoperatie *W<sub>a</sub>* het bestand uit de directorystructuur haalt, en schrijfoperatie *W<sub>b</sub>* de gealloceerde file system blokken als ongebruikt markeert in de bitmap.
Indien enkel *W<sub>a</sub>* uitgevoerd zou worden, zou een bestand dat niet in de directorystructuur zit nog steeds plaats innemen op de disk.
Je zou dus een memory leak hebben.
Indien enkel *W<sub>b</sub>* uitgevoerd zou worden, zou een bestand in de directory verwijzen naar niet-gealloceerde blokken uit het file system, die eventueel later aan een ander bestand zouden gekoppeld worden.

Sommige operaties bestaan dus uit verschillende schrijfoperaties, die allemaal uitgevoerd moeten worden, om te vermijden dat het file system in een inconsistente staat terecht komt.
xv6 lost deze uitdaging op door middel van [transacties](https://en.wikipedia.org/wiki/Transaction_processing).
Een transactie groepeert verschillende schrijfoperaties.
Er wordt gegarandeerd dat ofwel alle schrijfoperaties in een transactie uitgevoerd worden, ofwel geen enkele.
Dit wordt gegarandeerd via een *transaction log*.

Alle gegroepeerde schrijfoperaties in een transactie worden eerst geschreven naar een aparte regio van de disk, de log.
Wanneer alle schrijfoperaties binnen een transactie naar de log geschreven zijn, kan je zeker zijn dat zelfs wanneer op dat moment een crash voorvalt, de geschreven data nog steeds beschikbaar is.
Alles staat namelijk in de log op de disk.
Op dat moment zijn de geschreven blokken op de disk echter nog niet gewijzigd.

Wanneer alle schrijfoperaties in de log staan, kan de transactie *gecommit* worden.
De operaties kunnen vanuit de log naar de juiste plaats op de disk geschreven worden.
Een crash kan er hoogstens voor zorgen dat het kopiëren van de log naar de juiste disk sectoren maar gedeeltelijk uitgevoerd wordt.
Bij het opnieuw opstarten vanuit een crash, kunnen al deze schrijfoperaties echter gewoon opnieuw hervat worden, want ze staan nog steeds klaar in de log.
Er is dus geen data verloren gegaan.
Transacties garanderen dat alle schrijfoperaties in één transactie als een geheel worden uitgevoerd.

Indien de machine crashte op een moment dat de transactie nog niet volledig klaarstond in de log, wordt de transactie gewoon nooit uitgevoerd.
Op dat moment kan het namelijk zijn dat het committen van deze incomplete transactie ervoor zou zorgen dat het file system in een inconsistente state terecht komt.
Een transactie wordt dus ofwel in zijn volledigheid uitgevoerd, ofwel helemaal niet.

De functies [`begin_op`][begin_op] en [`end_op`][end_op] worden gebruikt om een transactie te starten en te beëindigen.
Je zal deze operaties terugvinden in verschillende system calls die gebruik maken van het file system.

> **:question: Bekijk de functie [`log_write`][log_write].
> Deze wordt gebruikt om een bewerkte buffer vanuit de cache naar de schijf te schrijven.
> De functie [`bwrite`][bwrite] wordt echter ook gebruikt om dezelfde reden.
> Wat is het verschil tussen deze functies?
> Wanneer gebruik je [`bwrite`][bwrite] en wanneer gebruik je [`log_write`][log_write]?
> De implementatie van [`write_log`][write_log] kan misschien verhelderend werken.**

## Memory inodes

Een [`struct dinode`][dinode] is de on-disk representatie van een inode.
Wanneer we een dinode willen bewerken, zullen we deze in-memory voorstellen met behulp van de [`struct inode`][inode].
De in-memory representatie heeft extra velden, die niet permanent op een disk bewaard moeten worden.

> :information_source: Wanneer we vanaf nu spreken over dinode bedoelen we de specifieke on-disk representatie van een inode.
> Een dinode is dus een structuur op een bepaalde disk.
> Een inode is de in-memory kopie.
> Het is mogelijk dat er meerdere disks zijn, met meerdere on-disk file systems, elk met hun eigen dinodes.
> inodes kunnen dus naar dinodes van verschillende disks verwijzen.

<!--- Het veld `dev` geeft aan op welke block device de in-memory inode opgeslagen is.
Indien je meerdere disks hebt is het belangrijk bij te houden op welke disk de overeenkomstige dinode bewaard werd. -->

Een on-disk xv6 filesystem bevat een vast aantal dinodes.
Dit aantal wordt bewaard in de superblock `sb` in het veld `sb->ninodes`.
Deze dinodes bevinden zich in een array op de disk, startend van blok `sb->inodestart`.

Een unieke inode wordt geïdentificeerd door het paar (`dev`, `inum`).
De waarde `dev` is de id van een block device (bvb een disk), `inum` is de index in de dinode-array van dat block device.
Voor een specifieke [`struct inode`][inode] identificeren de velden `dev` en `inum` dus samen de exacte blok op de gegeven disk/block device waar de overeenkomstige dinode bewaard is.

Om een nieuwe inode aan te maken op een block device `dev`, wordt gebruik gemaakt van de functie [`ialloc`][ialloc].
Deze functie kijkt of er nog niet-gealloceerde dinodes (`dinode->type == 0`) beschikbaar zijn op de disk.
Indien een niet-gealloceerde dinode gevonden wordt, wordt deze geïnitialiseerd met het gegeven type.

> **:question: Waarom moet [`ialloc`][ialloc] geen gebruik maken van [`balloc`][balloc] om een nieuwe inode aan te maken?**

## Inode cache

Pointers naar actieve inodes (*in xv6-code: `ip` of inode pointer*) worden in-memory bewaard in een array genaamd de inode cache (`icache`).
De functie [`iget`][iget] kijkt of een specifieke dinode reeds in de cache aanwezig is.
Zo ja, wordt de bestaande inode pointer teruggegeven.
Zo niet, wordt een ongebruikte inode pointer (`ip->ref == 0`) gereturned.

<!-- TODO Waarom is de inode cache een fixed size array??
Zou het voor meerdere disks niet logischer zijn om een linked list te maken? Lijkt me een artificiële limitatie -->

Het veld `ip->valid` geeft aan of de inode reeds gelezen is van de disk.
De functie `iget` returnt namelijk enkel een inode pointer uit de cache,
maar garandeert niet dat de inode reeds de informatie van de overeenkomstige dinode heeft uitgelezen.

De functie [`ilock`][ilock] moet gebruikt worden om een lock te verkrijgen vooraleer er gelezen of geschreven mag worden naar een inode.
[`ilock`][ilock] zal, naast het verkrijgen van een lock, ook controleren of een inode al dan niet valid (gelezen van disk) is.
Indien dit niet het geval is, zal hier de data van de dinode uitgelezen worden met behulp van [`bread`][bread].

Het is mogelijk dat dezelfde inode pointer uit de inode cache door meerdere stukken code in gebruik is op hetzelfde moment.
Het veld `inode->ref` bewaart het aantal open referenties naar een specifieke inode.
Wanneer een inode opgevraagd wordt met `iget` wordt deze waarde verhoogd met 1.
Op het moment dat een stuk code een inode niet meer nodig heeft, wordt [`iput`][iput] opgeroepen.
In `iput` wordt `inode->ref` verlaagd met 1.
Indien de laatste referentie via `iput` wordt vrijgegeven, wordt de inode pointer terug vrijgemaakt in de cache zodat deze hergebruikt kan worden om te verwijzen naar een andere dinode.

## Bewerken inode

Via een inode pointer kan de inhoud van een inode bewerkt worden.
Met behulp van [`itrunc`][itrunc] wordt de inhoud van een bepaalde inode volledig gewist.
[`stati`][stati] wordt gebruikt om de meta-informatie van een inode op te vragen.
[`readi`][readi] en [`writei`][writei] worden gebruikt om respectievelijk te lezen en schrijven naar de datablokken van een inode.

> **:question: Bekijk de code van [`readi`][readi] en [`writei`][writei] aandachtig en probeer te begrijpen hoe deze functies werken.
> Zal een `writei`-operatie meteen naar de disk schrijven?
> Zo niet, wanneer wordt de geschreven informatie dan wel effectief naar de disk geschreven?**

## Directories

De functie [`dirlookup`][dirlookup] zoekt een directory entry in een gegeven directory.
Dit gebeurt door met `readi` de inhoud van de directory inode te lezen, entry per entry, tot de gezochte naam teruggevonden wordt.
Het resultaat is de inode waarnaar de directory entry verwijst.
Met [`dirlink`][dirlink] voeg je een inode toe aan een directory.
Bij aanmaak van een directory wordt standaard de entry `.`, verwijzend naar zichzelf, en de entry `..`, verwijzend naar de parent directory inode, toegevoegd.

Absolute paden in directories worden in UNIX voorgesteld als volgt:
`/dirx/diry/dirz/filename`.
De directory `/` is de root directory, met subdirectory `dirx`.
`dirx` heeft als subdirectory `diry`, enzovoort.
Relatieve paden zijn paden die niet starten met `/` en worden relatief van de huidige *working directory* van een proces geresolved.

De functie [`namex`][namex] neemt een absoluut of relatief pad en geeft als resultaat de inode terug waarnaar dit pad verwijst.

## File abstractie in UNIX

<!-- TODO code links -->

Zoals we ondertussen weten worden bestanden voorgesteld door inodes en bewaard in directories die ook in inodes bewaard worden.
Dat is de essentie van het file system en deze structuren worden bewaard in `fs.img`, de image file van het file system.

UNIX gebruikt de file-abstractie echter ook op een tweede manier.
In UNIX-gebaseerde systemen wordt de `struct file` gedefinieerd.
Een `struct file` kan verwijzen naar een inode met daarin een bestand of een device, maar kan ook verwijzen naar een pipe.
Naar elk van deze entiteiten kan je lezen of schrijven.
Om programmeren eenvoudiger te maken, kan je devices of pipes dus openen als files.
Je kan hierdoor dezelfde functies die gebruikt worden om te lezen of the schrijven naar bestanden, ook gebruiken om te lezen of te schrijven naar pipes of device drivers.

De system call `open` wordt gebruikt om een bestand te openen (of aan te maken) in/uit het file system.
Open bestanden bevinden zich in de `ftable`, een array van `struct file`.
Het maximaal aantal open bestanden in xv6 is gelijk aan `NFILE`.
Via `filealloc` wordt een lege entry gezocht in de `ftable` en deze toegewezen.
`fopen` zal `filealloc` dus gebruiken om een verwijzing te maken naar een nieuwe open bestand.

De functie `pipealloc` uit `kernel/pipe.c` maakt een pipe aan, en gebruikt daarvoor ook `filealloc`.
Een pipe is dus ook een entry in de file table.
Om een pipe te implementeren wordt er echter gewoon een frame gealloceerd met `kalloc`.
Dit heeft niets met het onderliggende file system te maken.

Een proces kan een verwijzing hebben naar een open bestand in de `ftable`.
Elk proces heeft een array van open files.
Dit is een array van *file pointers* die wijzen naar de `ftable`.
Een index in deze array noemen we een *file descriptor*.
Met `fdalloc` wordt een ongebruikte file descriptor toegewezen aan een `struct file *`.
Het is dus mogelijk dat meerdere processen via file descriptors verwijzen naar dezelfde geopende bestanden.
Merk op dat dit bijvoorbeeld zal gebeuren na het oproepen van `fork` indien het geforkte proces open file descriptors had.

Schrijven naar een device, pipe, of bestand gebeurt via `filewrite`.
Indien de file een pipe is wordt de schrijfoperatie doorgestuurd naar `pipewrite`.
Indien de file een device is, zal gezocht worden naar een function pointer die een specifieke schrijffunctie bevat voor dat device.
Indien het een gewoon bestand is, zal `writei` gebruikt worden om naar de datablokken van de inode van het bestand te schrijven.

In `user/init.c` wordt met behulp van de functie `mknod` een device file aangemaakt met de naam console en device major number `CONSOLE`.
In de kernel wordt dit device major number in `consoleinit()` gelinkt aan de functies `consoleread` en `consolewrite`.
De file descriptor `0` van het init proces verwijst dus naar de device inode met de naam console.
Door tweemaal `dup(0)` op te roepen verwijzen ook file descriptor `1` en `2` naar de console.
Aangezien `init` het root user process is in Linux, zullen alle processen geforked uit `init` deze configuratie overnemen, tenzij ze manueel de file descriptors overschrijven.
File descriptor `0` wordt typisch de `stdin` genoemd, `1` de `stdout` en `2` de `stderr`.
Standaard verwijzen deze dus naar de `console`, maar je kan deze (bvb via `>` en `<` in een shell) laten doorverwijzen naar bestanden.

> :warning: Er is een belangrijk verschil tussen de `struct file` en de `struct FILE`.
> De eerste is de structuur, gebruikt door de kernel, om files voor te stellen.
> Deze kan niet gebruikt worden vanuit user code.
> De `struct FILE` daarentegen is een deel van de C standard library, staat los van het operating system en is uitsluitend bedoeld voor user code.
> Functies zoals `fopen` en `fwrite` werken met de `struct FILE` en gebruiken intern de system calls `open` en `write` die werken op basis van de file descriptors bewaard in de `struct FILE`.
> In xv6 staat de `struct file` gedefinieerd in `kernel/file.h`.
> De xv6 C standard library heeft geen `struct FILE`.

> **:question:Probeer abstract uit te leggen wat er in xv6 gebeurt op het moment dat je vanuit user-space naar een open bestand probeert te schrijven, van system call tot effectieve schrijfoperatie naar de harde schijf.**

<!-- TODO file descriptors uitleggen -->

<!-- TODO dinode vs inode verwarring anders aanpakken -->

<!-- TODO openen, schrijven naar een bestand -->


[block_size]:https://github.com/besturingssystemen/xv6-riscv/blob/02ca399d0590a57d9ba05fcf556546141a5e2a09/kernel/fs.h#L11
[superblock]:https://github.com/besturingssystemen/xv6-riscv/blob/02ca399d0590a57d9ba05fcf556546141a5e2a09/kernel/fs.h#L19
[bget]:https://github.com/besturingssystemen/xv6-riscv/blob/028af2764622d489583cd88935cd1d2a7fbe8248/kernel/bio.c#L55
[bread]:https://github.com/besturingssystemen/xv6-riscv/blob/028af2764622d489583cd88935cd1d2a7fbe8248/kernel/bio.c#L91
[bwrite]:https://github.com/besturingssystemen/xv6-riscv/blob/028af2764622d489583cd88935cd1d2a7fbe8248/kernel/bio.c#L105
[begin_op]:https://github.com/besturingssystemen/xv6-riscv/blob/2501560cd691fcdb9c310dccc14ac4e7486c99d9/kernel/log.c#L124
[end_op]:https://github.com/besturingssystemen/xv6-riscv/blob/2501560cd691fcdb9c310dccc14ac4e7486c99d9/kernel/log.c#L143
[log_write]:https://github.com/besturingssystemen/xv6-riscv/blob/2501560cd691fcdb9c310dccc14ac4e7486c99d9/kernel/log.c#L204
[write_log]:https://github.com/besturingssystemen/xv6-riscv/blob/2501560cd691fcdb9c310dccc14ac4e7486c99d9/kernel/log.c#L176
[kalloc]: https://github.com/besturingssystemen/xv6-riscv/blob/85bfd9e71f6d0dc951ebd602e868880dedbe1688/kernel/kalloc.c#L65
[kfree]: https://github.com/besturingssystemen/xv6-riscv/blob/85bfd9e71f6d0dc951ebd602e868880dedbe1688/kernel/kalloc.c#L42
[balloc]: https://github.com/besturingssystemen/xv6-riscv/blob/675060882480c21915629750a5a504d9da445ba3/kernel/fs.c#L63
[bfree]: https://github.com/besturingssystemen/xv6-riscv/blob/675060882480c21915629750a5a504d9da445ba3/kernel/fs.c#L89
[dinode]:https://github.com/besturingssystemen/xv6-riscv/blob/02ca399d0590a57d9ba05fcf556546141a5e2a09/kernel/fs.h#L36
[inode]:https://github.com/besturingssystemen/xv6-riscv/blob/2b5934300a404514ee8bb2f91731cd7ec17ea61c/kernel/file.h#L23

[ialloc]: https://github.com/besturingssystemen/xv6-riscv/blob/675060882480c21915629750a5a504d9da445ba3/kernel/fs.c#L192
[iput]: https://github.com/besturingssystemen/xv6-riscv/blob/675060882480c21915629750a5a504d9da445ba3/kernel/fs.c#L325
[iget]: https://github.com/besturingssystemen/xv6-riscv/blob/675060882480c21915629750a5a504d9da445ba3/kernel/fs.c#L239
[ilock]: https://github.com/besturingssystemen/xv6-riscv/blob/675060882480c21915629750a5a504d9da445ba3/kernel/fs.c#L286

[itrunc]: https://github.com/besturingssystemen/xv6-riscv/blob/675060882480c21915629750a5a504d9da445ba3/kernel/fs.c#L407
[stati]: https://github.com/besturingssystemen/xv6-riscv/blob/675060882480c21915629750a5a504d9da445ba3/kernel/fs.c#L439
[readi]: https://github.com/besturingssystemen/xv6-riscv/blob/675060882480c21915629750a5a504d9da445ba3/kernel/fs.c#L451
[writei]: https://github.com/besturingssystemen/xv6-riscv/blob/675060882480c21915629750a5a504d9da445ba3/kernel/fs.c#L479

[dirent]: https://github.com/besturingssystemen/xv6-riscv/blob/02ca399d0590a57d9ba05fcf556546141a5e2a09/kernel/fs.h#L61
[dirlookup]: https://github.com/besturingssystemen/xv6-riscv/blob/675060882480c21915629750a5a504d9da445ba3/kernel/fs.c#L526
[dirlink]: https://github.com/besturingssystemen/xv6-riscv/blob/675060882480c21915629750a5a504d9da445ba3/kernel/fs.c#L554
[namex]: https://github.com/besturingssystemen/xv6-riscv/blob/675060882480c21915629750a5a504d9da445ba3/kernel/fs.c#L623

[stat]: https://github.com/besturingssystemen/xv6-riscv/blob/2b5934300a404514ee8bb2f91731cd7ec17ea61c/kernel/stat.h#L6

[console]:https://github.com/besturingssystemen/xv6-riscv/blob/2b5934300a404514ee8bb2f91731cd7ec17ea61c/kernel/file.h#L47

[xv6 repo]: https://github.com/besturingssystemen/xv6-riscv
