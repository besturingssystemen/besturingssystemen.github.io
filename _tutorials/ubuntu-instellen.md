---
layout: default
title: Ubuntu instellen
nav_order: 1
nav_exclude: false
---

# Packages

Open een terminal met ```CTRL+ALT+T```.

* Update je Linux-omgeving:

    ```console
    $ sudo apt update
    $ sudo apt upgrade
    ```

* Installeer alle nodige packages in Ubuntu:

    ```console
    $ sudo apt install build-essential git qemu-system-misc gcc-riscv64-linux-gnu 
    ```

> :warning: Indien je een oude release van Ubuntu hebt (<20.04) zal je niet de laatste versie van de RISC-V compiler kunnen installeren. In dat geval moet je je distributie upgraden met ```sudo do-release-upgrade``` of de compiler zelf builden vanuit de RISC-V GNU Toolchain Git repository.

# xv6

Ten slotte willen we het besturingssysteem [xv6](https://github.com/besturingssystemen/xv6-riscv) clonen en compileren naar [RISC-V](https://riscv.org/):

* Clone de xv6 git-repository

    ```console
    $ git clone https://github.com/besturingssystemen/xv6-riscv
    ```

* Compileer xv6

    ```console
    $ cd xv6-riscv
    $ make qemu
    ```

Indien de compilatie succesvol is verlopen zal de xv6 shell automatisch gestart worden in de huidige shell. Je krijgt nu onderstaande boodschap te zien:

```console
xv6 kernel is booting

hart 2 starting
hart 1 starting
init: starting sh
$ 
```

Sluit de xv6 shell af met de toetsenbordcombinatie ```CTRL+A x```.

> :bulb: ```CTRL+A x``` != ```CTRL+A+x```. Duw eerst op ```CTRL+A```, laat dit los, en druk vervolgens op ```x```.
