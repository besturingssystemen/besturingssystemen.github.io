---
layout: default
title: "Oefening: Lock contention"
nav_order: 3
parent: "Zitting 5: Locks"
---

# Oefening: Lock contention

Het doel van spinlocks is dus de uitvoering van critical sections door meerdere processoren te _serializeren_.
Met anderen woorden, terwijl een processor een critical section aan het uitvoeren is, zullen de andere processoren moeten wachten.
Alhoewel dit de correctheidsproblemen van `kalloc` oplost, heeft het een impact op de performantie: op elk moment zal slechts één processor een frame kunnen alloceren.
Andere processoren die op hetzelfde moment een frame nodig hebben, zullen moeten wachten en geen nuttige instructie kunnen uitvoeren.

_Lock contention_ is de situatie waar meerdere processoren tegelijk dezelfde critical section binnen willen gaan en dus `acquire` oproepen op dezelfde spinlock.
Hoe vaker dit voorkomt, hoe groter de impact op de performantie zal zijn.
Een manier om dit te kwantificeren is om het aantal iteraties in de `while` loop van `acquire` te tellen; dit is een maat voor de hoeveelheid nutteloze cycles uitgevoerd door processoren.

Om lock contention in xv6 te kunnen meten, hebben we spinlocks uitgebreid met [twee tellers][spinlock counters]: `num_acquires` [telt][num_acquires count] het aantal keer dat `acquire` werd opgeroepen per spinlock en `contention` [telt][contention count] het aantal iteraties in de `while` loop van `acquire`.
Verder zijn er twee kernel functies toegevoegd in `perf.h`:

- [`void perf_register_spinlock(struct spinlock* lock)`][perf_register_spinlock]: Registreer een spinlock om af te printen.
- [`void perf_print_spinlocks()`][perf_print_spinlocks]: Print de contention counters van alle geregistreerde spinlocks.
  Deze functie wordt opgeroepen als je <kbd>CTRL</kbd>+<kbd>L</kbd> typt in de console.

> **:question: Oefening.** Meet de lock contention van de `kalloc` spinlock.
> 1. Roep `perf_register_spinlock` op in [`kinit`][kinit] (vergeet `perf.h` niet te `#include`n);
> 2. Voer `usertests` uit;
> 3. Typ <kbd>CTRL</kbd>+<kbd>L</kbd> in de xv6 console.
>
>    Als het goed is, zul je zien dat de lock contention van `kmem.spinlock` relatief laag is ten opzichte van het totale aantal keer dat `acquire` werd opgeroepen.
>    Dit komt omdat `usertests` geen tests uitvoert die op meerdere processoren tegelijkertijd proberen geheugen te alloceren.
>    We hebben daarom een nieuwe test, [`stressmem`][stressmem], toegevoegd die drie processen start (xv6 runt standaard op drie processoren) en in elk process in een [loop `sbrk` oproept][alloc_dealloc] om een page te alloceren en dan weer vrij te geven.
>
> 4. Meet de lock contention na het uitvoeren van `stressmem`.
>
>    Nu zul je zien dat de lock contention significant is.

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
