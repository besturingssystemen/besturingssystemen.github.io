---
layout: default
title: "Oefening: Proces mappings veiliger maken"
nav_order: 9
parent: "Levenscyclus proces"
grand_parent: "Zitting 3: Virtual memory"
---

## Oefening: Proces mappings veiliger maken

In sectie 3.8 van het xv6 boek wordt uitgelegd hoe `exec` secties uit een ELF file in het geheugen laadt via [`uvmalloc`][uvmalloc]. 

> :information_source: Een gedetailleerde uitleg over ELF-files kan je ook [hier](../extra/elf) terugvinden.

We gaan dit nu wat meer in detail bekijken via het volgende programma (in `hello.c`):

```c
#include "kernel/types.h"
#include "user/user.h"

int message_id = 42;
const char* message = "Hello, world";

void print_message()
{
  printf("[%d] %s\n", message_id, message);
}

int main()
{
  vmprintmappings();

  print_message();

  return 0;
}
```

Laten we eens kijken wat de secties zijn in het gecompileerde ELF bestand:
<pre>
[ubuntu-shell]$ riscv64-linux-gnu-readelf -l user/_hello

Elf file type is EXEC (Executable file)
Entry point 0x83a
There are 2 program headers, starting at offset 64

Program Headers:
  Type           Offset             VirtAddr           PhysAddr
                 FileSiz            MemSiz              Flags  Align
  LOAD           0x0000000000001000 0x0000000000000000 0x0000000000000000
                 0x000000000000089a 0x000000000000089a  R E    0x1000
  LOAD           0x0000000000002000 0x0000000000001000 0x0000000000001000
                 0x000000000000000c 0x0000000000000028  RW     0x1000

 Section to Segment mapping:
  Segment Sections...
   00     .text .rodata
   01     .sdata .sbss .bss
</pre>

Je kan zien dat er twee secties zijn die in het geheugen geladen moeten worden (type `LOAD`):
- Op virtueel adres `0x0` (`VirtAddr`), een sectie die `0x89a` bytes groot is in het geheugen (`MemSiz`).
  De permissies op deze sectie (`Flags`) zijn read (`R`) en execute (`E`).
  Dit is de sectie die de code en constanten zoals string literals bevat.
- Op virtueel adres `0x1000`, een read-write (`RW`) sectie die `0x28` bytes groot is.
  Dit is de data sectie die globale variabelen bevat.

En laten we dan eens kijken hoe deze executable in het geheugen geladen wordt:
<pre>
$ make qemu
...
$ hello
0x0000000000000000 -> 0x0000000087f43000, mode=U, perms=rwx
0x0000000000001000 -> 0x0000000087f40000, mode=U, perms=rwx
0x0000000000002000 -> 0x0000000087f3f000, mode=S, perms=rwx
0x0000000000003000 -> 0x0000000087f3e000, mode=U, perms=rwx
0x0000003fffffe000 -> 0x0000000087f76000, mode=S, perms=rw-
0x0000003ffffff000 -> 0x0000000080007000, mode=S, perms=r-x
[42] Hello, world
</pre>

De eerste twee pagina's komen overeen met de twee secties in het ELF bestand.
Echter, de permissies komen _niet_ overeen!
Beide pagina's hebben volledige read-write-execute rechten.

Alhoewel dit werkt, is het niet optimaal vanuit een security oogpunt.
Zo is het bijvoorbeeld mogelijk om de code te overschrijven, iets wat bij de meeste programma's niet de bedoeling is en meestal wijst op een bug.
Om dit eens uit te testen, kan je het volgende stukje code toevoegen _voor_ de oproep van `print_message`:
```c
*(uint16*)print_message = 0x8082;
```

Deze code zal de waarde `0x8082` (de hexadecimale voorstelling van de `jr ra` (`ret`) instructie in RISC-V) naar het adres van de `print_message` functie schrijven.
Het resultaat is dus dat de functie meteen returnt zonder het bericht af te printen.
Op de meeste besturingssystemen zal dit niet werken en zal het programma crashen door een _segmentation fault_ (of iets gelijkaardigs) veroorzaakt door een page fault.
Je kan dit eens proberen op Linux bijvoorbeeld.
Aangezien de code in xv6 writable gemapt is, gaat dit daar _wel_ werken.
Controleer dit door het aangepaste programma uit te voeren.

