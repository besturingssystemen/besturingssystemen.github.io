---
layout: default
title: Structuur page table
nav_order: 4
parent: "Pagina mappen"
grand_parent: "Zitting 3: Virtual memory"
---

## Structuur page table

Page tables volgen een zeer specifieke structuur waarin elke bit een eigen betekenis heeft, zodat de RISC-V Memory Management Unit (MMU) deze efficiënt in hardware kan doorlopen.

### Bitvoorstellling PTE

Een page table kan je in het geheugen telkens terugvinden in het begin van een frame.
Elke page table bestaat uitsluitend uit 512 page table entries (PTE).
Een page table entry is niets anders dan een lange bitstring (64 bits) waarbij elke bit (of groep van bits) een eigen betekenis heeft.

Beantwoord de volgende vragen:

* We weten dat een page table 512 entries heeft en we weten dat elke entry 64 bit groot is. Hoe groot is een volledige page table?
* We weten dat we een offset van 12 bit gebruiken om een byte in een frame of pagina te addresseren. Hoe groot zijn pagina's of frames in Sv39?
* We weten dat een page table geplaatst wordt aan de start van een frame. Past een page table in één enkele frame?



![sv39 page table entry](../../../../img/sv39-pte.png)

Bovenstaande figuur geeft de bit-layout weer van zo'n page table entry.

> :information_source: Zoals je ziet worden bits genummerd van rechts naar links.  [Hier](https://stackoverflow.com/questions/23551187/why-bits-are-numbered-from-right-to-left) kan je een goede verklaring vinden. Dit is een conventie die zo goed als altijd toegepast wordt. De meest rechtse bit wordt ook vaak de LSB (least significant bit) genoemd, aangezien deze bit het minste gewicht heeft. De meest linkse bit wordt vaak de MSB (most significant bit) genoemd, deze heeft het meeste gewicht. 

Laten we de verschillende bits van een page table entry verder bespreken:

* `PPN`: Bits 10 - 53 zijn de 44 bits die naar een frame verwijzen. Ze bevatten dus het frame nummer (*physical page number*, PPN)
* `U`: Bit 4 is de user bit. Een pagina kan enkel in user-mode gebruikt worden indien de `U`-bit actief is.
* `X`: Bit 3 is de executable bit. Indien een pagina code bevat kan deze enkel uitgevoerd worden indien de `X`-bit van deze pagina actief is.
* `W`: Bit 2 is de writeable bit. Data kan enkel naar een pagina geschreven worden indien `W` actief is.
* `R`: Bit 1 is de readable bit. Data kan enkel van een pagina gelezen worden indien `R` actief is.
* `V`: Bit 0 is de valid bit. Enkel indien deze bit actief is, wordt de gehele page table entry als geldig beschouwd. Alle voorgaande bits hebben enkel een betekenis indien `V` actief is. Om een pagina te *unmappen* is het dus voldoende om `V` op 0 te zetten.

> :information_source: Enkele minder relevante bits voor de geïnteresseerden: 
> * `RSW`: Bits 8 - 9 mogen door een besturingssysteem gebruikt worden op een manier naar keuze. Ze worden door de MMU genegeerd. xv6 gebruikt deze bits niet.
> * `D`, `A`: Bit 7 is de dirty bit, bit 6 de accessed bit. De dirty bit houdt bij of een pagina gelezen, geschreven of opgehaald werd sinds de laatste keer dat de accesed bit op 0 werd gezet. Deze bits worden gebruikt om het cachen en swappen van pagina's efficiënter te laten verlopen. Voor meer infomatie verwijzen we jullie naar Silberschatz (8.4.1 Basic Page Replacement of zoek in je termen-index naar de dirty bit).
> * `G`: Bit 5 is de *global mapping* bit. Page tables worden per proces gealloceerd. Het is echter mogelijk om bepaalde page tables te delen tussen *alle* processen. Indien je de `G` bit in dat geval actief maakt kan de processor betere performantie leveren. Deze page tables zou je bijvoorbeeld permanent in een cache geladen kunnen houden.

### Access control

Bij de bespreking van de bitvoorstelling van een page table entry is duidelijk geworden dat paging niet uitsluitend gebruikt wordt om adressen te vertalen.
Het vertalingsproces wordt ook gebruikt om aan access control te doen.

Zo kan de kernel bepaalde pagina's ontoegankelijk maken voor een user-space proces, door de `U`-bit op 0 (inactief) te zetten.
Pagina's kunnen read-only gemaakt worden door `R` te activeren en `W` en `X` te deactiveren.

Onderstaande figuur geeft enkele mogelijke combinaties van `R`, `W` en `X` en hun betekenis:

![rwx encoding](../../../../img/rwx-encoding.png)


### Page faults

Wanneer we een virtueel adres of pagina aanspreken en ons niet houden aan de access control regels geëncodeerd in de page table entry, treedt een *page fault exception* op.

Een *exception* in hardware zorgt ervoor dat de huidige programma-executie onderbroken wordt.
De processor switcht van modus en springt naar een vast adres in de code van de kernel.
De code op dit adres noemen we de *trap handler*.

> :information_source: Merk op dat ook de `ecall`-instructie uit vorige oefenzitting ervoor zorgde dat er naar de *trap handler* gesprongen werd. Daarnaast is het ook mogelijk dat naar de trap handler gesprongen wordt als gevolg van een interrupt. De trap handler moet dus bepalen wat de reden is van de trap.

De meesten van jullie zullen onbewust al een exception hebben veroorzaakt, door een bug in jullie code.
xv6 zal in dat geval het falende proces meteen beëindigen.
Vervolgens print xv6, via de trap handler, de reden van de exception.
Hier enkele mogelijkheden:

> :information_source: De onderstaande tabel is enkel geldig indien een exception optreedt die in supervisor mode kan worden afgehandeld. Sommige exceptions worden afgehandeld in machine mode, daarvoor kan je een andere tabel vinden in de [RISC-V specificaties](https://riscv.org/technical/specifications/).

![Supervisor exception codes](../../../../img/supervisor-exception-codes.png)


Denk nu terug aan de eerste oefenzitting. 
Je schreef een hello world programma met een simpele return uit main.
`crt0` was nog niet toegevoegd, dus de `jr ra` instructie sprong naar een waarde in `ra` die nergens geïnitialiseerd was.
Stel dat je springt naar een willekeurig adres in de virtuele adresruimte van je proces.

  * Welke excepties uit bovenstaande tabel kunnen optreden? Merk op dat page faults niet de enige vorm van exceptions zijn.

Page faults kunnen ten slotte dus ook optreden op het moment dat de MMU een virtueel adres probeert te vertalen maar een bepaalde page table entry niet gevonden wordt.
De pagina is op dat moment dus niet gemapt.
Het besturingssysteem zou op dat moment kunnen beslissen om alsnog een pagina te mappen (zie *demand paging* in je boek).

  * Voer het onderstaande C-programma uit in xv6. Welke fout krijg je en waarom?

```c
 #include "user/user.h"
 #include "kernel/riscv.h"
 int main()
 {
     int* p = (int*) MAXVA;
     printf("%d", *p);
     return 0;
 } 
```
