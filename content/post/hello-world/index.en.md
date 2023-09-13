---
title: "Tutorial: Encendiendo un LED con la placa de desarrollo Tang Nano 9K"
description: Descubre cómo encender un LED utilizando la placa de desarrollo Tang Nano 9K de Sipeed
slug: Tang-nano-primero-pasos
date: 2023-03-06 00:00:00+0000
image: cover.jpg
categories:
    - Tutoriales
tags:
    - FPGA 
    - Verilog 
    - Tang Nano 9K
weight: 1       # You can add weight to some posts to override the default sorting (date descending)
---
The Tang Nano is a series of FPGA development boards developed by Sipeed, based on FPGAs from Gowin, a Chinese company known for their low cost and high performance. In this tutorial, I will show you how to light an LED using the Tang Nano 9K board, which features a Gowin GW1N-9K FPGA, 9K LUTs, 240Kb RAM, 128Kb ROM, 2 PLLs and 2 SPIs. Through the Gowin IDE, you will learn how to program the board and make a simple connection to achieve this goal.

Steps to light an LED with the Tang Nano 9K board:

1. Download and install the Gowin IDE:
   - Visit the [Gowin website](https://www.gowinsemi.com/en/support/home/) to download the IDE.
   - The educational version is free, but you can also request a free license for the professional version by completing a registration.

2. Create a new project in the IDE:
   - Open the Gowin IDE and select the "File" tab.
   - Click "New Project" and choose a name and location for your project, such as "LED".
   - Make sure to save the project in the Gowin project folder.

Select the appropriate device:
   - When creating the project, you must select the device that corresponds to your version of the Tang Nano board.
   - For the Tang Nano 9K with FPGA GW1N-LV9QN48C6/I5, choose the corresponding option in the IDE.

4. Create a new Verilog file:
   - Click on the "File" tab and select "New File".
   - File name: "LED.v".
   - Save the file in the project folder.

5. Write the Verilog code to turn on the LED:
   - Open the file "LED.v" and write the following code:

````verilog
module LED(
    input btn,
    output reg led
);

    always @(posedge btn) begin
        led <= ~led;
    end

endmodule
```

6. Create the pin mapping file:
   - Use the IDE's FloorPlanner to create the pin mapping file.
   - Go to the "Tools" tab and select "FloorPlanner".
   - For button 1 is pin 4 and led 1 is pin 10, this can be seen in the [board schematic](https://dl.sipeed.com/shareURL/TANG/Nano%209K/2_Schematic).

7. Program the FPGA:
   - Go to the "Tools" tab and select "Program Device".
   - Follow the instructions to upload the program to the FPGA.

8. Observe the result:
   - Once the program is uploaded to the FPGA, LED 1 on the Tang Nano 9K board will turn on and off each time you press button 1.

Note that the buttons and LEDs on this FPGA have inverse behavior, meaning that the value is 0 when a button is pressed and 1 when it is not pressed. Similarly, when the value is 0, the LED is on, and when the value is 1, the LED is off. To adjust for this, a modification has been made to the Verilog code by inverting the default value of the LED.

````verilog	
module LED(
    input btn,
    output reg led = 1
);

    always @(posedge btn) begin
        led <= ~led;
    end
endmodule
```

For the more technical people the button configuration of the Tang Nano 9K board is in pull-up and the leds are in common anode, this is shown in the board schematic:

![picture 1](https://i.imgur.com/6VTTzwh.png)  

![picture 2](https://i.imgur.com/08mRW9q.png)  



With this tutorial, you have learned how to light an LED using Sipeed's Tang Nano 9K development board. You have also discovered an important feature of this board, where the operation of the integrated LEDs and buttons is reversed compared to most development boards. Explore more possibilities and keep experimenting with this powerful FPGA development tool!