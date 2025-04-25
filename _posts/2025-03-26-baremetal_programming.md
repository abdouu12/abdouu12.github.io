---
title: "Starting From Nothing: STM32 Bare-Metal Without HAL or CubeMX"
date: 2025-04-25 12:07:00 +0000
categories: [Embedded Systems, STM32, Bare-Metal]
tags: [stm32, bare-metal, arm cortex, linker script, startup code, assembly, embedded c]
published: true
---

I’ve done a bunch of embedded projects — the latest was an RC car controlled over Wi-Fi using an ESP32. Sounds cool, I know. And yeah, it was fun. My friends were blown away watching it zip around. But when I sat down to write a blog post about it… I had nothing to say.

I mean, it wasn’t that complex.

Any hobbyist can throw together some wires, connect a few modules, and use the HAL (Hardware Abstraction Layer) to get things running. The code barely crossed 50 lines. Turning a motor on? Just digitalWrite(pin, HIGH) and it’s done.

It worked — but I had no real idea how. I couldn’t explain what was happening under the hood. And that’s when impostor syndrome hit me like a brick.

So yeah — I realized the solution wasn’t another flashy project.
I needed something harder, something that would actually teach me something real. Something that would set me apart from your average Arduino or ESP32 hobbyist.

I needed to go bare-metal.

No libraries. No HAL. No helper functions. Just me, the registers, and the datasheet.

And what better way to start my journey toward becoming a real embedded systems engineer…
than blinking an LED — from scratch.

# Requirements:

`-An MCU:`
This was my first time doing baremetal programming. I used STM32 Nucleo F446 because it was the most complex MCU(microcontroller) I had.You don’t have to use the one I used as this will work on any MCU.

`-Datasheet and reference manual:`
In baremetal programming you will manipulate the registers and interact with the memory directly so the datasheet and reference manual are a must. Besides learning to read directly from the datasheet and reference manual is a skill every embedded systems engineer needs to be familiar with. Good news is that these are available for free on the internet and a simple google search will do.

`-Compiler:`
Since STM32 uses ARM processors (the F446 runs a Cortex-M4), you need an ARM compiler. I used arm-none-eabi-gcc, which is free and widely supported.

`-Text Editor / IDE:`
You don’t need a full IDE, but you’ll want a solid editor. I used VS Code with the ARM toolchain and Cortex-Debug extension. You could also use something like CLion, Sublime, or just a terminal setup with make.

`-Makefile:`
Optional, but useful. Instead of manually compiling with arm-none-eabi-gcc every time, a simple Makefile saves time and lets you control every step of the build.

`-A flashing tool:`
You need a way to actually put your compiled code onto the MCU. Most Nucleo boards have an onboard ST-Link debugger, so you can use STM32CubeProgrammer, ST-Link Utility, or OpenOCD. If you're using VS Code, the Cortex-Debug extension can handle flashing through ST-Link too.


# The backbone:

Alright now that we got everything ready, what files do we need? 

- `main.c`: where the LED blink logic lives.  
- `stm32f446.h` *(optional)*: a custom header with register definitions.  
- `startup.S`: defines the vector table and reset handler.  
- `linker.ld`: linker script; tells the compiler where to place stuff in memory.  
- `Makefile` *(optional)*: automates the build process so you don’t have to type `arm-none-eabi-gcc` a thousand times.

These are the minimum files we need in order to make thins blinky work.

# Principles:

If you already know this stuff, feel free to skip to the code.
I’m going to assume you don’t, which is why I’ll explain a few core concepts and syntax patterns that show up everywhere in embedded C.
(Fair warning — this part’s a bit long)

## Bitwise sytnax:
In embedded C, bitwise operations are everywhere. You use them to control specific bits in a register — like turning on a pin, enabling a clock— without touching the rest of the register.

Here are the basics:

& – bitwise AND
Used to clear specific bits
Example: reg &= ~(1 << n); clears bit n

| – bitwise OR
Used to set specific bits
Example: reg |= (1 << n); sets bit n

^ – bitwise XOR
Used to toggle bits
Example: reg ^= (1 << n); flips bit n

~ – bitwise NOT
Inverts all bits
Useful for clearing: ~(1 << n) creates a mask where only bit n is 0

