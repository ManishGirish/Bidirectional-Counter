#Bidirectional Counter :
Project Overview
This project implements a smart entry-exit counter using the LPC2148 microcontroller, two IR sensors, and a 16x2 LCD. It displays the count in real time on the LCD and logs the data to an Excel sheet via serial communication, making it useful for real-time monitoring and record keeping.

 Key Components
Component	Description
LPC2148 MCU	ARM7-based microcontroller
IR Sensors x2	Entry (P0.9) and Exit (P0.10) detection
16x2 LCD	For count display (4-bit mode)
USB to Serial Module	PL2303 / CH340 / FTDI for PC connection
Excel Integration	Using TeraTerm / Python + Excel

