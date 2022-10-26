---
layout: default
title: Page tables
nav_order: 3
parent: "Adrestranslatie in xv6"
grand_parent: "Zitting 3: Virtual memory"
---

## Page tables in xv6 en RISC-V

In xv6 worden pagina's gemapt met behulp van de functie `mappages`.
Dit is de declaratie van deze functie:

```c
// Create PTEs for virtual addresses starting at va that refer to
// physical addresses starting at pa. va and size might not
// be page-aligned. Returns 0 on success, -1 if walk() couldn't
// allocate a needed page-table page.
int
mappages(pagetable_t pagetable, uint64 va, uint64 size, uint64 pa, int perm);
```

> :information_source: De implementatie van mappages vergt veel uitleg en leert ons weinig nieuwe informatie. Voor de geïnteresseerden geven we [hier](../extra/page_table_code) deze uitleg. Volg echter eerst de rest van de oefenzitting.

Stel dat je de trampolinepagina zou moeten mappen met behulp van de functie [`mappages`][mappages].

  * Welke waarden zou je toekennen aan de parameters `va`, `size` en `pa`?

[mappages]: https://github.com/besturingssystemen/xv6-riscv/blob/d4cecb269f2acc61cc1adc11fec2aa690b9c553b/kernel/vm.c#L138

