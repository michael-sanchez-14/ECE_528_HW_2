# ECE 528/L Homework 2

## Question 1
A hardware timer is a circuit within the microcontroller that is responsible for counting in order to measure time. Embedded systems commonly use these hardware timers for pulse widthm modulation(PWM).

## Question 2
A clock signal can be used for many different things in embedded systems, such as different communication protocols. 
The primary clock sources available on the MSP432P401R are :
* Auxiliary Clock(ACLK)
    * This clock is selected through software by individual peripherals and it is restricted to a maximum frequency of 128kHz
* Master Clock(MCLK)
    * Selected through software, this clock is used by both the CPU and peripheral modules
* Subsystem Master Clock(HSMCLK)
    * HSMCLK is software selectable by individual peripheral modules
* Low-Speed Subsystem Master Clock(SMCLK)
    * Uses the HSMCLK clock resource selection and it is limited in frequency to half the rated maximum frequency of HSMCLK
* Low-Speed Backup Domain Clock(BCLK)
    * this is primarily used in the backup domain, and is restricted to a maximum frequency of 
    32.768kHz

## Question 3
The maximum count value before roll over for a 16 bit timer is -> 2^16 - 1 = 65,535

## Question 4
The overflow period for a 24 bit timer that counts upward from zero at 10 MHz
is -> (2^24)/10 MHz = 1.677 seconds

## Question 5
The SysTick timer on the MSP432P401R is a 24 bit count down timer. It can be used as either a Real time operating system timer or a simple counter. We can use this timer as either a delay or even use it to do something periodically, for example turning on and off an LED every few seconds. For the SysTick timer within the STCSR Register we can set bit 2 to use the Core clock, while clearing this bit means the clock source is not applicable.

## Question 6
The SysTick timer contains 4 register on the MSP432P401R:
* SysTick Control and Status Register(STCSR)
    * This register is used to enable different SysTick features.
* SysTick Reload value Register(STRVR)
    * This register allows us to store a specific value to load into the current value register once it reaches 0.
* SysTick Current Value Register(STCVR)
    * This register allows us to find out what the current value in the register is.
* SysTick Callibration Value Register(STCR)
    * This register is used to allow software to scale to any required speed, to do this it either multiplies or divides.

## Question 7
The Timer_A peripheral on the MSP432P401R is a 16 bit timer as well as counter with several different modes. The main operating modes being:
* Stop
    * This mode stops the timer
* Up
    * This mode counts from 0 to the value in TAxCCR0, this happens repeatedly
* Continuous
    * This mode counts from 0 to 0FFFFh, this happens repeatedly
* Up/down
    * This mode counts from 0 to the value in TAxCCR0, then back down to 0

## Question 8
* TAxCTL
    * The Timer_Ax control register allows us to enable certain features, for example bits 4-5 allow us to chose the mode we are in while bits 8-9 are used for the clock source
* TAxCCTLn
    * The Timer_Ax Capture/Compare Control register allows us to choose how we want to capture/compare the data, these registers range from CCTL0 to CCTL6
* TAxR
    * The Timer_Ax counter register stores the count of Timer_A
* TAxCCRn
    * The Timer_Ax Capture/Compare register allows us to either hold the data and compare it to the TAxR value or copy the TAxR value into this register
* TAxIV
    * The Timer_Ax Interrupt Vector register tells us the source of where an interrupt is coming from
* TAxEX0
    * The Timer_Ax Expansion 0 register allows us to select the divider value for the input clock, these values range from 1 up to 8

## Question 9
Pulse width modulation is a technique that uses pulse widths to change a waveform in order to control a system or even transmit information.
Some examples of applications that use PWM are controlling different kinds of motors, controlling the brightness of an LED or even audio generation.

## Question 10
There are 4 modules available within Timer_A there are also up to 5 PWM generators.

## Question 11
To initialize P2.4 as a PWM output driven by Timer_A0 CCR1 we first configured P2.4 to use the primary module function, to do this we set bit 4 in the SEL0 register and clear bit 4 in the SEL1 register. Next we set bit 4 in the DIR register so the pin is an output. After doing this we set the value in the CCR[0] = 60000, by doing so it results in 20ms, next for a duty cycle of 50% we take half of the CCR[0] value and assign that to CCR[1]. Next we clear the EX0 register and configure the output mode as toggle/reset in the CCTL[1] register. Finally through the CTL regsiter we select the SMCLK, divide the timer clock by 2 and selecting the up/down mode.
```
    P2->SEL0 |= 0x10;
    P2->SEL1 &= ~0x10;
    P2->DIR |= 0x10;
    TIMER_A0->CCR[0] = 60000;
    TIMER_A0->CCR[1] = 30000;
    TIMER_A0->EX0 = 0x0000;
    TIMER_A0->CCTL[1] |= 0x40;
    TIMER_A0->CTL |= 0x270;
```
## Question 12
To configure Timer_A0 to generate a periodic interrupt every 2ms, to first start this I clear bits 4 and 5 to clear the timer, next set bit 9 in the CTl register for SMLCK and to divide by 1. Once this is done set the fourth bit in the CCTL[0] register, this is done to enable the interrupt request for capture/compare interrupt flags. Next making the EX0 register 0 will divide the SMCLK frequency by 1 and after this we set the interrupt priority level and enable the interruppt in NVIC, lastly we set bits in the CTL register to make the clock in up mode.
```
TIMER_A0->CTL &= ~0x0030;
TIMER_A0->CTL |= 0x0200;
TIMER_A0->CCTL[0] |= 0x0010;
TIMER_A0->CCR[0] = (32000 - 1);
TIMER_A0->EX0 = 0x0000;
NVIC->IP[2] = (NVIC->IP[2] & 0xFFFFF00F) | 0x00000060;
NVIC->ISER[0] |= 0x00000100;
TIMER_A0->CTL |= 0x0014;
```

## References

* https://www.sciencedirect.com/topics/engineering/hardware-timer
* https://www.embeddedrelated.com/glossary/pwm
