---
title: "ASIC Tamagothi un proyecto para TinyTapeout"
tags: 
- FPGA 
- Verilog 
- Tang Nano 9K 
- ASIC 
- Tamagothi
description: En este post se muestra como generar números "aleatorios" en una FPGA o mejor dicho pseudo-aleatorios.
Categories:
 - Proyectos
date: 2023-9-17 00:00:00+0000
updated: 2023-9-18 00:00:00+0000
---
Para el proyecto personal TinyTapeout 4, diseñé un ASIC Tamagotchi que reside en el chip. Dado el espacio limitado en el chip, este Tamagotchi es simple pero funcional. Además, representa un excelente punto de partida para aprender sobre el diseño de ASIC y explora conceptos interesantes, como la implementación de la comunicación serial UART y la generación de números pseudoaleatorios, que mencioné en un post anterior.

## Acerca del Tamagotchi
El diseño del Tamagotchi es muy sencillo. Consiste en una pequeña memoria que almacena caracteres en formato ASCII para permitir la representación gráfica del Tamagotchi en la consola serial. También incluye una lógica de control basada en máquinas de estados para mostrar diferentes animaciones que representan los estados del Tamagotchi. Además, cuenta con una especie de "barra de necesidades" que muestra sus estados de hambre, sueño y aburrimiento. Estas necesidades aumentan con el tiempo y pueden reducirse mediante la entrada de comandos desde el teclado, correspondientes a cada necesidad. En caso de que alguna de estas necesidades no se satisfaga adecuadamente, el Tamagotchi morirá y será necesario reiniciar el chip para jugar nuevamente, tal como sucede en un Tamagotchi real.

## Diseño 
El corazón del Tamagotchi es, en gran medida, el generador de números pseudoaleatorios. Esto se debe a que controla cómo y cuáles de las estadísticas del Tamagotchi aumentarán de manera aparentemente "aleatoria".

Este diseño, a pesar de su simplicidad, abarca una serie de conceptos y funciones interesantes que hacen que el proyecto sea valioso tanto desde el punto de vista educativo como funcional. Además, el ASIC Tamagotchi en el chip es un proyecto creativo y entretenido que permite explorar la creación de un sistema embebido en un entorno de recursos limitados.

## Generador de números pseudoaleatorios
El generador de números pseudoaleatorios es un Linear Feedback Shift Register (LFSR) de 8 bits. El LFSR es un circuito secuencial que genera una secuencia de bits que se repite después de un cierto número de ciclos de reloj. La secuencia de bits generada por un LFSR parece aleatoria, pero en realidad es determinista y periódica. La longitud del período (el número de ciclos de reloj antes de que la secuencia se repita) depende de cómo se configure el LFSR, específicamente de qué bits se utilizan para la retroalimentación.

```verilog
module random(
    input wire clk,           // Clock input
    input wire rst,           // Reset input
    input wire [7:0] seed,    // 8-bit seed input
    output wire [7:0] rand_out // 8-bit pseudo-random number output
);

reg [7:0] lfsr_reg;          // 8-bit register to hold the LFSR state

always @(posedge clk or posedge rst) begin
    if (rst) begin
        lfsr_reg <= seed; // Initialize the LFSR with the seed
    end else begin
        // XOR feedback taps for an 8-bit LFSR: 8, 6, 5, 4
        lfsr_reg <= {lfsr_reg[6:0], lfsr_reg[7] ^ lfsr_reg[4] ^ lfsr_reg[5] ^ lfsr_reg[3]};
    end
end

assign rand_out = lfsr_reg;

endmodule
```

## Memoria de caracteres
La memoria de caracteres es una memoria ROM de 8 bits que almacena caracteres en formato ASCII para permitir la representación gráfica del Tamagotchi en la consola serial. La memoria de caracteres es una memoria de solo lectura, por lo que los caracteres se cargan en la memoria durante la síntesis. La memoria de caracteres se implementa como una memoria de solo lectura de 8 bits con 256 palabras, donde cada palabra es un carácter ASCII de 8 bits. La memoria de caracteres se carga con los caracteres ASCII que representan las diferentes animaciones del Tamagotchi.

## Comunicación serial UART
La comunicación serial UART se implementa mediante una máquina de estados finitos (FSM) que controla la transmisión y recepción de datos. Use de base la transmisión que implemente en un post anterior 

## Lógica de control
La logica de control es bastante simple en su concepto, para cada necesidad existe un registro que se va decrementando con el tiempo, cuando el registro llega a cero el Tamagotchi muere. Para evitar que el registro llegue a cero se puede ingresar un comando desde el teclado que aumenta el registro.
