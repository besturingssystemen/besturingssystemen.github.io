---
layout: default
title: RISC-V vs DRAMA
nav_order: 1
parent: "System calls vs function calls"
grand_parent: "Zitting 2: System calls"
search_exclude: true
---

Denk terug aan functie-oproepen in DRAMA in het vak SOCS. Conceptueel gezien zijn deze identiek hetzelfde. SOCS gebruikt echter nederlandstalige termen voor vele concepten, dit kan tot verwarring leiden. Hier een snelle cheat sheet:

| DRAMA | RISC V | Naam | Verklaring |
| --- | --- | --- | --- |
| HIA.w reg, val | li reg, val | Load Immediate (immediate=constante) | Laad waarde `val` in register `reg`
| HIA reg1, (reg2)  | ld reg1, (reg2) | Load Double Word (double word=64-bit) | Laad waarde op het adres in `reg2` in register `reg1`
| HIA.a reg, symbol | la reg, symbol | Load Address | Laad het adres van `symbol` in register `reg`
| BIG reg1, (reg2)  | sd reg1, (reg2) | Store Double Word | Bewaar waarde in `reg1` op het adres in `reg2`
| OPT.w reg, val | addi reg, reg, val | Add Immediate | Tel waarde `val` op bij het register `reg`
| OPT dst, src   | add dst, dst, src | Add | dst = dst + src (`dst` en `src` zijn registers)
| SPR label | j label | Jump | Spring naar symbool `label`
| BST value | addi sp, sp, -8 | | Bewaar `value` op de stack (stapel)
|           | sd value, 0(sp) | |
| HST reg   | ld value, 0(sp) | | Haal waarde van de stack en bewaar in `reg`
|           | addi sp, sp, 8  | |
| SBR symbol| jal ra, symbol | Jump And Link (link=sla return adres op) | Schrijf de waarde van de programmateller naar het return adres register (TKA in DRAMA, ra in Risc V) en spring naar `symbol`
| KTG       | jr ra          | Jump Register | Spring naar het adres in het return address register
| N/A       | ecall          | Environment Call | Veroorzaak een trap die naar de trap handler van de kernel springt

Merk op dat de meeste instructies in RISC-V drie operanden hebben.
De eerste is het register waar het resultaat in opgeslagen wordt, de twee laatsten zijn de inputs.
In DRAMA hebben de meeste instructies maar twee operanden en is de eerste zowel de output als één van de inputs.

Verder volgt RISC-V, zoals de naam laat uitschijnen, de [RISC][risc] filosofie.
Dit wilt onder andere zeggen dat de meeste instructies maar één effect hebben.
En DRAMA-instructie zoals `BST`, die een waarde op de stack opslaat _en_ de stack pointer aanpast, zal in RISC-V dus twee instructies nodig hebben.

[risc]: https://en.wikipedia.org/wiki/Reduced_instruction_set_computer