Om dit probleem op te lossen, zullen we de [`exec`][exec] functie aan moeten passen.
In de [lus die de ELF secties laadt][exec load loop], wordt [`uvmalloc`][uvmalloc] opgeroepen om voor elke sectie de nodige mappings toe te voegen.
Het probleem is dat `uvmalloc` elke sectie als `rwx` mapt.
We gaan dus code toevoegen om de flags van een mapping aan te passen en gelijk te stellen aan de flags van de overeenkomstige ELF sectie.

- Implementeer de volgende functie in `vm.c` (en voeg de functiedefinitie toe in `defs.h`):
  ```c
  void vmsetflags(pagetable_t pagetable, uint64 va, uint64 len, uint flags)
  ```
  Deze functie moet voor alle pagina's in het interval `[va, va + len)` de flags van de PTE op `flags` zetten.
    - Gebruik [`walk`][walk] om de PTE van een virtueel adres te krijgen.
    - De constante [`PTE_FLAGS_MASK`][PTE_FLAGS_MASK] kan gebruikt worden om de flag bits in een PTE te masken.
      Je kan de volgende code gebruiken om de flags te overschrijven (probeer te begrijpen hoe dit werkt!): 
      ```c
      *pte &= ~PTE_FLAGS_MASK;
      *pte |= (flags & PTE_FLAGS_MASK);
      ```
    - Een goede gewoonte is om checks toe te voegen om zeker te ziin dat niets fataal misloopt. Hiervoor kan je altijd de functie `panic(char *info)` gebruiken om de kernel af te sluiten met een foutboodschap als iets fataal misloopt dat je niet verwacht onder normale omstandigheden.
- Roep `vmsetflags` op in `exec` na het laden van elke sectie.
  Om de juiste flags mee te geven, moet je de ELF flags (in [`struct proghdr`][struct proghdr]) vertalen naar de juiste PTE flags.
  Gebruik hiervoor de [constanten][ELF flags consts] gedefinieerd in [`elf.h`][elf.h].
  Vergeet zeker niet de `PTE_V` en `PTE_U` flags te zetten!

Verifieer je implementatie via de `vmprintmappings` syscall.

> :information_source: In deze oefening moeten we bepaalde bits in page table entries aanpassen. Om met bitvoorstellingen van getallen te werken in C gebruiken we de bitwise operatoren zoals `&` (AND), `|` (OR) en `~` (NOT). Voor meer informatie kan je [hier](https://github.com/informaticawerktuigen/oefenzitting-c/tree/main/les3-bits-and-bytes#bitoperatoren) terecht.

[walk]: https://github.com/besturingssystemen/xv6-riscv/blob/d4cecb269f2acc61cc1adc11fec2aa690b9c553b/kernel/vm.c#L81
[uvmalloc]: https://github.com/besturingssystemen/xv6-riscv/blob/720a130ceafcc55ec3624b47e8a1368f3f5f00ae/kernel/vm.c#L215
[exec]: https://github.com/besturingssystemen/xv6-riscv/blob/720a130ceafcc55ec3624b47e8a1368f3f5f00ae/kernel/exec.c#L12
[exec load loop]: https://github.com/besturingssystemen/xv6-riscv/blob/720a130ceafcc55ec3624b47e8a1368f3f5f00ae/kernel/exec.c#L41-L59
[PTE_FLAGS_MASK]: https://github.com/besturingssystemen/xv6-riscv/blob/720a130ceafcc55ec3624b47e8a1368f3f5f00ae/kernel/riscv.h#L345
[struct proghdr]: https://github.com/besturingssystemen/xv6-riscv/blob/720a130ceafcc55ec3624b47e8a1368f3f5f00ae/kernel/elf.h#L30
[ELF flags consts]: https://github.com/besturingssystemen/xv6-riscv/blob/720a130ceafcc55ec3624b47e8a1368f3f5f00ae/kernel/elf.h#L44-L47
[elf.h]: https://github.com/besturingssystemen/xv6-riscv/blob/720a130ceafcc55ec3624b47e8a1368f3f5f00ae/kernel/elf.h
