---
layout: default
title: "Permanente evaluatie"
nav_order: 4
parent: "Zitting 5: Locks"
---

# Permanente evaluatie

Voor de permanente evaluatie gaan we `kalloc` aanpassen om de lock contention te verminderen.
De oorzaak van de hoge contention in `kalloc` is het gebruik van één enkele free list voor alle processoren.
Als elke processor een eigen free list zou hebben, kunnen we de contention (bijna) volledig vermijden.

Er is echter één probleem met dit idee: wat als de free list van een processor leeg is maar een andere processor nog wel vrije frames heeft?
Het zou jammer zijn als `kalloc` om die reden zou falen.
Er moet dus een manier verzonnen worden zodat processors vrije frames van elkaar kunnen "stelen".
Dit is ook de reden dat we de contention nooit volledig kunnen vermijden: processors zullen soms op elkaar moeten wachten terwijl frames verplaatst worden tussen free lists.

Het idee is dus het volgende: in plaats van één enkele globale variabele [`kmem`][kmem] (die een free list en een spinlock bevat), maken we zo een variabele per processor.
`kalloc` en `kfree` gebruiken dan de `kmem` variabele van de huidige processor om frames te alloceren en vrij te geven.
Wanneer `kalloc` geen frames meer vindt in deze free list, gaat het zoeken in de free list van andere processoren en verplaatst het een aantal frames.

> **:question: Permanente evaluatie.** Implementeer per-processor free lists voor `kalloc`.
> Verifieer dat xv6 nog steeds goed werkt via de `usertests`.
> Verifieer dat de lock contention vermindert door alle locks te registreren met `perf_register_spinlock` en `stressmem` te runnen.

> :bulb: xv6 heeft een vast maximum aantal processoren gedefinieerd door de [`NCPU`][NCPU] constante.
> Maak dus een array aan van `NCPU` `kmem` variabelen.

> :bulb: In RISC-V heeft elke processor in een multiprocessor systeem een unieke id (een getal beginnende bij 0) dat terug te vinden is in het `mhartid`<sup>1</sup> CSR.
> Dit register is enkel toegankelijk in machine mode, niet in supervisor mode waarin xv6 runt.
> Om het processor id toch te kunnen achterhalen, [slaat xv6 `mhartid` op in het `tp` register][store mhartid] tijdens het booten in machine mode.
> Deze waarde kan later opgevraagd worden met de [`cpuid`][cpuid] functie.
> Je kan dit als index in de `kmem` array gebruiken om de free list van de huidige processor te krijgen.
>
> <sup>1</sup> RISC-V gebruikt de term _hart_ (hardware thread) om te verwijzen naar een processor.

> :bulb: [`kinit`][kinit] wordt door [`main`][main] opgeroepen op CPU 0.
> Je kan tijdens de initialisatie van `kalloc` alle frames toewijzen aan deze CPU.
> De andere CPUs zullen dan frames stelen wanneer ze er nodig hebben.
>
> Je kan eventueel eerst het simpelste geval van 1 CPU testen, zonder reeds het stelen van frames te implementeren, door xv6 als volgt op te starten:
> 
> ```shell
> [ubuntu-shell]$ CPUS=1 make qemu
> ```
> 
> Zorg hierna dat je stelen van frames correct implementeert en je oplossing ook werkt met meerdere CPUs!

> :bulb: `stressmem` alloceert standaard herhaaldelijk één frame en dealloceert deze onmiddelijk.
> Dit zal er dus niet voor zorgen dat processors vaak frames moeten stelen.
> Er is daarom een tweede test toegevoegd die herhaaldelijk zoveel mogelijk geheugen probeert te alloceren.
> Run hiervoor `stressmem --oom` (OOM staat voor _out-of-memory_).

> :bulb: Een belangrijke parameter voor je implementatie is het aantal frames dat per keer gestolen zal worden.
> Experimenteer met verschillende waardes en kies de beste.

