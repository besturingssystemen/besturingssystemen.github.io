---
layout: default
title: "Achtergrond: On-disk File System"
nav_order: 1
parent: "Zitting 6: File Systems"
---

# Achtergrond: On-disk File System

## Block device

Traditionele opslagmedia kunnen opgedeeld worden in blokken van vaste grootte.
De blokken kan je vervolgens elk een adres toekennen.
De verschillende apparaten kunnen we dus abstract voorstellen als apparaten waarnaar je blokken geheugen kan lezen en schrijven.
Deze abstracte apparaten noemen we block devices.

xv6 neemt aan dat het file system bewaard wordt op zo'n block device.
De grootte van een blok is [in xv6 ingesteld op `BSIZE` bytes][block_size].
Het block device waarop xv6 zijn file system bewaart, en dat dus opgedeeld is in blokken van grootte `BSIZE`, wordt de *disk* genoemd.

> **:question: Wat is de grootte van een blok in xv6? Je kan dit best noteren, je zal dit later in de oefenzitting nodig hebben.**

> :information_source: Merk op dat deze *disk* een hard-drive, solid state drive, USB drive, ... zou kunnen zijn. In qemu emulator wordt een hard drive geëmuleerd op basis van een *image* file (voor xv6: `fs.img`). Deze image bevat de inhoud van de virtuele disk.

## Disk layout

We weten dus dat onze disk opgedeeld is in blokken van grootte `BSIZE`.
De data die je kan terugvinden in de verschillende blokken wordt vastgelegd in de disk layout.
Onderstaande afbeelding toont deze layout.

![xv6-disk-layout]({{ site.url }}/img/disk-layout-xv6.png)

