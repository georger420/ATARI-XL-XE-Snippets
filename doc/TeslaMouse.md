## Tesla Mouse Description

The EM 3 WN 16607 mouse is a "classic" professional mouse with three buttons. At the end of its 1.5-meter connection cable, it has a standard 9-pin "joystick-style" connector. The mouse’s electronics are powered by the computer, with a maximum current consumption of 80 mA and a supply voltage of 5V. One of the most important parameters of any mouse is its resolution—for this mouse, the resolution is specified as 0.52 mm per program step.

A critical aspect is how the mouse communicates with the computer. All seven signals from the mouse are transmitted to the computer in parallel, as seen from the connector pin layout:  

| Pin | Signal |
| :-----: | :-----: |
| 1 | Ay |
| 2 | Bx |
| 3 | By |
| 4 | Ax |
| 5 | Middle button |
| 6 | Left button |
| 7 | +5V |
| 8 | GND (Ground) |
| 9 | Right button |  

The **Ax** and **Bx** signals provide information about horizontal position changes, while **Ay** and **By** indicate vertical position changes.

### Connecting the Mouse to ATARI

From the signal layout, it is clear that connecting the mouse to an ATARI computer is as simple as plugging the mouse connector into joystick port 0 or 1. No additional hardware or electronics are required.

### Software Handling

Since ATARI computers do not have built-in routines for mouse support, users must write these routines themselves.

To simply check if the mouse is working, the following BASIC program line can be used:

    10 ? PEEK(53279),STRIG(0),PADDLE(0),PADDLE(1):GOTO10

This line continuously reads signals from the mouse and displays them as numbers. If the mouse is moved very slowly in small increments, the first number on the printed line should change. Pressing the left button should change the second number from 1 to 0. Pressing the middle or right button should change the third or fourth number from 12–13 to 228.

### Advanced Usage and Performance

For serious applications, BASIC alone will not be sufficient. To allow the mouse to move smoothly, signals must be read continuously and very frequently. Considering that the mouse generates approximately two signals per millimeter, and assuming a movement speed of around 25 cm per second, the required reading frequency is:

2 × 250 = 500, meaning at least 500 readings per second.

Signal reading should be regular, which clearly indicates that mouse signals should be processed within an interrupt routine. The most commonly used vertical blank interrupt (VBI) occurs only 50 times per second, making it unsuitable for this purpose. In reference [1], the ATARI ST mouse handling uses an interrupt driven by Timer 1, set to a frequency of 1 kHz. This routine has proven to be highly effective for this mouse as well. Only a few instructions needed modification due to different mouse wiring for the "ST" model.

Using this routine, the cursor can be moved around the screen as a blinking arrow (using PMG for rendering). The assembly source code for this routine is provided below.

### Overview of the Routine
The source code consists of two routines:

RUN - Initializes PMG graphics, sets up Timer 1, and starts interrupt generation.

TIMIRQ - Handles mouse movement, checks valid execution conditions, saves processor registers, and updates the cursor position accordingly.

First, the routine reads the joystick port, filters redundant bits, and determines horizontal movement. If no horizontal movement occurs, an XFLG flag is set, and it skips further horizontal processing. If movement is detected, the routine adjusts the XPOS register by 1 accordingly.

Next, it checks vertical movement. If none is detected, the routine restores processor registers and exits. If vertical movement occurs, the routine updates YPOS, erases the old cursor shape, redraws it one row higher or lower, restores processor registers, and exits.

### Routine Source Code

The source code is in [teslamys-mal.txt](../src/teslamys.mal.txt) file.

### Demonstration Program

Below is a BASIC program demonstrating how the mouse-handling routine can be used for drawing on the screen. Pressing the left mouse button will draw a line, starting from the endpoint of the previous line.

    10 REM **DEMO MYSI **
    20 IF PEEK(28679)<>104 THEN GOSUB 1000
    30 GRAPHICS 7+16:COLOR 1
    40 Q=USR(28679):REM inicializace preruseni
    50 XPOS=28675:YPOS=XPOS+1:REM adresy registru okamzitych souradnic
    60 POKE XPOS,120:POKE YPOS,64:REM pocatecni pozice
    70 PLOT 75,50:REM vykresleni pocatecniho bodu
    100 TRAP 100
    110 IF STRIG(0) THEN 110
    120 X=PEEK(XPOS)-49:Y=PEEK(YPOS)-16
    130 DRAWTO X,Y
    140 GOTO 110
    900 REM 
    1000 RESTORE 1100:S=0:FOR I=28672 TO 28917:READ A:POKE I,A:S=S+A:NEXT I
    1010 IF S<>26174 THEN ? "CHYBA V RADCICH DATA!":STOP 
    1020 RETURN 
    1100 DATA 76,8,112,120,64,0,0,104,120,169,46,141,47
    1110 DATA 2,169,63,141,192,2,169,120,141,0,208,169,112
    1120 DATA 141,7,212,169,2,141,29,208,162,127,169,0,157
    1130 DATA 0,114,202,16,250,162,4,189,241,112,157,64,114
    1140 DATA 202,16,247,169,32,141,0,210,165,16,9,1,133
    1150 DATA 16,141,14,210,169,84,141,16,2,169,112,141,17
    1160 DATA 2,141,9,210,88,96,165,66,208,89,141,9,210
    1170 DATA 138,72,152,72,165,20,41,16,240,12,169,0,133
    1180 DATA 20,173,192,2,73,4,141,192,2,173,0,211,41
    1190 DATA 10,201,10,240,32,174,5,112,240,32,162,0,142
    1200 DATA 5,112,162,1,201,2,208,2,162,255,138,24,109
    1210 DATA 3,112,141,3,112,141,0,208,76,159,112,169,1
    1220 DATA 141,5,112,173,0,211,41,5,201,5,208,11,169
    1230 DATA 1,141,6,112,104,168,104,170,104,64,174,6,112
    1240 DATA 240,245,162,0,142,6,112,162,1,201,4,240,2
    1250 DATA 162,255,138,72,174,4,112,160,4,169,0,157,0
    1260 DATA 114,232,136,16,249,104,24,109,4,112,41,127,141
    1270 DATA 4,112,160,4,174,4,112,185,241,112,157,0,114
    1280 DATA 232,136,16,246,76,173,112,8,144,224,224,240


### Using Other Types of Mouse

The described mouse handling routine works with other types of mice as well. As mentioned earlier, it is compatible with the **ATARI ST** mouse and has been successfully tested with a mouse kit provided by **KLUB 602**. However, modifications to the routine are necessary for both, as each has a different connector wiring.

#### Modification for the ATARI ST Mouse

Below are the lines that need to be changed in the previously mentioned source code:

    2650 AND #3
    2660 CMP #3
    2730 CMP #1
    2740 BEQ LR1
    2850 AND #12
    2860 CMP #12

#### Modification for the KLUB 602 Mouse  

For this mouse, the same changes apply as for the ATARI ST mouse, except for line **2740**, which remains unchanged. Additionally, line **3020** needs to be modified as follows:

    3020 BNE UD1

In reference [3], which is part of the mentioned mouse kit, additional solutions for mouse handling are described, along with methods for adapting existing programs to support mouse control.

### References  

[1] Clausdorf, Dittrich: Tips & Tricks zum ATARI XE/XL. A DATA BECKER BOOK.  
[2] Pavel Dočekal: Memory Addresses for ATARI 600XL/800XL 1 and 2. ČSVTS OKP Liberec.  
[3] Petr Stedina: Electronic Mouse for ATARI Microcomputer. KLUB 602.


