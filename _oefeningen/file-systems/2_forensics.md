---
layout: default
title: "Oefening: Forensics"
nav_order: 2
parent: "Zitting 6: File Systems"
---

# Oefening: Forensics

Om de directories en files in een file system uit te lezen, hebben we geen OS nodig.
Begrip van de disk layout en interne structuren is voldoende om puur op basis van bytes in een image file te determineren welke bestanden of directories er op de disk staan.
Dit soort skills komen van pas bij data recovery, [forensics](https://en.wikipedia.org/wiki/Computer_forensics), etc... .

In deze oefening gaan we met behulp van hexdump (`hd`) een xv6 file system analyseren.
Download hiervoor eerst het bestand [forensics.img](../../../files/forensics.img).

1. Gebruik `hd` om de [`struct superblock`][superblock] te printen.

   >:bulb: De `-n` flag limiteert het aantal bytes in de output.
   Met `-n 1024` print je dus exact 1 disk block.
   Met `-s` kan je de bytes printen startend van een bepaalde byte offset.

2. Achterhaal het totaal aantal inodes in deze image en op welke disk block deze inodes starten.

3. Bereken de grootte van een [`struct dinode`][dinode].
   
4. Bekijk nu met `hd` de disk block waar de inodes starten. Kan je de root directory terugvinden? Op welke data block staat de root directory bewaard? Print deze data block met `hd` ter verificatie.

   > :bulb: De root directory is altijd opgeslagen in inode 1 in xv6.

5. De laatste entry in de root directory (hint: [`struct dirent`][dirent]) is opnieuw een folder.
   Wat is de naam van deze directory?
   Naar welke inode verwijst deze directory?
   
6. Gebruik `hd` om *enkel* de inode te printen van deze folder. Welke parameters gebruik je?

7. Gebruik `hd` om *enkel* de eerste data block te printen van deze folder. Welke parameters gebruik je?

8. Wat voor soort `inode` is de laatste entry in deze folder? Op welke datablok(ken) staat de inhoud? Hoe groot is de entry?

9. Gebruik `dd` om de file uit 7 en 8 te extracten uit het file system. Voer `man dd` uit om te kijken welke parameters je nodig  hebt. Hint: zet `bs=1`.

> **:question: *Optioneel maar leerrlijk, volg eerst de rest van de oefenzitting!* Automatiseer voorgaand proces. Schrijf een script dat recursief alle folders van een xv6 file system uitprint startend bij de root-folder, in een programmeertaal naar keuze.**

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

