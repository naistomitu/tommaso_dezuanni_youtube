# Wireless Comunication

**Tommaso Dezuanni - ER1585**  
*Politechnika Poznanska (a.c. 2023/2024)*

The following code, written in assembly for a ATmega16 microcontroller, is intended to guide remotely a little car via wirlesse connection. The buttons are pressed on the electronic boarda and they are connected to the port A, which works as input for the ATmega16. Each button corresponds to a specific direction, and the specific function of each direction has been provided by the professor in the lab session. The code of the function is then transmitted by a little antenna connected to the UART port, used as output for the ATmega16 in this exercise.  
*Unfortunately, I managed to retrive only two of the direction funtions. In order to show the final result, a link to a youtube video is added at the end of this report.*

```assembly
// INTRODUCTION
.nolist
.include "m16def.inc"
.list
.listmac
.device ATmega16
;----------------------------------
// MANAGEMENT OF THE PROGRAM MEMORY
.cseg
.org 0x0000
jmp 0x0030

.org 0x0030
START:
;----------------------------------
// SETTINGS AND PARAMETERS
; stack setting
ldi r16, high(RAMEND)
out SPH, r16
ldi r16, low(RAMEND)
out SPL, r16

; setting port A as input: button pressed
ldi r16, 0b00000000
out DDRA, r16

; setting UART parameters: transmitter
ldi r16, 8
out UBRRL, r16
ldi r16, 0
out UBRRH, r16

ldi r16,  1<<TXEN
out UCSRB, r16

ldi r16, (1<<URSEL)|(0<<USBS)!(1<<UCSZ1)|(1<<UCSZ0)
out UCSRC, r16
;----------------------------------
// CODE
MAIN:
    in r16, PINA
    call WHICH_BUTTON
    jmp MAIN;
;----------------------------------
//SUBROUTINES

WHICH_BUTTONS:
    cpi r16, 0b11111110
    breq FORWARD
    cpi r16, 0b11111101
    breq BACKWORD
    cpi r16, 0b11111011
    breq RIGHT
    cpi r16, 0b11110111
    breq LEFT
    ret

FORWARD:
    ldi r16, 0xFE
    rcall serout
    ldi r16, 0x0A
    rcall serout
    ldi r16, 0xE7
    rcall serout
    ldi r16, 0xC8
    rcall serout
    ldi r16, 0xFE
    rcall serout
    ldi r16,0x0A
    rcall serout
    ldi r16, 0xE8
    rcall serout
    ldi r16, C8
    rcall serout
    ret

RIGHT:
    ldi r16, 0xFE
    rcall serout
    ldi r16, 0x0A
    rcall serout
    ldi r16, 0xE7
    rcall serout
    ldi r16, 0xC8
    rcall serout
    ldi r16, 0xFE
    rcall serout
    ldi r16,0x0A
    rcall serout
    ldi r16, 0xE8
    rcall serout
    ldi r16, 00
    rcall serout
    ret

SEROUT:
    sbis UCSRA, UDRE
    rjmp SEROUT
    out UDR, r16
ret
```

Some important explanations:  

* for the UART setting we have set the UBRR register according to an oscillation frequency of 8 MHz and a BAUD of 9600
* the TXEN bit set as 1 enables the transmission for ATmega16 to outside devices throughout the UART port. Since we don't use the line to trasnmit data we leve the RXEN set a 0 (default)
* URSEL select UCSRC or UBBRH register: in this case it is set as 1, corresponding to UCSRC
* USBS sat the number of stop bits: 0 means 1 stop bit
* UCSZ0 and UCSZ1 set the first and second bit of the character size
* the *in r16, PINA* instruction makes the ATmega16 read the button pressed, information that arrives from the port A. The information is then saved in the register $r16
* in the WHICH_BUTTONS sub-routine, the *cpi* instruction makes the ATmega16 do a comparison between the value saved in r16 (data coming from Port A) with a binary sequence. If the two values are equal, then *breq* makes the PC jump to the "FORWARD" label
* FORWARD, RIGHT or any additional direction funcion is sent byte by byte to the external device via wireless, passing through the UART port. In the "serout" subroutine *sbis* check if the the UDRE register is set. If it's set the previous trasnmission is not yet concluded, and the next byte must wait. Otherwise, *sbis* makes the PC jump the next line, going directly to the *out* instruction and sending the data to the antenna

The video of the result is available at this link: [youtube](https://youtube.com/shorts/3CmIKYbzuWA?feature=share)
