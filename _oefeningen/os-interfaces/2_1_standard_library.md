---
layout: default
title: Standard Library
nav_order: 0
parent: "Userspace"
grand_parent: "Zitting 1: OS Interfaces"
---



# Libraries

De libraries die je kan terugvinden in de `#include`-statements zijn niet de standaard libraries die we kennen wanneer we C programmeren.

> :information_source: De [C Standard Library](https://en.wikipedia.org/wiki/C_standard_library) bestaat uit een verzameling nuttige functies die door C-programma's gebruikt kunnen worden. De implementatie van deze functies hangt echter af van het onderliggende besturingssysteem. Op Ubuntu en vele andere Unix-distributies heet deze implementatie `glibc` of [The GNU C Library](https://www.gnu.org/software/libc/). Een andere populaire implementatie heet [`musl`](https://musl.libc.org/).

xv6 biedt maar een zeer beperkte implementatie van libc aan. Daardoor kunnen we niet zomaar `#include <stdio.h>` gebruiken om bijvoorbeeld de functie `printf` op te roepen.
De libc-functies die xv6 aanbiedt kan je terugvinden in `user/user.h`.

* Open het bestand `user/user.h`

    ```console
    [ubuntu-shell]$ gedit user/user.h
    ```

De functies in dit bestand kunnen gebruikt worden door alle user space programma's. In plaats van de standaard C header files te includen (zoals `stdio.h`) zullen we in xv6 telkens het bestand `user/user.h` includen.

De functies in `user/user.h` maken zelf gebruik van types gedefinieerd in `kernel/types.h`. Start dus standaard een user space programma voor xv6 met

```c
#include "kernel/types.h"
#include "user/user.h"
```