> :warning: xv6 gebruikt timer interrupts om de tijd dat processen achter elkaar kunnen uitvoeren te beperken.
> Het kan dus op elk moment gebeuren dat de scheduler ervoor kiest om een proces te stoppen om een ander proces te laten uitvoeren.
> Wanneer het eerste proces later weer herstart wordt, kan dit op een andere processor gebeuren!
> Dit kan voor problemen zorgen voor code die `cpuid` gebruikt:
> ```c
> uint cpu = cpuid();
> // Timer interrupt here
> // Use "cpu", may not refer to current CPU due to scheduling!
> ```
> De makkelijkste manier om zulke problemen te voorkomen, is interrupts volledig uit te schakelen tijdens `kalloc`.
> Gebruik [`push_off` en `pop_off`][push pop off] om interrupts respectievelijk uit en aan te zetten.

[kalloc]: https://github.com/besturingssystemen/xv6-riscv/blob/85bfd9e71f6d0dc951ebd602e868880dedbe1688/kernel/kalloc.c#L65
[kfree]: https://github.com/besturingssystemen/xv6-riscv/blob/85bfd9e71f6d0dc951ebd602e868880dedbe1688/kernel/kalloc.c#L42
[kinit]: https://github.com/besturingssystemen/xv6-riscv/blob/85bfd9e71f6d0dc951ebd602e868880dedbe1688/kernel/kalloc.c#L26
[freerange]: https://github.com/besturingssystemen/xv6-riscv/blob/85bfd9e71f6d0dc951ebd602e868880dedbe1688/kernel/kalloc.c#L33
[exec]: https://github.com/besturingssystemen/xv6-riscv/blob/85bfd9e71f6d0dc951ebd602e868880dedbe1688/kernel/exec.c#L12
[exec uvmalloc]: https://github.com/besturingssystemen/xv6-riscv/blob/85bfd9e71f6d0dc951ebd602e868880dedbe1688/kernel/exec.c#L52
[uvmalloc]: https://github.com/besturingssystemen/xv6-riscv/blob/85bfd9e71f6d0dc951ebd602e868880dedbe1688/kernel/vm.c#L218
[uvmalloc kalloc]: https://github.com/besturingssystemen/xv6-riscv/blob/85bfd9e71f6d0dc951ebd602e868880dedbe1688/kernel/vm.c#L231
[uvmalloc mappages]: https://github.com/besturingssystemen/xv6-riscv/blob/85bfd9e71f6d0dc951ebd602e868880dedbe1688/kernel/vm.c#L237
[sys_sbrk]: https://github.com/besturingssystemen/xv6-riscv/blob/85bfd9e71f6d0dc951ebd602e868880dedbe1688/kernel/sysproc.c#L41
[growproc]: https://github.com/besturingssystemen/xv6-riscv/blob/85bfd9e71f6d0dc951ebd602e868880dedbe1688/kernel/proc.c#L243
[walk]: https://github.com/besturingssystemen/xv6-riscv/blob/85bfd9e71f6d0dc951ebd602e868880dedbe1688/kernel/vm.c#L71
[walk kalloc]: https://github.com/besturingssystemen/xv6-riscv/blob/85bfd9e71f6d0dc951ebd602e868880dedbe1688/kernel/vm.c#L94
[memlayout.h]: https://github.com/besturingssystemen/xv6-riscv/blob/85bfd9e71f6d0dc951ebd602e868880dedbe1688/kernel/memlayout.h
[KERNBASE]: https://github.com/besturingssystemen/xv6-riscv/blob/85bfd9e71f6d0dc951ebd602e868880dedbe1688/kernel/memlayout.h#L55
[PHYSTOP]: https://github.com/besturingssystemen/xv6-riscv/blob/85bfd9e71f6d0dc951ebd602e868880dedbe1688/kernel/memlayout.h#L56
[end]: https://github.com/besturingssystemen/xv6-riscv/blob/85bfd9e71f6d0dc951ebd602e868880dedbe1688/kernel/kernel.ld#L43
[struct run]: https://github.com/besturingssystemen/xv6-riscv/blob/85bfd9e71f6d0dc951ebd602e868880dedbe1688/kernel/kalloc.c#L17
[kmem]: https://github.com/besturingssystemen/xv6-riscv/blob/3fa0348a978d50b11ca29b58ab474b8753d6661b/kernel/kalloc.c#L21-L24
[kmem.freelist]: https://github.com/besturingssystemen/xv6-riscv/blob/85bfd9e71f6d0dc951ebd602e868880dedbe1688/kernel/kalloc.c#L23
[kmem.lock]: https://github.com/besturingssystemen/xv6-riscv/blob/85bfd9e71f6d0dc951ebd602e868880dedbe1688/kernel/kalloc.c#L22
[struct spinlock]: https://github.com/besturingssystemen/xv6-riscv/blob/85bfd9e71f6d0dc951ebd602e868880dedbe1688/kernel/spinlock.h#L6
[acquire]: https://github.com/besturingssystemen/xv6-riscv/blob/85bfd9e71f6d0dc951ebd602e868880dedbe1688/kernel/spinlock.c#L19
[release]: https://github.com/besturingssystemen/xv6-riscv/blob/85bfd9e71f6d0dc951ebd602e868880dedbe1688/kernel/spinlock.c#L45
[spinlock counters]: https://github.com/besturingssystemen/xv6-riscv/blob/3fa0348a978d50b11ca29b58ab474b8753d6661b/kernel/spinlock.h#L13-L14
[num_acquires count]: https://github.com/besturingssystemen/xv6-riscv/blob/3fa0348a978d50b11ca29b58ab474b8753d6661b/kernel/spinlock.c#L30
[contention count]: https://github.com/besturingssystemen/xv6-riscv/blob/3fa0348a978d50b11ca29b58ab474b8753d6661b/kernel/spinlock.c#L37
[perf.h]: https://github.com/besturingssystemen/xv6-riscv/blob/3fa0348a978d50b11ca29b58ab474b8753d6661b/kernel/perf.h#L1
[perf_register_spinlock]: https://github.com/besturingssystemen/xv6-riscv/blob/3fa0348a978d50b11ca29b58ab474b8753d6661b/kernel/perf.c#L9
[perf_print_spinlocks]: https://github.com/besturingssystemen/xv6-riscv/blob/3fa0348a978d50b11ca29b58ab474b8753d6661b/kernel/perf.c#L20
[stressmem]: https://github.com/besturingssystemen/xv6-riscv/blob/3fa0348a978d50b11ca29b58ab474b8753d6661b/user/stressmem.c
[alloc_dealloc]: https://github.com/besturingssystemen/xv6-riscv/blob/3fa0348a978d50b11ca29b58ab474b8753d6661b/user/stressmem.c#L35-L46
[NCPU]: https://github.com/besturingssystemen/xv6-riscv/blob/3fa0348a978d50b11ca29b58ab474b8753d6661b/kernel/param.h#L5
[store mhartid]: https://github.com/besturingssystemen/xv6-riscv/blob/3fa0348a978d50b11ca29b58ab474b8753d6661b/kernel/start.c#L44-L46
[cpuid]: https://github.com/besturingssystemen/xv6-riscv/blob/3fa0348a978d50b11ca29b58ab474b8753d6661b/kernel/proc.c#L54
[spinlock holding panic]: https://github.com/besturingssystemen/xv6-riscv/blob/85bfd9e71f6d0dc951ebd602e868880dedbe1688/kernel/spinlock.c#L25-L26
[push pop off]: https://github.com/besturingssystemen/xv6-riscv/blob/85bfd9e71f6d0dc951ebd602e868880dedbe1688/kernel/spinlock.c#L84-L110
[main]: https://github.com/besturingssystemen/xv6-riscv/blob/85bfd9e71f6d0dc951ebd602e868880dedbe1688/kernel/main.c#L9
[xv6-riscv bss]: https://github.com/besturingssystemen/xv6-riscv
