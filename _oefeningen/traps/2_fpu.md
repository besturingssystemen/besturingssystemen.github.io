---
layout: default
title: "Oefening: Floating point ondersteuning"
nav_order: 2
parent: "Zitting 4: Traps"
---

# Oefening: Floating point ondersteuning

Het is je misschien opgevallen dat alle user space programma's die we tot nu toe geschreven hebben, enkel _integer_ operaties gebruiken.
RISC-V ondersteunt ook _floating point_ operaties om niet-gehele getallen te bewerken.
Op de meeste processoren, alsook op RISC-V, worden zulke operaties uitgevoerd door een zogenaamde _floating point unit_ (FPU).
Deze FPU zal berekeningen uitvoeren op speciale registers die onafhankelijk zijn van de general purpose registers die gebruikt worden voor integer operaties.
Op RISC-V zijn er 32 floating-point registers, genaamd `f0` tot `f31`, en een floating-point control and status register genaamd `fcsr`.

1. Voeg het volgende user space programma toe dat gebruikt maakt van floating point operaties:
   ```c
   #include "user.h"

   double get_sum() {
     double sum = 0;

     for (double d = 1.23; d < 100; d *= 1.9) {
       sum += d;
     }

     return sum;
   }

   int main() {
     double sum = get_sum();
     printf("Sum: %d\n", (int)sum);
     return 0;
   }
   ```
   Op RISC-V beginnen alle floating point instructies met een `f`, bijvoorbeeld: `fld` (float load), `fadd`, `fmul`.
   Verifieer dat dit programma daadwerkelijk zulke instructies gebruikt door via `objdump` de assembly code van de `get_sum` functie af te printen (vervang `user/_test` door de executable van je eigen programma):
   ```shell
   [ubuntu-shell]$ riscv64-linux-gnu-objdump --disassemble=get_sum user/_test
   ```

   Merk op dat op de PC klassen, dit commando als volgt moet uitgevoerd worden:
   ```shell
   [ubuntu-shell]$ riscv64-unknown-elf-objdump --disassemble=get_sum user/_test
   ```

2. Voer de executable nu uit in xv6.
   Je zult merken dat het crasht door een exception.
   Welke exception wordt er precies gegenereerd en door welke instructie?

   Op RISC-V staat de FPU uit na het opstarten.
   Het [`mstatus`](../../../img/mstatus.png) CSR bevat twee bits (`FS`) die de status van de FPU beheren.
   De FPU kan aangezet worden door `FS` op `01` te zetten.

  > :warning: It seems that if you have a specific version of qemu, this FS bit is ignored. This seems to be a bug in qemu that was fixed in later versions.
  > You can check this by running `qemu-system-riscv64 --version`. If you have version 6.x.x, the above code may give no error. In that case, just proceed as if you would have gotten one.
  > The error should look like this error below.
   ```
   usertrap(): unexpected scause 0x0000000000000002 pid=3
            sepc=0x000000000000000c stval=0x0000000000000000
   ```

3. Zorg ervoor dat de FPU globaal aanstaat.
   De initiële waarde voor `mstatus` [wordt gezet][write mstatus] in de functie [`start`][start], de eerste C-functie die in de kernel wordt uitgevoerd.
   Je kan zien dat `mstatus` eerste gelezen wordt in de variabele `x` (geweldige naam!) via de functie [`r_mstatus`][r_mstatus].
   Dan worden er een aantal configuratie bits gezet in `x` (je hoeft niet te begrijpen wat deze precies doen) vooraleer `x` terug naar `mstatus` geschreven wordt via [`w_mstatus`][w_mstatus].
   Je kan de FPU dus aanzetten door de eerste bit van `FS` (bit 13 van `mstatus`) op 1 te zetten in de variabele `x`.
   > :bulb: Je kan in C een getal maken dat enkel bit 13 op 1 heeft staan via `(1 << 13)`.

   > :information_source: De Linux kernel staat voor efficiëntie redenen geen floating point code toe in kernel mode.
   > De FPU zal daar dus niet globaal aanstaan maar enkel bij het switchen naar user mode aangezet worden.
   > Ook de meeste user mode programma's maken geen gebruik van de FPU en het kan dus qua energieverbruik beter zijn om ook voor user space programma's de FPU niet standaard aan te zetten.
   > De kernel zou bijvoorbeeld de exception die gegenereerd wordt wanneer er een floating point instructie gebruikt wordt terwijl de FPU uitstaat op kunnen vangen en de FPU dan pas aanzetten.
   > Om deze oefening eenvoudig te houden, zetten we de FPU globaal aan maar we houden jullie niet tegen om een efficiëntere implementatie uit te proberen.

   Verifieer nu dat je test programma uitgevoerd kan worden zonder exceptions te genereren.

4. Voeg nu het volgende user space programma toe:
   ```c
   #include "user.h"

   double get_sum() {
     double sum = 0;

     for (double d = 1.23; d < 100; d *= 1.9) {
       sum += d;
       sleep(1);
     }

     return sum;
   }

   int main() {
     double sum = 0;

     if (fork() != 0) {
       sum = get_sum();
       wait(0);
     } else {
       sum = get_sum();
     }

     printf("Sum: %d\n", (int)sum);

     return 0;
   }

   ```
   Dit programma voert een gelijkaardige lus uit als het vorige maar doet dit in een parent en in een child proces.
   Verder zal in elke iteratie van de lus een syscall gebeuren.
   > :information_source: `sleep(n)` in xv6 zal het oproepende proces `n * 100ms` doen slapen.
   > Dit geeft andere processen de tijd om uit te voeren.

   Merk je iets op?
   Voer dit programma meerdere keren uit en bekijk de resultaten.

   Je hebt waarschijnlijk gemerkt dat je inconsistente resultaten krijgt (zo niet, ga terug naar punt 4).
   Zoals eerder uitgelegd, moet de code in de trampoline de waarden in de general purpose register opslaan in het trapframe om de waarden niet te verliezen.
   De FPU gebruikt echter andere registers en deze worden niet bewaard in het trapframe.

5. Zorg ervoor dat de [floating point registers](../../../img/fpu-state.png) (FLEN=64 bits) juist opgeslagen en hersteld worden door de trampoline code:
    - Breid [`struct trapframe`][trapframe] uit.
    - Voeg code toe aan de [trampoline][trampoline] om alle floating point registers op te slaan in (gebruik de `fsd` en `csrr` instructies) en weer te herstellen uit (`fld` en `csrw`) het trapframe.
6. Verifieer dat je test programma nu _wel_ consistente resultaten geeft.

[trampoline]: https://github.com/besturingssystemen/xv6-riscv/blob/1f555198d61d1c447e874ae7e5a0868513822023/kernel/trampoline.S
[trapframe]: https://github.com/besturingssystemen/xv6-riscv/blob/2b5934300a404514ee8bb2f91731cd7ec17ea61c/kernel/proc.h#L52
[write mstatus]: https://github.com/besturingssystemen/xv6-riscv/blob/103d9df6ce3154febadcf9a67791d526ec6b07ac/kernel/start.c#L23-L27
[r_mstatus]: https://github.com/besturingssystemen/xv6-riscv/blob/103d9df6ce3154febadcf9a67791d526ec6b07ac/kernel/riscv.h#L23
[w_mstatus]: https://github.com/besturingssystemen/xv6-riscv/blob/103d9df6ce3154febadcf9a67791d526ec6b07ac/kernel/riscv.h#L31
[start]: https://github.com/besturingssystemen/xv6-riscv/blob/103d9df6ce3154febadcf9a67791d526ec6b07ac/kernel/start.c#L19

