---
title: "Tutorial: Como generar números aleatorios en una FPGA"
tags: 
- FPGA 
- Verilog 
- Tang Nano 9K
description: En este post se muestra como generar números "aleatorios" en una FPGA o mejor dicho pseudo-aleatorios.
categories:
- Tutoriales
date: 2023-20-8 00:00:00+0000
---
Generar números aleatorios es imposible, por lo menos en el ámbito computacional, claramente en muchos lenguajes de programación existen las funciones random, lo que hacen estas funciones es generar números pseudo-aleatorios y eso es lo que vamos a hacer

En verilog también existe una función para generar números pseudo-aleatorios, la función es `$random`, esta función solo es para simulación no para implementación, por lo que no nos sirve para lo que queremos hacer.
{:.warning}

Derivando tiene un video que lo explica mucho mejor.
<div>{%- include extensions/youtube.html id='RzEjqJHW-NU' -%}</div>

Para generar los números pseudo-aleatorios vamos a usar un linear-feedback shift register o LFSR, para la generación se usan los siguientes pasos:

1. **Registro de Desplazamiento**: Un LFSR consta de un registro de desplazamiento que contiene una serie de bits. Los bits se desplazan hacia la derecha (o izquierda) en cada ciclo de reloj.

2. **Retroalimentación Lineal**: La característica principal de un LFSR es que algunos de los bits del registro se combinan mediante operaciones lógicas (generalmente XOR) y se retroalimentan en la entrada. Estos bits se denominan "taps" o "retroalimentaciones".

3. **Ciclo de Reloj**: En cada ciclo de reloj, todos los bits en el registro se desplazan hacia la derecha (o izquierda) en una posición. El valor del nuevo bit de entrada (el que ingresa en el registro) se calcula a partir de la retroalimentación de los bits existentes en el registro según las reglas de retroalimentación definidas. Esto es lo que hace que el registro de desplazamiento sea "lineal".

4. **Secuencia Pseudoaleatoria**: La secuencia de bits generada por un LFSR parece aleatoria, pero en realidad es determinista y periódica. La longitud del período (el número de ciclos de reloj antes de que la secuencia se repita) depende de cómo se configure el LFSR, específicamente de qué bits se utilizan para la retroalimentación.

5. **Salida**: Puedes considerar la salida del LFSR como el bit más a la derecha en el registro de desplazamiento. Este bit se toma como la salida de la secuencia pseudoaleatoria. La secuencia de bits en esta salida se repite después de un cierto número de ciclos, pero puede parecer muy aleatoria durante ese período.

<p align="center">
  <img src="https://upload.wikimedia.org/wikipedia/commons/9/99/Lfsr.gif" />
</p>

## Implementación
Para implementar el LFSR vamos a usar el siguiente código:
```verilog
module random(
    input wire clk,           // Clock input
    input wire rst,           // Reset input
    output wire [7:0] rand_out // 8-bit pseudo-random number output
);

reg [7:0] lfsr_reg;          // 8-bit register to hold the LFSR state

always @(posedge clk or posedge rst) begin
    if (rst) begin
        lfsr_reg <= 8'b1111_1111; // Initialize LFSR with all ones
    end else begin
        // XOR feedback taps for an 8-bit LFSR: 8, 6, 5, 4
        lfsr_reg <= {lfsr_reg[6:0], lfsr_reg[7] ^ lfsr_reg[4] ^ lfsr_reg[5] ^ lfsr_reg[3]};
    end
end

assign rand_out = lfsr_reg;

endmodule
```
Este código es para un LFSR de 8 bits, si quieres un LFSR de 16 bits solo tienes que cambiar el tamaño del registro y los taps, para un LFSR de 16 bits los taps son 16, 14, 13, 11.

El código es bastante simple, se compone de en realidad solo dos operaciones el desplazamiento y el XOR, el desplazamiento se hace con `{lfsr_reg[6:0], lfsr_reg[7]}` y el XOR con `lfsr_reg[7] ^ lfsr_reg[4] ^ lfsr_reg[5] ^ lfsr_reg[3]`. También tiene el reset que inicializa el registro con todos unos

El valor de reset sirve también como semilla de nuestro generador de números pseudo-aleatorios
{:.info}