<< – left shift
Moves a bit to the left
Example: 1 << 5 gives you 0b100000, which is bit 5 set

You’ll use these constantly when working with registers. Almost every GPIO setup line is some combination of &= ~() and |=

## Pointers:
If you want to store a bunch of toys, you use a drawer. You’ve just *allocated* a chunk of space and filled it with stuff. Then you label that drawer with a number so you don’t forget where it is.

That label — the number — is basically what a pointer is.

Pointers are essential when working with microcontrollers because most of the time, you’re not dealing with regular variables — you’re reading from or writing to specific memory addresses that control hardware. GPIO, timers, UART — all of it happens through pointers.

## Memory-Mapped Registers:
The STM32 nucleo is a 32 bit microcontroller, it has 32 bit registers, a register is simply a drawer where you can sore infomration that information is in 0s and 1s,  the MCU comes with dozens and dozens of these registers, each register serves a unique function and we'll explore some of them here, let's start with the General Purpose Input Output Registers.

### GPIOR:
GPIOs are the pins on the MCU. They serve as the bridge between external electronics and the microcontroller. Whether it’s an LED, a button, or a sensor — GPIOs are how you connect the outside world to your code.

#### GPIOx_MODER:
This register is used to set the mode (or state) of each GPIO pin. Each pin gets 2 bits, and there are 4 possible modes:

00: Input – reads the pin. Used for buttons, switches, sensors, etc.

01: Output – manually drives the pin by setting its voltage level (HIGH or LOW).

10: Alternate – the pin is controlled by another peripheral like UART, SPI, or PWM. You don’t set or read it manually.

11: Analog – used for analog input (ADC), or to reduce power consumption by disconnecting digital logic.

#### GPIOx_ODR:
If you set the MODER register to output then you will use this register to write values.
example:
```cpp
GPIOA->ODR |= (1 << 5);
```
The equivalent of this in HAL is HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_SET);

#### GPIOx_IDR:
Same as ODR but instead of writing values it reads values and obviously we need MODER to be set to input. 

#### GPIOx_BSRR:
Stands for Bit Set/Reset Register it does exactly what it stands for it sets bit or flips them meaning it gives HIGH or LOW values to pins, but it has an advantage over ODR which is speed. Unlike ODR it doesn't involve the read-modify -write process, it's direct hardware write and it doesn't risk affecting any other pin.

### RCC:
RCC stands for **Reset and Clock Control** and it mainly manages these 3 things:
-Which peripherals get power
-What speed the system runs at
-Resetting peripherals when needed

Before we can use any GPIO pin we have to enable the **clock**. In STM32 every peripheral (UART, SPI, timers, etc) is connected to a bus and each bus is fed by a clock domain. If the clock to a peripheral is off it won't do anything, even if you write it to registers.



# The code:

We understood the basics for now, we'll run into more concepts I will have to explain once we get to the startup code and linker script but for now we have what we need to write the **main.c**.

To help with this, it’s always good practice to use a header file to store all of our structs.
(I’m going to assume you’re familiar with C structs.)

Since we’re trying to blink PA5, we’ll define a struct specifically for GPIOA — which is the port that pin 5 belongs to.

This struct will contain all the registers that GPIOA exposes — MODER, ODR, IDR, BSRR, and so on.
The variables inside the struct are unsigned 32-bit integers, and we assign them using the uint32_t type.

Why do we put volatile before the variable type?
That's because of how the compiler behaves. If we don't put volatile, the compiler will try to optimize it — meaning it might cache the value or skip reading/writing it properly, which would break everything when working with hardware.
```cpp
typedef struct {
    volatile uint32_t MODER;
    volatile uint32_t OTYPER;
    volatile uint32_t OSPEEDR;
    volatile uint32_t PUPDR;
    volatile uint32_t IDR;
    volatile uint32_t ODR;
    volatile uint32_t BSRR;
    volatile uint32_t LCKR;
    volatile uint32_t AFRL;
    volatile uint32_t AFRH;
} GPIOA_def;

```
The other registers aren’t important for the blinky, but they’ll become important later when we make things more complex.

Once we define the struct, we need to define a macro.
The type will be a pointer to our struct, and we’ll give it the address of GPIOA.

