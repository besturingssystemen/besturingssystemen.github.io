---
layout: default
title: "Achtergrond: Locking"
nav_order: 2
parent: "Zitting 5: Locks"
---

# Achtergrond: Locking
{: .no_toc }

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>


xv6 ondersteunt _multiprocessor_ systemen.
Dit wil zeggen dat meerdere processoren _tegelijkertijd_ code aan het uitvoeren zijn.
Deze code kan tot verschillende processen behoren maar het kan ook gebeuren dat meerdere processoren tegelijkertijd kernelcode aan het uitvoeren zijn.
Vanaf het moment dat data structuren in de kernel door meerdere processoren gelijktijdig gebruikt worden, kunnen er problemen ontstaan.

> :warning: Synchronizatieproblemen kunnen optreden zelfs zonder multiprocessor systeem. Wanneer twee processen een datastructuur delen en de kernel op willekeurige momenten wisselt tussen deze processen, lijkt het voor de processen alsof ze tegelijkertijd uitgevoerd worden, met standaard synchronizatieproblemen tot gevolg. In kernel space heb je hier op een single core processor minder snel last van, aangezien er maar één kernel is, maar de problematiek komt zelfs dan nog steeds voor. Traps kunnen kernel-code op willekeurige momenten onderbreken waardoor deze code in feite "in parallel" wordt uitgevoerd.

## Kritische sectie in `kalloc`

Neem bijvoorbeeld de volgende vereenvoudigde versie van `kalloc`:

```c
void* kalloc()
{
  // 1: Take a pointer to the first free frame
  struct run* r = kmem.freelist;

  if (r != 0) {
    // 2: If we still had a free frame, make freelist point to the next one
    kmem.freelist = r->next;
  }

  // 3: Return address of first free frame
  return (void*)r;
}
```

Stel je nu voor dat twee processoren `kalloc` tegelijkertijd uitvoeren.
Na het uitvoeren van stap 1 zullen ze beiden een lokale variabele `r` hebben die de waarde van `kmem.freelist` bevat (het adres van het eerste vrije frame).
Essentieel aan het probleem is dat `r` op beide processoren _dezelfde_ waarde zal hebben.
In stap 2 passen ze beide `kmem.freelist` aan naar `r->next` wat hier geen groot probleem is aangezien `r` dezelfde waarde heeft.
Stap 3 geeft op beide processoren het adres van hetzelfde frame terug aan de oproeper van `kalloc`.
We zitten nu dus in de situatie waar twee verschillende processoren hetzelfde frame gaan gebruiken voor verschillende doeleinden.
Stel je bijvoorbeeld voor dat één processor `exec` aan het uitvoeren was en de ander `sbrk`; hetzelfde frame zal dus gebruikt worden voor de code van één proces en voor de heap van een ander.

> **:question: Oefening.** Laten we eens bekijken wat er in de praktijk gebeurt als we geen locks zouden hebben in `kalloc`.
>
> 1. Haal de calls naar `acquire` en `release` weg uit [`kalloc`][kalloc] en run de `usertests`.
     Verklaar het resultaat.<sup>1</sup>
> 2. Doe nu hetzelfde voor `kfree` maar denk eerst na over wat er mis zou kunnen lopen.
     Is het gebrek aan locks in `kfree` even gevaarlijk als in `kalloc`?
>
> :bulb: <sup>1</sup> Eén van de dingen die concurrency (het tegelijkertijd uitvoeren van code) zo moeilijk maakt, is dat het vaak _non-deterministisch_ is.
> De bovenstaande beschrijving van wat er mis kan gaan met `kalloc` zal enkel gebeuren als er inderdaad twee of meerdere processoren ongeveer tegelijkertijd `kalloc` uitvoeren.
> Niets garandeert dat dit altijd gebeurt.
> Het kan dus zijn dat je `usertests` meerdere keren moet uitvoeren voordat je tegen een probleem aanloopt.

## Spinlocks in xv6

Om dit probleem op te lossen, moeten we er voor zorgen dat er maar één processor tegelijkertijd `kmem.freelist` kan manipuleren.
Dit typische _critical section_ probleem wordt in `kalloc` opgelost via _spinlocks_.

Een spinlock in xv6 wordt voorgesteld door een [`struct spinlock`][struct spinlock].
De belangrijkste variabele in deze struct is `locked`, een integer die aangeeft of er momenteel een processor in de critical section aan het uitvoeren is.
De volgende operaties worden aangeboden voor spinlocks:

- [`void acquire(struct spinlock* lk)`][acquire]: Wacht (_busy waiting_) tot `lk` niet meer `locked` is, zet `locked` en return.
  Deze functie wordt gebruikt om een critical section binnen te gaan.
- [`void release(struct spinlock* lk)`][release]: Zet `locked` op 0.
  Deze functie wordt gebruikt om een critical section te verlaten.

In principe voert `acquire` dus de volgende code uit:

```c
void acquire(struct spinlock* lk)
{
  while (lk->locked != 0) {
    // Busy waiting
  }

  lk->locked = 1;
}

```

Hier hebben we echter hetzelfde probleem als met de eerder getoonde implementatie van `kalloc`: wat als twee processoren tegelijkertijd `acquire` uitvoeren?
In dit geval kan het gebeuren dat beide processoren `lk->locked` als 0 lezen en dus voorbij de loop geraken.
Dit zorgt dus voor twee processoren in de critical section wat we net wilden vermijden me het gebruik van spinlocks.

