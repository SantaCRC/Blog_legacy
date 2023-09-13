---
title: "Tutorial: Implementar comunicación UART en FPGA"
tags: FPGA Verilog Tang Nano 9K
description: Descubre cómo implementar la comunicación UART en una FPGA
date: 2023-06-06 00:00:00+0000
categories:
    - Tutoriales
tags:
    - FPGA 
    - Verilog 
    - Tang Nano 9K
math: true
---
## Introducción
La comunicación UART es una de las más utilizadas en el mundo de la electrónica, ya que es una de las más sencillas de implementar y es muy útil para la comunicación entre dispositivos. En este tutorial aprenderás a implementar la comunicación UART en una FPGA, utilizando el lenguaje de descripción de hardware Verilog.

<p align="center">
  <img src="https://www.analog.com/-/media/images/analog-dialogue/en/volume-54/number-4/articles/uart-a-hardware-communication-protocol/335962-fig-02.svg?w=435" />
</p>

## ¿Cómo funciona la comunicación UART?
La comunicación UART es una comunicación asíncrona, lo que significa que no se necesita un reloj para sincronizar los datos. En su lugar, se utiliza un bit de inicio y un bit de parada para sincronizar los datos. La comunicación UART se basa en el envío de bytes, que son paquetes de 8 bits (puede variar pero 8 bits es la mas extendida). Cada byte se envía en serie, es decir, un bit a la vez. El bit de inicio es un bit de nivel bajo que indica que se va a enviar un byte. El bit de parada es un bit de nivel alto que indica que se ha terminado de enviar el byte. Entre el bit de inicio y el bit de parada se envían los 8 bits del byte. La siguiente imagen muestra un ejemplo de la comunicación UART.

<p align="center">
  <img src="https://ece353.engr.wisc.edu/wp-content/uploads/sites/607/2014/11/dataFrame-8N1.jpg" />
</p>

## Baudrate
El baudrate es la velocidad a la que se envían los datos. Se mide en baudios, que es el número de bits que se envían por segundo. Por ejemplo, si el baudrate es de 9600 baudios, se envían 9600 bits por segundo. El baudrate se puede calcular con la siguiente fórmula:

$$baudrate = \frac{\text{Frecuencia del reloj}}{\text{divisiones de relog}}$$

El baudrate tiene valores estandar por lo cual lo que se calcula en realidad son las divisiones de reloj, lo cual cambia la formula a la siguiente:

$$\text{divisiones de reloj} = \frac{\text{Frecuencia del reloj}}{\text{baudrate}}$$

Para un reloj de 27 Mhz a un baudrate de 115200, las divisiones de reloj serían:

$$\text{divisiones de reloj} = \frac{27 \text{ Mhz}}{115200} = 234.375$$

Como las divisiones de reloj es un numero entero, el resultado se redondea a 234. Por lo tanto, para un reloj de 27 Mhz a un baudrate de 115200, las divisiones de reloj serían 234.

## Leer datos de la UART
Para leer datos de la UART, se debe leer el bit de inicio, los 8 bits del byte y el bit de parada. Para la lectura y asegurarnos de leer correctamente los datos de la UART, se debe utilizar un reloj que sea 16 veces más rápido que el baudrate. Por ejemplo, si el baudrate es de 115200, el reloj debe ser de 1843200 Hz. Esto se debe a que se deben leer 10 bits (1 bit de inicio, 8 bits del byte y 1 bit de parada) y el reloj debe ser 16 veces más rápido que el baudrate. La lectura no se hace 16 veces por segundo, sino que se lee un bit cada 8 ciclos de reloj para asegurarnos que leemos el dato en la mitad del tiempo que tarda en llegar el siguiente bit. La siguiente imagen muestra un ejemplo de la lectura de datos de la UART.

<p align="center">
  <img src="https://svgshare.com/i/woE.svg" />
</p>


## Escribir datos en la UART
La escritura es simular a la lectura, para escribir datos en la UART, se debe escribir el bit de inicio, los 8 bits del byte y el bit de parada. Para la escritura y asegurarnos de escribir correctamente los datos en la UART, se debe utilizar un reloj que sea 16 veces más rápido que el baudrate. Esto se debe a que se deben escribir 10 bits (1 bit de inicio, 8 bits del byte y 1 bit de parada) al igual que en la lectura. La escritura no se hace 16 veces por segundo, sino que se escribe un bit cada 8 ciclos de reloj para asegurarnos que escribimos el dato en la mitad del tiempo que tarda en llegar el siguiente bit. Ambos procesos (lectura y escritura) se pueden hacer con al mismo tiempo ya que son independientes entre si y por cables diferentes.