At this point, we have to check the datasheet for guidance to find the base address of GPIOA.

![GPIOA base address](/assets/images/memorymap.png)

This is the **memory map**
Under it, you can find a table that lists the addresses of all the elements inside the STM32 — like GPIOs, timers, USARTs, and more.

![GPIOA base address](/assets/images/gpioamemeorymap.png)

Hence we will write it the following way:
```cpp
#define GPIOA ((GPIOA_def*) 0x40020000)
```
Moving on to the clock, just like we defined the gpio struct, we’ll do the same for RCC.

If you check the **block diagram** in the datasheet, you’ll see gpioa sits on the ahb1 bus.

We’ll define the other buses too, but for now we only care about ahb1.
![GPIOA base address](/assets/images/block.png)

Hence the code will be like the following:
```cpp
typedef struct {
    volatile uint32_t CR;
    volatile uint32_t PLLCFGR;
    volatile uint32_t CFGR;
    volatile uint32_t CIR;
    volatile uint32_t AHB1RSTR;
    volatile uint32_t AHB2RSTR;
    volatile uint32_t AHB3RSTR;
    uint32_t RESERVED0;
    volatile uint32_t APB1RSTR;
    volatile uint32_t APB2RSTR;
    uint32_t RESERVED1[2];
    volatile uint32_t AHB1ENR;
} RCC_def;
```
And we will reference the datasheet again to see what the base address of the RCC is.
![RCC base address](/assets/images/rcc.png)

```cpp
#define RCC ((RCC_def*) 0x40023800)
```

And we're done with the setup for the main.c file!

We’ll first define a simple delay function so we can actually see the LED blinking. 
A basic for loop will do the job:

```cpp
void delay(void) {
  for (volatile int i = 0; i < 1000000; i++);
}
```
Now for the int main() function. inside it, we’ll activate the RCC by setting bit 0 in RCC->AHB1ENR to enable the clock for GPIOA.

Then we’ll clear bits 11 and 10 in GPIOA->MODER, and set them properly to configure PA5 as an output pin.

Inside the main loop, we’ll turn the pin ON using BSRR, use the delay function, turn the pin OFF using BSRR, and then delay again before it turns back ON.

This time, we have to reference the reference manual and check the register map for GPIO to see how many bits we need to shift 1 by.
![MODER bits for PA5](/assets/images/moder.png)  
![BSRR layout](/assets/images/bsrr.png)
```cpp
int main() {
  RCC->AHB1ENR |= (1 << 0);


  GPIOA->MODER &= ~(3 << 10);  
  GPIOA->MODER |=  (1 << 10);  

  while (1) {
    GPIOA->BSRR = (1 << 5);  
    delay();
    GPIOA->BSRR = (1 << 21); 
    delay();
  }
}
```
That’s main.c done.

Moving on to the startup file —possibly the hardest part of this entire baremetal project. I decided to write it in assembly so I could really understand what’s going on under the hood.

But before jumping into the code, we need to clear up a few things first:
Why do we need a startup file?
What does it even do?

## Vector table, interrupts, and what happens before main():

When the MCU powers on, it needs to be told what to do — what data to move, what registers to activate, what peripherals to reset. all the logic lives in main(), but the MCU doesn’t just start there on its own. that’s where the startup code comes in. it handles the low-level setup: copying .data, zeroing .bss, setting up the stack, and finally jumping to main(). it also sets up a vector table to deal with interrupts and exceptions — because if a hardfault or some unexpected interrupt hits while the MCU is trying to blink an LED, it needs to know where to go. I’ll explain only the minimum needed to get this blinky working.

### Interrupts, what is it?

Imagine you're working as a receptionist at a hotel. you're focused on paperwork, and every 5 minutes you look up to check if any guests need help. all goes well until, in the middle of your work, a customer walks in — but leaves before your next 5-minute check. what happened? while you were processing your task, an event happened and you missed it.

The solution is to install a bell at the counter. now, whenever a customer comes in, they ring the bell and you immediately stop what you're doing to take care of them. once the interruption is handled, you go back to your paperwork.

This is more or less the same with an MCU — when the guest comes and rings the bell, we call that an **IRQ** (interrupt request). when an IRQ is triggered, the MCU runs its **ISR** (interrupt service routine), which is a special routine that handles that specific interrupt type.

