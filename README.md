<img src="./images/IFX_LOGO_600.gif" align="right" width="150"/>

# INTERRUPT NMI Timing Protection
**This code example demonstrates the Timing Protection configuration using Non Maskable Interrupt (NMI), shows its operation, initial setting, and interrupt handling.**  

## Device
The device used in this code example (CE) is:
- [TRAVEO™ T2G CYT4BF Series](https://www.infineon.com/cms/en/product/microcontroller/32-bit-traveo-t2g-arm-cortex-microcontroller/32-bit-traveo-t2g-arm-cortex-for-body/traveo-t2g-cyt4bf-series/)

## Board
The board used for testing is:
- TRAVEO™ T2G evaluation kit ([KIT_T2G-B-H_EVK](https://www.infineon.com/cms/en/product/evaluation-boards/kit_t2g-b-h_evk/), [KIT_T2G-B-H_LITE](https://www.infineon.com/cms/en/product/evaluation-boards/kit_t2g-b-h_lite/))

## Scope of work
In this example, the Interrupt Service Routine (ISR) processing time is protected. If the processing time limit is exceeded, an NMI is generated and appropriate action can be taken.

## Introduction  

**Interrupts**  
TRAVEO™ T2G platform supports the following interrupt features:
- Supports up to 1023 system interrupts
    - Eight Cortex-M7 external interrupts and eight Cortex-M7 internal (software only) interrupts. The CPU supports up to 
240 interrupts, but only sixteen interrupts are used by the TRAVEO™ T2G interrupt infrastructure. The eight external 
CPU interrupts support DeepSleep (WIC) functionality.
    - Eight Cortex-M0+ external interrupts and eight Cortex-M0+ internal (software only) interrupts. The CPU supports up to 
32 interrupts, but only sixteen interrupts are used by the TRAVEO™ T2G interrupt infrastructure. The eight external 
CPU interrupts support DeepSleep (WIC) functionality.
    - All the available system interrupt sources are usable in Active power mode and can wake up from Sleep power mode
    - A subset of available system interrupt sources capable of waking the device from DeepSleep power mode
    - Four system interrupts can be mapped to each of the CPU NMI
- Nested vectored interrupt controller (NVIC) integrated with each CPU core, yielding low interrupt latency
- Wakeup interrupt controller (WIC) enabling interrupt detection (CPU wakeup) in DeepSleep power mode
- Vector table may be placed in either flash or SRAM
- Configurable priority levels (eight levels for Cortex-M7 and four levels for Cortex-M0+) for each interrupt
- Level-triggered interrupt signals

More details can be found in [Technical Reference Manual (TRM)](https://www.infineon.com/dgdl/?fileId=5546d4627600a6bc017600bfae720007), [Registers TRM](https://www.infineon.com/dgdl/?fileId=5546d4627600a6bc017600be2aef0004) and [Data Sheet](https://www.infineon.com/dgdl/?fileId=5546d46275b79adb0175dc8387f93228).

## Hardware setup

This CE has been developed for:
- TRAVEO™ T2G evaluation kit ([KIT_T2G-B-H_EVK](https://www.infineon.com/cms/en/product/evaluation-boards/kit_t2g-b-h_evk/))<BR>
<img src="./images/KIT_T2G-B-H_EVK.gif"/><BR>
No changes are required from the board's default settings.
- TRAVEO™ T2G evaluation kit lite ([KIT_T2G-B-H_LITE](https://www.infineon.com/cms/en/product/evaluation-boards/kit_t2g-b-h_lite/))<BR>
<img src="./images/KIT_T2G-B-H_LITE.gif"/><BR>  
No changes are required from the board's default settings.


## Implementation
This design consists of two TCPWM counters, two user LEDs, and a user button. The counter is set to run at 1Hz and generate interrupt subject to processing time protection. When the ISR is activated, it first starts another counter running with 2Hz. The second counter generates an NMI to detect that the ISR processing time has exceeded 2s. Then waits until the user button is pressed. If the button is not pressed within 2s, an NMI is generated.

**STDOUT setting**

Initialization of the GPIO for UART is done in the <a href="https://infineon.github.io/retarget-io/html/group__group__board__libs.html#ga21265301bf6e9239845227c2aead9293"><i>cy_retarget_io_init()</i></a> function.
- Initialize the pin specified by CYBSP_DEBUG_UART_TX as UART TX, the pin specified by CYBSP_DEBUG_UART_RX as UART RX (these pins are connected to KitProg3 COM port)
- The serial port parameters become to 8N1 and 115200 baud

**GPIO port pin initialization**

Initialization of the GPIO port pin is done once in the <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__gpio__functions__init.html#gad61553f65d4e6bd827eb6464a7913461"><i>Cy_GPIO_Pin_Init()</i></a> function.
- Initialize the pin specified by CYBSP_USER_LED1/CYBSP_USER_LED2 as output (initial level = H, LED turns off)
- Initialize the pin specified by CYBSP_USER_BTN as input

**TCPWM initialization**

API calls for the TCPWM initialization is done in *init_Timer()* function.
- To initialize TCPWM counters, <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__tcpwm__functions__counter.html#ga6068a06ddc8a07c67bb6df86e920944c"><i>Cy_TCPWM_Counter_Init()</i></a> is called with using structure <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/structcy__stc__tcpwm__counter__config__t.html"><i>cy_stc_tcpwm_counter_config_t</i></a> which are auto-coded by Device Configurator as argument

    - *Figure 1. 1s counter setting*<BR><img src="./images/counter1.png" width="800"/>
    - *Figure 2. 2s counter setting*<BR><img src="./images/counter2.png" width="800"/>

- Then both counters are enabled by <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__tcpwm__functions__counter.html#ga1707e2cc291fe486fbea346157c65bff"><i>Cy_TCPWM_Counter_Enable()</i></a>, and their Compare 0 interrupt is enabled by <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__tcpwm__functions__common.html#ga47ec0d6214c01f67774fb97b9c3f0878"><i>Cy_TCPWM_SetInterruptMask()</i></a>
- For enroll 2s counter to generate NMI exception on its expiration, <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__sysint__functions.html#ga3163ed7de473929acded134fc67ab997"><i>Cy_SysInt_SetNmiSource()</i></a> is called
- Next, register a handler for each counter by <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__sysint__functions.html#gab2ff6820a898e9af3f780000054eea5d"><i>Cy_SysInt_Init()</i></a>
- Last, *NVIC_EnableIRQ()* is called to enable IRQ

**Start 1s counter**

If main loop detects the 1s counter is not started, it calls *start_TCPWM_Counter()* function to start it.
- At first it clears the counter by calling <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__tcpwm__functions__counter.html#gab1b0e66e97b06eb7fb2ab14aabab7975"><i>Cy_TCPWM_Counter_SetCounter()</i></a>
- Then it triggers the counter by calling <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__tcpwm__functions__common.html#gaafe86ec440bec9a2c23392f289cc3a8b"><i>Cy_TCPWM_TriggerStart_Single()</i></a>

**ISR of 1s counter**

The ISR function for 1s counter is *handle_TCPWM_Counter_Interrupt()*.
- At first, checking if the intended interrupt has occurred by <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__tcpwm__functions__common.html#gabb00d4f53ff5db82aa6f5e5ae0e79f8a"><i>Cy_TCPWM_GetInterruptStatusMasked()</i></a> before start ISR process
- Then, After clearing the interrupt by <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__tcpwm__functions__common.html#ga0c7a2d667dc983c82327463529f5d283"><i>Cy_TCPWM_ClearInterrupt()</i></a>, start the 2s counter which expiration set as NMI by calling *start_TCPWM_Counter()* function to protect timing
- Next, the LED indicating the start of the ISR process is lit by calling <a href="https://infineon.github.io/psoc6hal/html/group__group__hal__gpio.html#gaf66e7c3ed7b3b0711635d7687ae92291"><i>cyhal_gpio_write()</i></a>, and waits until the user button is pressed by calling <a href="https://infineon.github.io/psoc6hal/html/group__group__hal__gpio.html#ga3d38f1dd86c2e74b8ad10f1f6d1945c2"><i>cyhal_gpio_read()</i></a>
- Once the user button is pressed, stop the 2s counter by calling <a href="https://infineon.github.io/mtb-pdl-cat1/pdl_api_reference_manual/html/group__group__tcpwm__functions__common.html#gad1459b0ddabf388afe151de57f7cf4d4"><i>Cy_TCPWM_TriggerStopOrKill_Single()</i></a> and turn off the LED by calling <a href="https://infineon.github.io/psoc6hal/html/group__group__hal__gpio.html#gaf66e7c3ed7b3b0711635d7687ae92291"><i>cyhal_gpio_write()</i></a>

**Detection of timing overruns**

If the processing time of the ISR is not finished within 2sec, *handle_NMI_Exception()* function will be called as NMI exception.
- The confirmation and clearing of interrupt is same as the ISR of 1sec counter
- In this example, the action of timing overrun detection is only to lit the LED by <a href="https://infineon.github.io/psoc6hal/html/group__group__hal__gpio.html#gaf66e7c3ed7b3b0711635d7687ae92291"><i>gpio_hal_write()</i></a>, but it can be implemented other action to ensure the safety of the system

## Run and Test
For this example, a terminal emulator is required for displaying outputs. Install a terminal emulator if you do not have one. Instructions in this document use [Tera Term](https://ttssh2.osdn.jp/index.html.en).

After code compilation, perform the following steps to flashing the device:
1. Connect the board to your PC using the provided USB cable through the KitProg3 USB connector.
2. Open a terminal program and select the KitProg3 COM port. Set the serial port parameters to 8N1 and 115200 baud.
3. Program the board using one of the following:
    - Select the code example project in the Project Explorer.
    - In the **Quick Panel**, scroll down, and click **[Project Name] Program (KitProg3_MiniProg4)**.
4. After programming, the code example starts automatically. Confirm that the messages are displayed on the UART terminal.

    - *Figure 3. Terminal output on program startup*<BR><img src="./images/terminal.png" width="640" />

5. You can debug the example to step through the code. In the IDE, use the **[Project Name] Debug (KitProg3_MiniProg4)** configuration in the **Quick Panel**. For details, see the "Program and debug" section in the [Eclipse IDE for ModusToolbox™ software user guide](https://www.infineon.com/dgdl/?fileId=8ac78c8c8386267f0183a8d7043b58ee).

**Note:** **(Only while debugging)** On the CM7 CPU, some code in *main()* may execute before the debugger halts at the beginning of *main()*. This means that some code executes twice: once before the debugger stops execution, and again after the debugger resets the program counter to the beginning of *main()*. See [KBA231071](https://community.infineon.com/t5/Knowledge-Base-Articles/PSoC-6-MCU-Code-in-main-executes-before-the-debugger-halts-at-the-first-line-of/ta-p/253856) to learn about this and for the workaround.


## References  

Relevant Application notes are:
- AN235305 - GETTING STARTED WITH TRAVEO™ T2G FAMILY MCUS IN MODUSTOOLBOX™
- [AN224434](https://www.infineon.com/dgdl/?fileId=8ac78c8c7cdc391c017d0d3a71ec674a) - Clock configuration setup in TRAVEO™ T2G family CYT4B series
- [AN219842](https://www.infineon.com/dgdl/?fileId=8ac78c8c7cdc391c017d0d3a490a6732) - How to use interrupt in TRAVEO™ II
- [AN220224](https://www.infineon.com/dgdl/?fileId=8ac78c8c7cdc391c017d0d3a800a6752) - How to Use Timer, Counter, and PWM (TCPWM) in Traveo II Family

ModusToolbox™ is available online:
- <https://www.infineon.com/modustoolbox>

Associated TRAVEO™ T2G MCUs can be found on:
- <https://www.infineon.com/cms/en/product/microcontroller/32-bit-traveo-t2g-arm-cortex-microcontroller/>

More code examples can be found on the GIT repository:
- [TRAVEO™ T2G Code examples](https://github.com/orgs/Infineon/repositories?q=mtb-t2g-&type=all&language=&sort=)

For additional trainings, visit our webpage:  
- [TRAVEO™ T2G trainings](https://www.infineon.com/cms/en/product/microcontroller/32-bit-traveo-t2g-arm-cortex-microcontroller/32-bit-traveo-t2g-arm-cortex-for-body/traveo-t2g-cyt4bf-series/#!trainings)

For questions and support, use the TRAVEO™ T2G Forum:  
- <https://community.infineon.com/t5/TRAVEO-T2G/bd-p/TraveoII>  
