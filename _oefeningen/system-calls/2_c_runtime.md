---
layout: default
title: "Oefening: C Runtime"
nav_order: 2
parent: "Zitting 2: System calls"
---

# C runtime (crt0)

In vorige oefenzitting hebben we gemerkt dat wanneer we returnen uit main, we een exception krijgen. We kunnen nu begrijpen waarom dit gebeurt.

Een proces wordt gestart met behulp van `exec` door een programma in te laden en vervolgens te springen naar het entry point van dat proces. `xv6` gebruikt als entry point `main`. Dat wil zeggen dat `main` niet als functie wordt opgeroepen en je dus ook geen `return` kan uitvoeren uit main (want het zal geen geldig return adres hebben).

In vele Linux-distributies wordt gebruik gemaakt van [crt0](https://en.wikipedia.org/wiki/Crt0) om C-programma's te starten.
Het entry point van een C executable wordt geplaatst in `crt0`, vaak in een functie genaamd `_start`.

`_start` in `crt0` is dan het absolute beginpunt bij de uitvoering van een uit C gecompileerde executable. `_start` initialiseert typisch de C-runtime (dit is niet nodig voor de eenvoudige runtime van xv6) en roept vervolgens `main` op. Na uitvoering van `main` wordt de return-value van `main` doorgegeven aan de `exit` system call. Zo wordt het proces transparant afgesloten.

Om het mogelijk te maken om te returnen uit `main` zonder exceptions, voegen we nu onze eigen `crt0` toe aan xv6.

* Maak een bestand `user/crt0.c` aan
* Voeg `$U/crt0.o` toe aan `ULIB` in de Makefile van `xv6`. We zorgen er hiermee dus voor dat `crt0` gelinked wordt aan elk user-space programma.
* Implementeer `void _start(int argc, char* argv[])` in `user/crt0.c`

```c
#include "kernel/types.h"
#include "user/user.h"

extern int main(int argc, char* argv[]);

void _start(int argc, char* argv[])
{
    //TODO: Implement!
}
```

* Pas het entry point (`-e` flag) aan dat aan [`ld` wordt gegeven][ld rule] in de `Makefile` van xv6
* Test nu je implementatie van `crt0` door een eenvoudig `user/helloworld.c` programma te schrijven met een `return` uit `main` (in plaats van een oproep naar `exit`, zoals in de vorige oefenzitting). Let op: het bestand `user/helloworld.c` van de vorige oefenzitting zit niet meer in deze repository, en je zal dus een nieuw bestand moeten aanmaken. 
* Indien `crt0` correct is geïmplementeerd, krijg je geen user exception na uitvoering van het programma. Eventueel kan je extra `puts` print statements toevoegen om de uitvoering in `crt0` te volgen.

[ld rule]: https://github.com/besturingssystemen/xv6-riscv/blob/2b5934300a404514ee8bb2f91731cd7ec17ea61c/Makefile#L97