Of course, not all interrupts are equal, a fire alarm and a ring bell from a guest aren't the same, hence why there is a priority list for interrupts. We divide these into **maskable** and **unmaskable** interrupts, **NMI**(non maskable interrupts) are like fire alarms, they **can't be ignored** and the MCU can't disable them. examples include: Reset, Hardfault, Busfault, while maskable interrupts can be ignored. 

This explanation leads us to the **vector table**, an essential componenet of the startup, the vector table is just a list of interrupts and exceptions (error events) each one paired with their an address so that the MCU can know where to jump in case an IRQ is triggered.

Our processor expects the vector table to be located at the beginning of memory, right after reset.
here is what it looks like:

![vector table](/assets/images/vec.png)


In the code, we want to define the isr_vector and include all the interrupts we need. we use .section to organize them into their own dedicated block.
.word is equivalent to uint32_t in a high level language, we're putting everything from the isr vector section int he beginning of the flash this way 

We also want to use %progbits to tell the compiler that isr_vector contzians actual data and isn't uninitialized space like .bss
```s
.syntax unified
.cpu cortex-m4
.thumb


.section .isr_vector, "a", %progbits
    .word   _estack                         /* Initial SP                   */
    .word   reset_handler        + 1        /* Reset                        */
    .word   Default_Handler      + 1        /* NMI                          */
    .word   Default_Handler      + 1        /* HardFault                    */
    .word   Default_Handler      + 1        /* MemManage                    */
    .word   Default_Handler      + 1        /* BusFault                     */
    .word   Default_Handler      + 1        /* UsageFault                   */
    .word   0                                  /* Reserved                  */
    .word   0
    .word   0
    .word   0
    .word   Default_Handler      + 1        /* SVCall                       */
    .word   Default_Handler      + 1        /* DebugMon                     */
    .word   0
    .word   Default_Handler      + 1        /* PendSV                       */
    .word   Default_Handler      + 1        /* SysTick                      */

    /* IRQ0-IRQ90 */
    .rept 91
        .word Default_Handler + 1
    .endr
```
### Reset_handler — the first C-style code that actually runs
Reset_handler is the very first routine the MCU executes after it loads the initial stack pointer. Its only jobs are:

Copy .data from Flash to SRAM.

Zero .bss.

Jump to main().

### .text, .data, and .bss:

We know we need to transfer the instructions and variables during runtime from flash to SRAM — but how do we actually do that?
pointers will play a huge role in making this happen.

In simple terms:

.text is where the code lives, these are your compiled instructions.

.data is where initialized variables live as in variables with a set value at startup.

.bss is where uninitialized variables live which are things like int counter; that have no assigned value.

![vector table](/assets/images/data.png)


I drew a diagram explaining what's happening during the trasnfer:

When the MCU starts, it needs to copy all the initialized variables from Flash to SRAM.
those variables are stored in Flash right after the .text section — starting at _etext.
we want to copy them into RAM, starting from _sdata and ending at _edata.

![transfer](/assets/images/transfer.png)

To transfer it, we’ll use a loop and two pointers — a destination and an end pointer.
as long as the destination is less than the end, we’ll copy from the source to the destination using ```*destination = *source;``` in pseudo code, then we increment both the destination and the source, moving through memory one word at a time.*

In assemply, we will define 3 registers r0, r1, and r2 and use ldr to load them with the addresses of _sidata, _sdata and _edata
we will then use cmp to compare r1 and r2 and as long as r1 is less than r2, we’ll keep copying data.

The next step is to zero the .bss section. to do that, we’ll use a similar loop, but this time we’ll store zeros instead of copying data.

To finish it off we'll jump to main 
In assembly it looks like this:

```s
.section .text.reset_handler, "ax", %progbits
.thumb_func
.global reset_handler
reset_handler:
    /* we want to copy .data (Flash to SRAM) */
    ldr r0, =_sidata         /* source  */
    ldr r1, =_sdata          /* destination */
    ldr r2, =_edata          /* end     */
1:
    cmp r1, r2
    ittt lt
    ldrlt r3, [r0], #4
    strlt r3, [r1], #4
    blt 1b

    /* zero .bss */
    ldr r0, =_sbss
    ldr r1, =_ebss
    movs r2, #0
2:
    cmp r0, r1
    itt lt
    strlt r2, [r0], #4
    addlt r0, r0, #4
    blt 2b

    /* jump to main */
    bl  main

    /* if main() ever returns, stay here */
3:  b 3b
.size reset_handler, .-reset_handler

```
We also need a default handler that takes care of any interrupts we haven't implemented which will prevent any sudden crashes in the MCU.

```s
.section .text.Default_Handler, "ax", %progbits
.thumb_func
.global  Default_Handler
Default_Handler:
    b .

.size Default_Handler, .-Default_Handler
```

That's the startup code done.


Moving on to the linker script.

## The memory layout:

The linker script is critical. it tells the toolchain exactly where every section lives in Flash and SRAM so the startup code can find it. ```.text, .data, .bss```, the vector table— all of them are defined here. if the addresses are wrong, the MCU will boots into chaos.for syntax your friend is :[text](https://sourceware.org/binutils/docs/ld/Scripts.html)

There are three important parts of the linker script: the memory section where we define the sizes and addresses of the FLASH and RAM, defining the stack pointer, and the SECTIONS block for he .isr_vector, .text , .data, etc



Make reset_handler the default program-entry symbol.
```ld
ENTRY(reset_handler)
```


Starting with the memory section: through the datasheet, the FLASH has a length of 512K while the SRAM has a length of 128K, The flags (r/x/w) say whether the region is readable, writable, executable.



```ld
MEMORY
{
  FLASH (rx)  : ORIGIN = 0x08000000, LENGTH = 512K
  SRAM  (rwx) : ORIGIN = 0x20000000, LENGTH = 128K
}
```

Defines _estack at the very top of SRAM.
On reset the CPU loads this into MSP, so every PUSH/function call is safe.

```ld
PROVIDE(_estack = ORIGIN(SRAM) + LENGTH(SRAM));
```



SECTIONS block:
Everything below decides where each piece of the binary lands:
the isr vector goes in the vector table at the very start of the FLASH
the text goes in the reset_handler in the FLASH rigth after the vector table 
the data goes in initialized globals and runs in SRAM but is image-stored in FLASH
bss goes in un-initialized globals and is zero filled in SRAM

```ld
SECTIONS
{
  .isr_vector :
  {
    KEEP(*(.isr_vector))
  } >FLASH

.text :
{
    . = ALIGN(4);
    *(.text.reset_handler)   /* <- comes first */
    *(.text*)
    *(.rodata*)
    . = ALIGN(4);
    _etext = .;
} >FLASH

  .data :
  {
    _sidata = LOADADDR(.data);   /* <-- new: flash address for .data image */
    . = ALIGN(4);
    _sdata = .;

    *(.data)

    . = ALIGN(4);
    _edata = .;
  } >SRAM AT> FLASH

  .bss :
  {
    . = ALIGN(4);
    _sbss = .;
		
    *(.bss)
		
    . = ALIGN(4);
    _ebss = .;
  } >SRAM
}
```


And that's all!
You can copy my makefile here for automatic compilation:
```Makefile
# Toolchain and flags
CC = arm-none-eabi-gcc
CFLAGS = -mcpu=cortex-m4 -mthumb -Wall -ffreestanding -Iinclude
LDFLAGS = -nostdlib -T linker.ld -Wl,--gc-sections

# Files
TARGET = blinky.elf
SRC = main.c startup.S
OBJ = $(SRC:.c=.o)
OBJ := $(OBJ:.S=.o)

# Default build
all: $(TARGET).bin

# Link ELF
$(TARGET): $(OBJ)
	$(CC) $(CFLAGS) $(LDFLAGS) $^ -o $@

# Convert ELF to BIN
$(TARGET).bin: $(TARGET)
	arm-none-eabi-objcopy -O binary $< blinky.bin


# Clean build files 
clean:
	del /f /q *.o *.elf *.bin 2>nul || exit 0

```

Last step: flash the binary to the STM32 with ST-Link Utility.

Connect the board.

Load blinky.bin in ST-Link Utility.

Click Download/Verify.

Hit Run — enjoy the blink.