## Implementación en Verilog
A continuación se muestra el código en Verilog para implementar la comunicación UART en una FPGA. El código se puede descargar desde [Github](https://github.com/SantaCRC/fpga/tree/main/uart)

### Maquina de estados
Para implementar la comunicación UART vamos a usar dos maquinas de estados, una para la lectura y otra para la escritura. La maquina de estados para la lectura se llama `RX` y la maquina de estados para la escritura se llama `TX`. Ambas maquinas de estados se ejecutan al mismo tiempo ya que son independientes entre si y por cables diferentes.

#### Maquina de estados RX
Esta maquina tienen 5 estados, `IDLE`, `START_BIT`, `READ_WAIT`, `READ` y `STOP_BIT`. El estado `IDLE` es el estado inicial y se queda en este estado hasta que se detecta el bit de inicio. Cuando se detecta el bit de inicio, se pasa al estado `START_BIT` y se empieza a leer los 8 bits del byte, con el estado intermedio `READ_WAIT` que hace la division en 16 partes como explique en la teoría. Cuando se termina de leer los 8 bits del byte, se pasa al estado `STOP_BIT` y se espera a que llegue el bit de parada. Cuando se detecta el bit de parada, se pasa al estado `IDLE` y se termina la lectura del byte. 

> Nota: Usamos el localparam `HALF_DELAY_WAIT` y el `DELAY_FRAMES` para hacer la division en 16 partes. El `DELAY_FRAMES` es el numero de ciclos de reloj que se espera en el estado `READ_WAIT` y el `HALF_DELAY_WAIT` es la mitad del `DELAY_FRAMES`. El `DELAY_FRAMES` se calcula con la formula que mostré en la teoría.

```verilog
localparam RX_STATE_IDLE = 0;
localparam RX_STATE_START_BIT = 1;
localparam RX_STATE_READ_WAIT = 2;
localparam RX_STATE_READ = 3;
localparam RX_STATE_STOP_BIT = 5;

always @(posedge clk) begin
    case (rxState)
        RX_STATE_IDLE: begin
            if (uart_rx == 0) begin
                rxState <= RX_STATE_START_BIT;
                rxCounter <= 1;
                rxBitNumber <= 0;
                byteReady <= 0;
            end
        end 
        RX_STATE_START_BIT: begin
            if (rxCounter == HALF_DELAY_WAIT) begin
                rxState <= RX_STATE_READ_WAIT;
                rxCounter <= 1;
            end else 
                rxCounter <= rxCounter + 1;
        end
        RX_STATE_READ_WAIT: begin
            rxCounter <= rxCounter + 1;
            if ((rxCounter + 1) == DELAY_FRAMES) begin
                rxState <= RX_STATE_READ;
            end
        end
        RX_STATE_READ: begin
            rxCounter <= 1;
            dataIn <= {uart_rx, dataIn[7:1]};
            rxBitNumber <= rxBitNumber + 1;
            if (rxBitNumber == 3'b111)
                rxState <= RX_STATE_STOP_BIT;
            else
                rxState <= RX_STATE_READ_WAIT;
        end
        RX_STATE_STOP_BIT: begin
            rxCounter <= rxCounter + 1;
            if ((rxCounter + 1) == DELAY_FRAMES) begin
                rxState <= RX_STATE_IDLE;
                rxCounter <= 0;
                byteReady <= 1;
            end
        end
    endcase
end
```

#### Maquina de estados TX
Con una aproximación similar realizamos la maquina de estados para la escritura. Esta maquina tienen 5 estados, `IDLE`, `START_BIT`, `WRITE` , `STOP_BIT` y por ultimo `DEBOUNCE` que es el estado diferente, este estado lo que hace es mantener en alto la salida, lo que indica que no se están enviando datos, recordemos que el start bit es un bit de nivel bajo. 

```verilog
localparam TX_STATE_IDLE = 0;
localparam TX_STATE_START_BIT = 1;
localparam TX_STATE_WRITE = 2;
localparam TX_STATE_STOP_BIT = 3;
localparam TX_STATE_DEBOUNCE = 4;

always @(posedge clk) begin
    case (txState)
        TX_STATE_IDLE: begin
            if (btn1 == 0) begin
                txState <= TX_STATE_START_BIT;
                txCounter <= 0;
                txByteCounter <= 0;
            end
            else begin
                txPinRegister <= 1;
            end
        end 
        TX_STATE_START_BIT: begin
            txPinRegister <= 0;
            if ((txCounter + 1) == DELAY_FRAMES) begin
                txState <= TX_STATE_WRITE;
                dataOut <= testMemory[txByteCounter];
                txBitNumber <= 0;
                txCounter <= 0;
            end else 
                txCounter <= txCounter + 1;
        end
        TX_STATE_WRITE: begin
            txPinRegister <= dataOut[txBitNumber];
            if ((txCounter + 1) == DELAY_FRAMES) begin
                if (txBitNumber == 3'b111) begin
                    txState <= TX_STATE_STOP_BIT;
                end else begin
                    txState <= TX_STATE_WRITE;
                    txBitNumber <= txBitNumber + 1;
                end
                txCounter <= 0;
            end else 
                txCounter <= txCounter + 1;
        end
        TX_STATE_STOP_BIT: begin
            txPinRegister <= 1;
            if ((txCounter + 1) == DELAY_FRAMES) begin
                if (txByteCounter == MEMORY_LENGTH - 1) begin
                    txState <= TX_STATE_DEBOUNCE;
                end else begin
                    txByteCounter <= txByteCounter + 1;
                    txState <= TX_STATE_START_BIT;
                end
                txCounter <= 0;
            end else 
                txCounter <= txCounter + 1;
        end
        TX_STATE_DEBOUNCE: begin
            if (txCounter == 23'b111111111111111111) begin
                if (btn1 == 1) 
                    txState <= TX_STATE_IDLE;
            end else
                txCounter <= txCounter + 1;
        end
    endcase      
end
```

### Led de estado
Para verificar que nuestra comunicacion este funcionando para ambos sentidos, debido a que cada cable es independiente, usamos los leds integrados de en mi caso la FPGA Tang Nano 9K. Para esto usamos el siguiente codigo:

```verilog
always @(posedge clk) begin
    if (byteReady) begin
        led <= ~dataIn[5:0];
    end
end
```
> Nota: dataIn es el byte que se recibe de la UART y sale de la maquina de estados RX, para la tang nano 9k el led es activo bajo, por lo cual se debe negar el byte para que se encienda el led.

Para verificar el envio de datos vamos a usar el puerto serial para leer los datos, vamos a crear un arreglo de datos para enviar y lo vamos a enviar cada vez que se presione el boton 1. Para esto usamos el siguiente codigo:

```verilog
reg [3:0] txState = 0;
reg [24:0] txCounter = 0;
reg [7:0] dataOut = 0;
reg txPinRegister = 1;
reg [2:0] txBitNumber = 0;
reg [4:0] txByteCounter = 0;

assign uart_tx = txPinRegister;

localparam MEMORY_LENGTH = 18;
reg [7:0] testMemory [MEMORY_LENGTH-1:0];

initial begin
    testMemory[0] = "F";
    testMemory[1] = "a";
    testMemory[2] = "b";
    testMemory[3] = "i";
    testMemory[4] = "a";
    testMemory[5] = "n";
    testMemory[6] = "a";
    testMemory[7] = "l";
    testMemory[8] = "v";
    testMemory[9] = "a";
    testMemory[10] = "r";
    testMemory[11] = "e";
    testMemory[12] = "z";
    testMemory[13] = ".";
    testMemory[14] = "d";
    testMemory[15] = "e";
    testMemory[16] = "v";
    testMemory[17] = " ";
end
```	
y con esto ya tenemos nuestra comunicación UART implementada en Verilog.

## Pinout de la FPGA
En una FPGA cualquier pin sirve como `RX` y `TX`, en el caso de la Tang Nano 9K tiene una conexion directa con el puerto usb, por lo cual se puede conectar directamente a la computadora y usar el puerto serial para leer y escribir datos. En el caso de la Tang Nano 9K, el pin `RX` es el pin `18` y el pin `TX` es el pin `17`. La siguiente imagen muestra el pinout de la Tang Nano 9K.
![Pinout de la Tang Nano 9K](https://imgur.com/cYhpNUv.png)

## Conclusión
En este tutorial, has aprendido a implementar la comunicación UART en una FPGA utilizando el lenguaje de descripción de hardware Verilog. Comprendes cómo funciona la comunicación UART, cómo leer y escribir datos, y cómo implementarla en Verilog. Además, has aprendido a utilizar los LEDs integrados en la FPGA para verificar el funcionamiento de la comunicación. Si tienes alguna pregunta, no dudes en dejar un comentario, y estaré encantado de ayudarte a resolverla. ¡Buena suerte en tu proyecto de FPGA!

## Codigo completo
```verilog
`default_nettype none

module top
#(
    parameter DELAY_FRAMES = 234 // 27,000,000 (27Mhz) / 115200 Baud rate
)
(
    input  clk,
    input  uart_rx,
    output uart_tx,
    output reg [5:0] led,
    input btn1
);

localparam HALF_DELAY_WAIT = (DELAY_FRAMES / 2);

reg [3:0] rxState = 0;
reg [12:0] rxCounter = 0;
reg [7:0] dataIn = 0;
reg [2:0] rxBitNumber = 0;
reg byteReady = 0;

localparam RX_STATE_IDLE = 0;
localparam RX_STATE_START_BIT = 1;
localparam RX_STATE_READ_WAIT = 2;
localparam RX_STATE_READ = 3;
localparam RX_STATE_STOP_BIT = 5;

always @(posedge clk) begin
    case (rxState)
        RX_STATE_IDLE: begin
            if (uart_rx == 0) begin
                rxState <= RX_STATE_START_BIT;
                rxCounter <= 1;
                rxBitNumber <= 0;
                byteReady <= 0;
            end
        end 
        RX_STATE_START_BIT: begin
            if (rxCounter == HALF_DELAY_WAIT) begin
                rxState <= RX_STATE_READ_WAIT;
                rxCounter <= 1;
            end else 
                rxCounter <= rxCounter + 1;
        end
        RX_STATE_READ_WAIT: begin
            rxCounter <= rxCounter + 1;
            if ((rxCounter + 1) == DELAY_FRAMES) begin
                rxState <= RX_STATE_READ;
            end
        end
        RX_STATE_READ: begin
            rxCounter <= 1;
            dataIn <= {uart_rx, dataIn[7:1]};
            rxBitNumber <= rxBitNumber + 1;
            if (rxBitNumber == 3'b111)
                rxState <= RX_STATE_STOP_BIT;
            else
                rxState <= RX_STATE_READ_WAIT;
        end
        RX_STATE_STOP_BIT: begin
            rxCounter <= rxCounter + 1;
            if ((rxCounter + 1) == DELAY_FRAMES) begin
                rxState <= RX_STATE_IDLE;
                rxCounter <= 0;
                byteReady <= 1;
            end
        end
    endcase
end

always @(posedge clk) begin
    if (byteReady) begin
        led <= ~dataIn[5:0];
    end
end

reg [3:0] txState = 0;
reg [24:0] txCounter = 0;
reg [7:0] dataOut = 0;
reg txPinRegister = 1;
reg [2:0] txBitNumber = 0;
reg [4:0] txByteCounter = 0;

assign uart_tx = txPinRegister;

localparam MEMORY_LENGTH = 18;
reg [7:0] testMemory [MEMORY_LENGTH-1:0];

initial begin
    testMemory[0] = "F";
    testMemory[1] = "a";
    testMemory[2] = "b";
    testMemory[3] = "i";
    testMemory[4] = "a";
    testMemory[5] = "n";
    testMemory[6] = "a";
    testMemory[7] = "l";
    testMemory[8] = "v";
    testMemory[9] = "a";
    testMemory[10] = "r";
    testMemory[11] = "e";
    testMemory[12] = "z";
    testMemory[13] = ".";
    testMemory[14] = "d";
    testMemory[15] = "e";
    testMemory[16] = "v";
    testMemory[17] = " ";
end

localparam TX_STATE_IDLE = 0;
localparam TX_STATE_START_BIT = 1;
localparam TX_STATE_WRITE = 2;
localparam TX_STATE_STOP_BIT = 3;
localparam TX_STATE_DEBOUNCE = 4;

always @(posedge clk) begin
    case (txState)
        TX_STATE_IDLE: begin
            if (btn1 == 0) begin
                txState <= TX_STATE_START_BIT;
                txCounter <= 0;
                txByteCounter <= 0;
            end
            else begin
                txPinRegister <= 1;
            end
        end 
        TX_STATE_START_BIT: begin
            txPinRegister <= 0;
            if ((txCounter + 1) == DELAY_FRAMES) begin
                txState <= TX_STATE_WRITE;
                dataOut <= testMemory[txByteCounter];
                txBitNumber <= 0;
                txCounter <= 0;
            end else 
                txCounter <= txCounter + 1;
        end
        TX_STATE_WRITE: begin
            txPinRegister <= dataOut[txBitNumber];
            if ((txCounter + 1) == DELAY_FRAMES) begin
                if (txBitNumber == 3'b111) begin
                    txState <= TX_STATE_STOP_BIT;
                end else begin
                    txState <= TX_STATE_WRITE;
                    txBitNumber <= txBitNumber + 1;
                end
                txCounter <= 0;
            end else 
                txCounter <= txCounter + 1;
        end
        TX_STATE_STOP_BIT: begin
            txPinRegister <= 1;
            if ((txCounter + 1) == DELAY_FRAMES) begin
                if (txByteCounter == MEMORY_LENGTH - 1) begin
                    txState <= TX_STATE_DEBOUNCE;
                end else begin
                    txByteCounter <= txByteCounter + 1;
                    txState <= TX_STATE_START_BIT;
                end
                txCounter <= 0;
            end else 
                txCounter <= txCounter + 1;
        end
        TX_STATE_DEBOUNCE: begin
            if (txCounter == 23'b111111111111111111) begin
                if (btn1 == 1) 
                    txState <= TX_STATE_IDLE;
            end else
                txCounter <= txCounter + 1;
        end
    endcase      
end
endmodule
```