De allereerste blok van de disk, blok 0, wordt gebruikt om de *boot code* te bewaren.
We noemen dit de [boot sector](https://en.wikipedia.org/wiki/Boot_sector).
Het is conventie om de boot sector te bewaren in de allereerste blok van een mass storage device.
Deze conventie laat ons toe om andere boot loaders te installeren of om de boot loader te bewerken, om zo verschillende soorten besturingssystemen te op te kunnen starten op éénzelfde machine, vanuit verschillende soorten mass storage devices.

> :bulb: De boot sector is nog geen deel van het file system.

> :bulb: De boot sector wordt niet gebruikt in xv6.

> :information_source: Mass storage devices kunnen opgedeeld worden in verschillende partities, elk met hun eigen bootgedeelte. Op sector 0 vinden we in dat geval de [Master Boot Record](https://en.wikipedia.org/wiki/Master_boot_record), met informatie over de verschillende partities op die specifieke schijf. Indien een partitie een OS bevat zal deze partitie vervolgens een [Volume Boot Record](https://en.wikipedia.org/wiki/Volume_boot_record) hebben met daarin de boot code van het OS op die partitie. Wanneer het xv6 boek of deze oefenzitting spreekt over een disk, moet je dit voorstellen als een partitie, niet als de volledige harde schijf. De boot sector van xv6 zou dus in de VBR zitten van een gepartitioneerde schijf.

### Superblock

Blok 1 is het eerste blok van het filesystem zelf.
In xv6 wordt dit blok het superblock genoemd.
Het superblock bevat meta-informatie over het filesystem.
Op basis van het superblock kan je de volledige disk layout van het file system achterhalen.

* Bekijk [`struct superblock`][superblock] in `kernel/fs.h`. Vergelijk met bovenstaande figuur waarin de lay-out van de disk wordt beschreven.

Het veld `magic` bevat een [magic number](https://en.wikipedia.org/wiki/Magic_number_(programming)), dat gebruikt wordt om het xv6 file system te identificeren. Het magic number van het xv6 file system bestaat uit de bytes `0x40 0x30 0x20 0x10`, ofwel het getal `0x10203040` voorgesteld met de [little-endian](https://en.wikipedia.org/wiki/Endianness) byte order.
Magic numbers in bestanden en file systems worden gebruikt om de verschillende soorten van elkaar te onderscheiden. Zo start bevoorbeeld elk [ELF-bestand](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format) met de bytes `0x7F 0x45 0x4c 0x46`, het magic number van ELF-files.

> **:question: Voer `make` uit in de xv6-repo. Voer vervolgens het commando `hd -s 1024 -n 1024 fs.img` uit.
> Dit commando zal het superblock in `fs.img` afprinten.
> Kan je het magic number van het xv6 filesystem terugvinden? Wat is het byte-adres van de eerste byte van dit magic number? Waarvan komt deze waarde?**

> :bulb: Het commando `hd` (_hex dump_) toont de inhoud van een bestand byte per byte.
> Elke lijn van de output heeft (standaard) het volgende formaat:
> ```ascii
> 00000410  1e 40 00 00 02 00 00 00  20 00 00 00 2d 00 00 00  |.@...... ...-...|
> \______/  \______________________________________________/  \________________/
>  offset        2x 8 bytes vanaf offset in het bestand      zelfde bytes in ASCII
> ```
> Alle waarden worden in hexadecimale notatie weergegeven.
> De kolom met ASCII waarden zal een `.` tonen als de overeenkomstige byte geen ASCII interpretatie heeft.
>
> Als er meerdere lijnen zouden zijn met enkel 0-bytes, wordt enkel de eerste weergeven gevolgd door een lijn met een `*`.
> De volgende getoonde lijn is de eerste die niet-0-bytes bevat.
>
> Via `-s` kunnen we de offset aangeven vanaf waar `hd` de inhoud van het bestand toont.
> `-n` geeft aan hoeveel bytes getoond moeten worden.
> Zoals met alle commando's, kan je meer informatie vinden in de _manual page_.
> Voer daarvoor het volgende commando uit in een Linux terminal: `man hd`.

Het tweede veld `size` bevat de grootte, uitgedrukt in aantal blokken, van het volledige file system image.

> **:question: Bereken de grootte in bytes van het file system op basis van het tweede veld `size` dat je kan vinden via `hd`. Voer vervolgens `ls -l` uit en vergelijk met de grootte van `fs.img`.**
>
> :bulb: Het type `uint` is telkens 4 bytes groot, de waarden staan opgeslagen in [little-endian byte order](https://en.wikipedia.org/wiki/Endianness).

Vervolgens worden `nblocks`, `ninodes` en `nlog` bewaard, die respectievelijk het aantal data blokken, inodes en log blokken weergeven.
Ten slotte bevatten `logstart`, `inodestart` en `bmapstart` de blok-adressen van het eerste blok van respectievelijk de `log`, de inodes en de `bitmap` secties.
We leggen in de komende secties uit wat de `log`, inodes en `bitmap` secties net voorstellen.

## File tree

Files in file systems worden typisch georganiseerd door middel van een [boomstructuur](https://en.wikipedia.org/wiki/Tree_(data_structure)) van directories (folders/mappen) en bestanden.
Elke directory kan zowel subdirectories als bestanden bevatten.
De root directory (in UNIX-based systemen wordt deze directory `/` genoemd) is het startpunt, de buitenste folder.

In UNIX systemen worden bepaalde *device drivers* ook voorgesteld door middel van speciale bestanden in het file system.
Schrijfoperaties of leesoperaties naar deze bestanden worden doorgestuurd naar de overeenkomstige *driver*.
Zo is het eenvoudig om dezelfde I/O interface te gebruiken voor devices als voor files.
Dit zijn dus geen normale bestanden.
Ze worden vaak [device files](https://en.wikipedia.org/wiki/Device_file) of special files genoemd.

Elke *node* van de boomstructuur kan dus een bestand, directory, of een special file zijn.
Deze *nodes* worden in xv6 en andere UNIX-based file systems voorgesteld door [inodes](https://en.wikipedia.org/wiki/Inode).

## Disk Inodes

Een *inode* stelt abstract een node voor in de boomstructuur van het file system.
Op de disk worden alle inodes van een file system bewaard.

Deze worden in xv6 voorgesteld door een [`struct dinode`][dinode] (disk inode).
Een `struct dinode` heeft een `type`, dat kan verwijzen naar een bestand, directory of special file.
De grootte in bytes van een bestand wordt bewaard in `size`.
De inhoud van het bestand wordt bewaard op één of meerdere blokken van de disk.

> :bulb: Het type `short` is twee bytes groot in xv6.

![disk-inode]({{ site.url }}/img/dinode.png)

De `addrs` array bevat de diskadressen van deze verschillende diskblokken.
De adressen verwijzen naar een bloknummer in de datasectie van de disk (zie disk layout).

De grootte van de adresarray is `NDIRECT + 1`.
Er zijn dus `NDIRECT` mogelijke disk blokken die data van het bestand kunnen bevatten.
Een bestand kan bijgevolg al zeker grootte `NDIRECT * BSIZE` hebben.

Op locatie `addrs[NDIRECT]` bevindt zich echter ook nog het adres van een speciaal disk blok.
In dit disk blok staan nog eens `NINDIRECT` verschillende disk adressen.
Ook deze `NINDIRECT` blokken kunnen data van de inode bevatten.
Een inode kan dus maximaal `NDIRECT + NINDIRECT` blokken bevatten.
De adressen van de eerste `NDIRECT` datablokken kan je terugvinden in de `struct dinode`, die van de laatste `NINDIRECT` blokken kan je terugvinden in het blok op adres `addrs[NDIRECT]`.

De dinodes worden op disk opgeslagen in de _inodes_ sectie beginnende bij het block aangegeven met `inodestart` in het superblock.
De blokken in de inode sectie worden geïnterpreteerd als een array van `dinode` structs.
Als er verwezen wordt naar een dinode, zal de gebeuren met een index in deze array.

## Bestanden

Een bestand is een inode met het type `T_FILE`.
De data van een bestand wordt bewaard in de datablok(ken) van de inode.

> **:question: Wat is de maximale bestandsgrootte in xv6? Hoe zou je xv6 kunnen aanpassen om bestanden van arbitraire grootte toe te laten?**

## Device

Een device is een inode met het type `T_DEVICE`.

De velden `major` en `minor` in de `struct dinode` worden gebruikt voor device inodes.
Drivers van devices worden in het OS geassocieerd met een nummer.
Het veld `major` staat voor *device major number* en verwijst naar een specifieke driver.
In xv6 wordt de major number van de console/UART driver [in `kernel/file.h` gedefinieerd][console].

Een device inode met major number 1 verwijst dus naar de console driver.
Reads en writes van en naar deze inode zullen doorgestuurd worden naar deze driver.
Indien een driver meerdere manieren/modes heeft om reads/writes af te handelen, kan een specifieke modus gespecifieerd worden met het het veld `minor`, ofwel het *device minor number*.
Dit veld is driverafhankelijk.

## Directory

Een directory (folder) is een inode met het type `T_DIR`.
De data van een directory inode bevat een array van *directory entries*.
Elke entry bevat een naam en verwijst naar een inode op hetzelfde block device.
Elk van deze entries kunnen dus opnieuw bestanden zijn, device files/special files of andere folders.
Een directory entry wordt voorgesteld door een [`struct dirent`][dirent].

> :information_source: De verschillende inode types (`T_FILE`, `T_DEVICE` en `T_DIR`) staan gedefinieerd in [`kernel/stat.h`][stat]. Ze komen allen overeen met een vaste numerieke waarde.

> **:question: Stel dat je een directory hebt met exact 1 gealloceerde data block in de inode. Wat is het maximaal aantal directory entries dat deze directory kan bevatten?**

> **:question: Wat is het effectieve maximale aantal entries dat een directory in xv6 kan bevatten?**


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
