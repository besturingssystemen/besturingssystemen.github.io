---
layout: default
title: "System calls vs function calls"
nav_order: 3
parent: "Zitting 2: System calls"
search_exclude: true
has_children: true
has_toc: false
---

# System calls vs function calls

Ondertussen weten we hoe een proces opgestart kan worden, door middel van de system calls `fork` en `exec`.
We weten ook dat we de system call `write` kunnen gebruiken om te schrijven naar de console, een buffer, een bestand, of naar een pipe geopend met de system call `pipe`.

System calls geven processen de mogelijkheid diensten te vragen aan het besturingssysteem.
Bij het uitvoeren van een system call geeft een proces tijdelijk de controle over de processor door aan het OS.

Een system call lijkt op het eerste zicht erg op een function call.
In de vorige sessie hebben de userspace functie `puts(char* str)` geschreven die een string schrijft naar de `stdout`.

Een proces dat een string wil schrijven naar de `stdout` kan de *function call* `puts` gebruiken, als volgt:

```c
const char* str = "Hello, world!";
puts(str);
```

Een proces dat een string wil schrijven naar de `stdout` kan ook de *system call* `write` gebruiken, als volgt:

```c
const char* str = "Hello, world!\n";
write(1, str, strlen(str));
```

In dit geval biedt C echter een wrapper-functie aan, zodat het mogelijk is om een system call op te roepen als een functie.
De echte syscall gebeurt dus in de implementatie van de `write` wrapper-functie.
Deze wrappers moeten echter op assembly-niveau geïmplementeerd worden.
Het is dus tijd om terug in assembly te duiken.

> :exclamation: `puts` en andere functies zoals `printf` maken intern ook gebruik van de system call `write`. Het is niet mogelijk te schrijven zonder een system call. Dat verandert niets aan het feit dat oproepen naar `puts` en `printf` gewone function calls zijn.
