# BOOT
This repository was create to follow the content of the course 'STM32Fx Microcontroller Custom Bootloader Development'.

* Board: NUCLEO-F401RE
* MCU: STM32F401RET6U

## Memory address

* Internal flash memory - its where the code will be placed - Begins 0x0800 0000, divided in sectors and goes up to 0x0803 FFFF.
* Internal SRAM1 - used for global data, static variables, stack and heap purposes - Begins 0x2000 0000 and goes up to 0x2000 FFFF.
* System Memory (ROM) - Bootloader - Begins 0x1FFF 0000 and goes up to 0x1FFF 77FF.


## Reset sequence

* 1. When you reset the MCU, the PC of the processor is loaded with the value 0x0000 0000. *In STM32 uC, is loaded with the value 0x0800 0000*.
* 2. The processor reads the value at memory location 0x0000 0000 in to MSP (Main Stack Pointer) register. *In STM32 uC, MSP value stored at 0x0800 0000*.
* 3. Then the processor reads the value at 0x0000 0004 (value is the address of the reset handler). *In STM32 uC, processor reads the value of the address of the reset handler found at 0x0800 0004*.
* 4. PC jumps to the reset handler. *In STM32 uC, jumps to the reset handler*.

Both addresses can be linked by doing the "memory aliasing" (address value depends on MCU).

# Hardware/Firmware configuration

The documentation (reference manual) shows the options for Boot modes.

BOOT1 (PB2) and BOOT0 (B) pins.

![image](https://user-images.githubusercontent.com/58916022/215750510-5e7f53b9-b6fa-48eb-8fab-75626859e259.png)

*NOTE: When the device boots from SRAM, in the application initialization code, you have to relocate the vector table in SRAM using the NVIC exception table and the offset register.*

![image](https://user-images.githubusercontent.com/58916022/215751604-6c1ad05a-fc80-4793-9237-13cbf05ef020.png)

As we can see, UART2 isn't connected by PA2 and PA3. We canno't talk to ST's Bootloader over this Virtual COM Port. 
![image](https://user-images.githubusercontent.com/58916022/215752046-67a9fe86-0e1d-4277-a0a2-06d4e1ca1d89.png)

USART2 is pointed as a boot option in user manual, but on AN2606 USART3 appears instead of USART2. For some reason, I try to connect to USART3 (PC10 and PC11) and it didn't worked.

![image](https://user-images.githubusercontent.com/58916022/215753204-60011869-849e-48f3-bd34-ebdbff31d5a6.png)

PA9 and PA10 from USART1 will be used instead. We need a USB-UART conversor (FTDI Adapter FT232RL [YP-05] borrowed from a friend). 

![image](https://user-images.githubusercontent.com/58916022/215753774-3c02b32c-5038-46a1-b566-decf679737c4.png)

Also, *REMEMBER THAT RX IS CONNECTED TO TX*.

![image](https://user-images.githubusercontent.com/58916022/217532938-d5bea1b7-e99e-4b04-96d1-d54a6c360a1e.png)

BOOT0 pin has to be connected to VCC during board reset to enter in to system memory boot. BOOT1 has to be connected to GND. Do that and press the Reset button.

The BOOT0 (pin 7 from CN7 connector) can be connected to pin 5 of CN7 connector or pins 2 or 4 from CN6 connector.

The BOOT1 (pin 22 from CN10 connector) can be connected to pin 20 of CN10 connector.

# Software

The [FLASHER-STM32](https://www.st.com/en/development-tools/flasher-stm32.html) is required. ST website says that the FLASHER-STM32 was replaced by STM32CubeProgrammer. So, we also downloaded the [STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html) for tests purpose.

After install Flasher-STM32, for some reason, the application is found by searching for 'Demonstrator GUI'. Connect the USB-Serial board and open the software.

![image](https://user-images.githubusercontent.com/58916022/217073929-7d84314d-773e-4547-a9cb-97ea3e2f38b5.png)

To check which port the USB-Serial is connected, right click in 'This PC' -> 'Manage' and then in 'Device Manager' you will be able to see all drivers connected to the 'Ports (COM & LPT)'. COM4 is our USB-Serial and COM3 the Nucleo board.

![image](https://user-images.githubusercontent.com/58916022/217075125-e73eac49-39f2-4384-a443-8c1a92bcb21f.png)

Select the COMx of the USB-Serial and click 'Next'.

![image](https://user-images.githubusercontent.com/58916022/217076325-f535f212-3b0a-4920-89ee-9047b7c00dbf.png)

The following message will appear:

![image](https://user-images.githubusercontent.com/58916022/217076524-72fa155b-98c4-4859-affa-353e0e3e8bb0.png)

Then, I press the Reset button and click 'Next' again.

![image](https://user-images.githubusercontent.com/58916022/217533237-45a2bff6-388d-486e-a7e5-db1bb29d0222.png)

I Open the STM32CubeIDE and compile an older code (from other classes) that blinks the LD2 in order to generate the *.hex* file. 

I copy the following line:

```
arm-none-eabi-objcopy -O ihex ${ProjName}.elf ${ProjName}.hex
```

To project properties:

![image](https://user-images.githubusercontent.com/58916022/217535584-f3796bb1-2daa-4fcc-a79d-63015708bd51.png)

And then compiled:

![image](https://user-images.githubusercontent.com/58916022/217535851-9b6b915a-004d-47e8-b6a2-8231e7b467c9.png)

Clicked 'Next' in the demonstrator software:

![image](https://user-images.githubusercontent.com/58916022/217536011-802e9879-afe1-47c7-8018-fc59935779ed.png)

Tick the option 'Download to device', and selected the file by clicking the three dots. **Don't forget to tick the option 'Jump to the user program' or else the code will always enterm on system bootloader mode**. Then just clicked to 'Next':

![image](https://user-images.githubusercontent.com/58916022/217540335-1e8ec1cf-4f3f-42a6-b820-c53f261ff4cf.png)

Demonstrator will start to download the data.

![image](https://user-images.githubusercontent.com/58916022/217536744-256d8325-bc81-40f2-bb80-ae298ab26fea.png)

And to finish just click 'Close'.

![image](https://user-images.githubusercontent.com/58916022/217536924-2fa3b47b-c65e-4a31-9512-7fc68660f7b8.png)

Imediatlly after downloading the .hex file to the NUCLEO board, the LD2 lid. If I click on the Reset button, LD2 becomes off, since the NUCLEO board enter in system bootloader again.
