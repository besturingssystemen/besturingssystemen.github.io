---
layout: default
title: "Oefening: RISC-V function call"
nav_order: 2
parent: "System calls vs function calls"
grand_parent: "Zitting 2: System calls"
---

# Assembly function calls

Neem onderstaand simpel C-programma:

```c
int value = 5;

int sum(int i, int j){
    return i + j;
}

int main(){
    sum(value, 10);
}
```

Vertaald naar RISC-V assembly zou dit er als volgt kunnen uitzien:

* Lees de onderstaande assembly-code en zorg ervoor dat je elke regel begrijpt.
  
```s
.data                   #Start de data-sectie
    value: .word 0x5    #Init global value met waarde 5

.text                   #Start de code-sectie

.globl main             #Exporteer het symbool main
                        #Dit zorgt ervoor dat oa. crt0
                        #een functie-oproep naar main
                        #kan uitvoeren.


sum:                    #Definieer het symbool sum
                        #Calling convention:
                        #   a0: input argument 1 (i)
                        #   a1: input argument 2 (j)
                        #   a0: return value
    add a0, a0, a1      #Tel i en j op, geef terug via a0
    jr ra               #Spring naar het adres in register ra
                        #ra is het return-address register

main:                   #Definieer het symbool main

    addi sp, sp, -8     #Reserveer 8 bytes (64 bit) 
                        #op de call stack door de stack
                        #pointer te verlagen met 8
                        #(Stack groeit van hoog naar laag)

    sd ra, (sp)         #Bewaar de huidige waarde van
                        #het return-adres register ra
                        #op de call stack

    la t0, value        #Laad in t0 het adres van 
                        #de global value

    ld a0, (t0)         #Laad de waarde op het adres in t0
                        #in a0

    li a1, 10           #Laad de waarde 10 in a1

    jal ra, sum         #Spring naar symbool sum en
                        #sla in het return-address register
                        #ra het adres op waarnaar sum moet
                        #terugkeren

    ld ra, (sp)         #Haal het oude return-adres terug
                        #van de stack en bewaar het in ra

    addi sp, sp, 8      #Geef de gereserveerde 8 bytes op
                        #de stack terug vrij

    jr ra               #Spring naar het adres in register
                        #ra (een adres in crt0?)
```

In RISC-V worden functie-parameters in de eerste plaats doorgegeven via de registers `a0` - `a7`. Wanneer er niet genoeg registers zijn voor het aantal parameters, worden extra parameters via de stack meegegeven. Een volledige beschrijving van de compiler-conventies voor een RISC-V functie-oproep kan je vinden in de [RISC-V calling conventions](https://riscv.org/wp-content/uploads/2015/01/riscv-calling.pdf).

* Voeg nu een bestand `user/hello_asm_puts.S` toe. Je kan vertrekken van de volgende assembly-code:

 ```s
.text
.globl main
main:
    #TODO functie-oproep voorbereiden (hint: ra!)
    jal ra, puts # dit springt naar de C-functie puts in user/puts.c
    #TODO return adres ra herstellen
    jr ra

.section .rodata
hello_str: .string "Hello, world!"
```

* Voeg `$U/_hello_asm_puts\` aan [`UPROGS`][UPROGS] in de Makefile
* Implementeer `main`. Denk hierbij aan het bewaren van het return address in `ra` voor je een functie oproept.

[UPROGS]: https://github.com/besturingssystemen/xv6-riscv/blob/2b5934300a404514ee8bb2f91731cd7ec17ea61c/Makefile#L122