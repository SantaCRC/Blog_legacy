---
title: "Tutorial: Encendiendo un LED con la placa de desarrollo Tang Nano 9K"
description: Descubre cómo encender un LED utilizando la placa de desarrollo Tang Nano 9K de Sipeed
slug: Tang-nano-primero-pasos
date: 2022-03-06 00:00:00+0000
image: cover.jpg
categories:
    - Tutoriales
tags:
    - FPGA 
    - Verilog 
    - Tang Nano 9K
weight: 1       # You can add weight to some posts to override the default sorting (date descending)
---

La Tang Nano es una serie de placas de desarrollo FPGA desarrolladas por Sipeed, basadas en las FPGAs de Gowin, una empresa china reconocida por su bajo costo y alto rendimiento. En este tutorial, te mostraré cómo encender un LED utilizando la placa Tang Nano 9K, que cuenta con una FPGA GW1N-9K de Gowin, 9K LUTs, 240Kb de RAM, 128Kb de ROM, 2 PLLs y 2 SPIs. A través del IDE de Gowin, aprenderás a programar la placa y realizar una conexión sencilla para lograr este objetivo.

Pasos para encender un LED con la placa Tang Nano 9K:

1. Descarga e instala el IDE de Gowin:
   - Visita la [página web de Gowin](https://www.gowinsemi.com/en/support/home/) para descargar el IDE.
   - La versión educativa es gratuita, pero también puedes solicitar una licencia gratuita de la versión profesional completando un registro.

2. Crea un nuevo proyecto en el IDE:
   - Abre el IDE de Gowin y selecciona la pestaña "File".
   - Haz clic en "New Project" y elige un nombre y una ubicación para tu proyecto, como "LED".
   - Asegúrate de guardar el proyecto en la carpeta de proyectos de Gowin.

3. Selecciona el dispositivo adecuado:
   - Al crear el proyecto, debes seleccionar el dispositivo que corresponda a tu versión de la placa Tang Nano.
   - Para la Tang Nano 9K con FPGA GW1N-LV9QN48C6/I5, elige la opción correspondiente en el IDE.

4. Crea un nuevo archivo Verilog:
   - Haz clic en la pestaña "File" y selecciona "New File".
   - Nombre del archivo: "LED.v".
   - Guarda el archivo en la carpeta del proyecto.

5. Escribe el código Verilog para encender el LED:
   - Abre el archivo "LED.v" y escribe el siguiente código:

```verilog
module LED(
    input btn,
    output reg led
);

    always @(posedge btn) begin
        led <= ~led;
    end

endmodule
```

6. Crea el archivo de mapeo de pines:
   - Utiliza el FloorPlanner del IDE para crear el archivo de mapeo de pines.
   - Ve a la pestaña "Tools" y selecciona "FloorPlanner".
   - Para el botón 1 es el pin 4 y el led 1 es el pin 10, esto se puede ver en el [esquemático de la placa](https://dl.sipeed.com/shareURL/TANG/Nano%209K/2_Schematic).

7. Programa la FPGA:
   - Ve a la pestaña "Tools" y selecciona "Program Device".
   - Sigue las instrucciones para subir el programa a la FPGA.

8. Observa el resultado:
   - Una vez que el programa esté cargado en la FPGA, el LED 1 de la placa Tang Nano 9K se encenderá y apagará cada vez que presiones el botón 1.

Ten en cuenta que los botones y LEDs en esta FPGA tienen un comportamiento inverso, lo que significa que el valor es 0 cuando se presiona un botón y 1 cuando no se presiona. Del mismo modo, cuando el valor es 0, el LED se enciende, y cuando el valor es 1, el LED se apaga. Para ajustar esto, se ha realizado una modificación en el código Verilog invirtiendo el valor predeterminado del LED.

```verilog	
module LED(
    input btn,
    output reg led = 1
);

    always @(posedge btn) begin
        led <= ~led;
    end
endmodule
```

Para los mas técnicos las configuración de los botones de la placa Tang Nano 9K están en pull-up y los leds se encuentran en ánodo común, esto se ve en el esquema de la placa:

![picture 1](https://i.imgur.com/6VTTzwh.png)  

![picture 2](https://i.imgur.com/08mRW9q.png)  



Con este tutorial, has aprendido cómo encender un LED utilizando la placa de desarrollo Tang Nano 9K de Sipeed. También has descubierto una característica importante de esta placa, en la que el funcionamiento de los LEDs integrados y los botones está invertido en comparación con la mayoría de las placas de desarrollo. ¡Explora más posibilidades y sigue experimentando con esta potente herramienta de desarrollo FPGA!