Zoals beschreven in sectie 6.4 van het Operating System Concepts boek, kan dit opgelost worden met een atomische `TestAndSet` instructie.
Op RISC-V heet deze instructie `amoswap` en de GCC compiler biedt een functie genaamd `__sync_lock_test_and_set` aan om deze instructie gemakkelijk te gebruiken.
De functie zet atomisch de waarde van een integer en geeft de oude waarde terug.
Hiermee kunnen we `acquire` juist implementeren:

```c
void acquire(struct spinlock* lk)
{
  // Atomically set lk->locked to 1 and return its old value
  while (__sync_lock_test_and_set(&lk->locked, 1) != 0) {
    // Busy waiting
  }
}
```

De `release` functie zet in principe `locked` simpelweg op 0.
Ook hier moet ervoor gezorgd worden dat deze assignment atomisch gebeurt:

```c
void release(struct spinlock* lk)
{
  // Atomic version of "lk->locked = 0"
  __sync_lock_release(&lk->locked);
}
```

> :bulb: De implementaties van [`acquire`][acquire] en [`release`][release] zijn in de praktijk nog iets complexer.
> Zo wordt de `__sync_synchronize` GCC functie gebruikt om ervoor te zorgen dat, onder andere, load en store instructies _voor_ `acquire` niet door bepaalde optimizaties tot _na_ `acquire` verplaatst worden.
> De commentaren in deze functies en sectie 6.6 van het xv6 boek gaan hier wat dieper op in.

> **:question: Oefening.** Bekijk en verklaar het gebruik van `acquire` en `release` in [`kalloc`][kalloc] en [`kfree`][kfree].

## Deadlocks

Spinlocks zorgen ervoor dat een processor soms moeten wachten op een andere processor bij het binnengaan van een critial section.
Wat als meerdere processors _op elkaar_ moeten wachten?
Stel je voor dat de volgende twee functies door verschillende processors uitgevoerd worden:

```c
void cpu0()
{
  acquire(&lock_a);
  acquire(&lock_b);
  // Critical section
  release(&lock_b);
  release(&lock_a);
}

void cpu1()
{
  acquire(&lock_b);
  acquire(&lock_a);
  // Critical section
  release(&lock_a);
  release(&lock_b);
}
```

Het zou kunnen gebeuren dat de instructies in de volgende volgorde uitgevoerd worden:

```text
   cpu0              | cpu1
   ------------------|------------------
1: acquire(&lock_a); |
2:                   | acquire(&lock_b);
3:                   | acquire(&lock_a);
4: acquire(&lock_b); |
```

In stap 3 zal `cpu1` moeten wachten op `cpu0` omdat `lock_a` genomen werd door `cpu0` in stap 1.
Tegelijkertijd zal `cpu0` in stap 4 moeten wachten op `cpu1` door `lock_b`.
Er zal nu geen voortgang meer gemaakt kunnen worden omdat beide processors op elkaar aan het wachten zijn.
Dit wordt een _deadlock_ genoemd.

Deadlocks kunnen voorkomen wanneer een processor _op hetzelfde moment_ meerdere spinlocks nodig heeft.
Als een andere processor dezelfde spinlocks ook nodig heeft, kan er een deadlock ontstaan wanneer de verschillende processors de locks in een andere volgorde proberen te krijgen.

Er is daarom een relatief eenvoudige vuistregel om deadlocks te voorkomen: zorg voor een consistente _lock ordering_.
Als je er voor zorgt dat wanneer er meerdere locks nodig zijn alle processors deze locks in dezelfde volgorde proberen te krijgen, zullen er geen deadlocks voor kunnen komen.

> **:question: Oefening.** Overtuig jezelf dat een consistente lock ordering het probleem in het bovenstaande voorbeeld oplost.

> :bulb: Het is je misschien opgevallen dat er zelfs met één lock een deadlock kan optreden: wanneer dezelfde processor een tweede keer eenzelfde lock probeert te krijgen.
> Aangezien deze situatie niet voor zou mogen komen in correcte code, zal xv6 in dit geval simpelweg [`panic` oproepen][spinlock holding panic].

> :bulb: Deadlocks zijn vaak zeer moeilijk te debuggen omdat je programma gewoon niets meer doet.
> Je kan echter [GDB][gdb] gebruiken om meer informatie te krijgen.
> Op het moment dat xv6 vast zit en je vermoedt dat er een deadlock is, typ je <kbd>CTRL</kbd>+<kbd>C</kbd>, dit zorgt ervoor dat alle processors stoppen met uitvoeren.
> Je kan nu de staat van elke processor bekijken met het commando `info threads`.
> Dit toont een lijst met alle processors en de functie waarin ze op dit moment aan het uitvoeren waren.
> Als er een deadlock was, zal je zien dat minstens twee processors `acquire` aan het uitvoeren waren.
> Je kan nu naar een specifieke processor switchen via `thread id` (waar je `id` vervangt door de Id in de `info threads` output) om daar in detail te bekijken wat er aan de hand is (bijvoorbeeld via het `backtrace` commando).

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
[gdb]: ../../../tutorials/gdb
[push pop off]: https://github.com/besturingssystemen/xv6-riscv/blob/85bfd9e71f6d0dc951ebd602e868880dedbe1688/kernel/spinlock.c#L84-L110
[main]: https://github.com/besturingssystemen/xv6-riscv/blob/85bfd9e71f6d0dc951ebd602e868880dedbe1688/kernel/main.c#L9
