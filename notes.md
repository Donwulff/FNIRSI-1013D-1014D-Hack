Banggood sells under DANIU brand, ELIKLIV on Amazon, [FNIRSI](http://fnirsi.cn/productinfo/575891.html) at AliExpress.

NOTE: The [FNIRSI 1013D](https://www.aliexpress.com/af/1013d.html) and [1014D](https://www.aliexpress.com/af/1014d.html) have one two-channel 100 MSPS Analog to Digital Converter for each probe. In practical terms this means it has 400 MSPS max. (contrary to the 1GSPS spec) which gives it the sampling performance of 40 MHz scope. Bandwidth and software may limit that performance even further. This appears to be true for [all sub-$200EUR](https://www.eevblog.com/forum/testgear/best-portable-scope-under-$200-for-pinball-work/) scopes as of 2022. Such a scope is still be useful for hobbyist projects, and indeed most embedded circuit work. In the case of [FNIRSI 1013D there also exists working open source firmware](https://github.com/pecostm32/FNIRSI_1013D_Firmware) which allows testing new concepts or implementing protocol decoders not found even on more expensive scopes. 1014D adds basic signal programmable signal generator, professional benchtop look and familiar user-interface for learning.

It will obviously never be okay for telecom, utomotive, medical etc. *production* work governed by quality, security or other management systems and regulations, but you should already know that. This is strictly for R&D, primarily in the paying from your own pocket hobbyist scene.


Related work:
* Alpha stage project [FNIRSI Open5012h project](https://github.com/ataradov/open-5012h) is similar but with no FPGA, single channel, and most notably programming requires swapping the MCU. Number of forks suggests there's great interest even for this.
* Class project for [FPGA based oscilloscope](https://github.com/agural/FPGA-Oscilloscope) with claimed 1mV sensitivity.
* [FPGA digital oscilloscope](https://www.fpga4fun.com/digitalscope.html) project, for learning.

Possibly implementing [sigrok](https://sigrok.org/wiki/Main_Page) compatible data-logger/computer interface?

### 29-DEC-2022 Forked repo to add some of my own notes, for now.

Many of the claims are currently sourced to [EEVblog discussion](https://www.eevblog.com/forum/testgear/new-bench-scope-fnirsi-1014d-7-1gsas/msg4604008/#msg4604008).

"Firmware" in this is generally meant to imply both software of the [Allwinner F1C100s](https://www.cnx-software.com/2022/01/27/allwinner-f1c100s-handheld-computer-should-cost-15-to-manufacture/) whether from the built-in SD-card or SPI chip, the 1014D instrument panel co-processor, or the FPGA SPI-flash or other programmable components inside the scope. In other words, everything that goes inside the covers of the scope, but doesn't require swapping components to change. Notably, the existing [1013D open firmware](https://github.com/pecostm32/FNIRSI_1013D_Firmware) can be ran without changing any of the firmware, by specially preparing the SD-card inside the scope via its USB connection.

#### Bandwidth and samples per second

On the basic specs, it was confirmed the FNIRSI 1014D operates at 200MSa/s for fastest time division setting. This is the nominal performance of [AD9288-100](https://www.analog.com/media/en/technical-documentation/data-sheets/AD9288.pdf), which according to the data-sheet has 475 MHz bandwidth and could be operated at least up to 110MSa/s for each of the 2 independent channels at reduced quality.
This is likely actually [MXT2088 Chinese clone](http://web.archive.org/web/20211001051828/http://www.mxtronics.com/n107/n124/n181/n184/c692/attr/2630.pdf) with identical specs. For example according to [teardown of OWON HSD272S](http://www.kerrywong.com/2021/09/18/teardown-of-an-owon-hds272s-3-in-1-handheld-oscilloscope-dmm-awg-compared-with-hantek-2d72/) that scope is running it at 125MHz per channel. Actual performance of silicon remains to be seen.

Discussion also notes stock firmware switches to sine wave approximation around ~44MHz, with a filter of around ~30MHz on the signal path. This is to be expected, since according to [Shannon-Nyquist Sampling Theorem](https://www.allaboutcircuits.com/technical-articles/nyquist-shannon-theorem-understanding-sampled-systems/) the maximum resolvable frequency is half of the sample rate, and higher frequencies getting into the ADC alias to look like different frequency. This is kind of a shame as it would still be possible to use the scope to align the phase (highest amplitude) of higher frequency signals of a bus to clock, and each other.

#### Vertical sensitivity, analog frontend

"Another drawback is the vertical sensitivity, which with only 100mV per division true range with a 1x probe setting it is not very good. The advertised 50mV per division is based on software zoom." [User marauder on the eevblog forums](https://www.eevblog.com/forum/testgear/fnirsi-1013d-100mhz-tablet-oscilloscope/msg4354150/#msg4354150) reports successfully replacing the OpAmp with [Texas Instruments OPA356](https://www.ti.com/lit/gpn/opa356). This chip has significantly better specs on paper, and might allow increasing the gain rate, although the primary purpose of the OpAmp appears to be acting as a buffer for the ADC.

There is [another discussion](https://www.eevblog.com/forum/testgear/fnirsi-1013d-100mhz-tablet-oscilloscope/msg3148590/#msg3148590) considering the OpAmp as OPA356 and explaining some of the choices/[simulation](https://www.eevblog.com/forum/testgear/fnirsi-1013d-100mhz-tablet-oscilloscope/msg3147914/#msg3147914): "If this is the case then the switching (using the values in the circuit) will follow fairly closely the sequence 5V/div, 2.5V/div, 1V/div, 500mV/div, 200mV/div and 100mV/div.  This leaves the 50mV/div scale and it is quite possible that it is done in software, reducing the resolution to 7 bits, unless there is another gain stage after the OPA356." Conversation needs some parsing to determine if changing components would make sense.

[Answer]([https://itecnotes.com/electrical/electronic-attenuation-oscilloscope-front-end/](https://electronics.stackexchange.com/questions/58115/how-to-achieve-high-impedance-input-on-opamp-without-sacrificing-bandwidth)) goes some way towards explaining high-impendance scope buffer with OpAmp in "non-inverting unity gain buffer configuration". [Texas Instruments Buffer Op Amp to ADC Circuit Collection](https://www.ti.com/lit/pdf/sloa098) doesn't have this exact configuration, in particular due to the AC/DC coupling selection.

Another [hack is providing the OpAmps with a dual power supply](https://www.eevblog.com/forum/testgear/fnirsi-1013d-100mhz-tablet-oscilloscope/msg4536116/#msg4536116) to fix ground level, though this is perhaps not for the faint of heart. The OpAmps, relays and the rest of the analog front-end reside under the two large tin-cans on the PCB.

#### FPGA work, sample depth

Its FPGA is said to have up to 24KB per channel (4 or 2 since both ADC have two channels?), with 2500 displayed of 3000 available with room for 4096? I think this is supposed to be 24KB for 2 channels, as it's supposed to have space to be quadrupled, but still unclear values. Another [FNIRSI 1013D teardown](https://www.cnx-software.com/2022/11/16/fnirsi-1013d-teardown-and-mini-review-a-portable-oscilloscope-based-on-allwinner-cpu-anlogic-fgpa/) shows EF2L45LG144B FPGA and gives its specifications as 35k DistributeRAM (bits, presumably) and 700k eRAM.

Anlogic IDE: https://dl.sipeed.com/shareURL/TANG/Premier/IDE 5.0.4
Mentioned: https://dl.sipeed.com/shareURL/TANG/Primer/IDE but that's 4.6.4
Reference: https://www.eevblog.com/forum/fpga/latest-version-of-anlogic-ide/
Translations: https://github.com/kprasadvnsi/Anlogic_Doc_English

Datasheet for [Altera Cyclone IV EP4CE6 (ep4ce6e22c8n)](https://www.altera-price.com/files/7f/EP4CE40F23C6N.pdf) [used in earlier versions of FNIRSI 1013D](https://www.eevblog.com/forum/testgear/fnirsi-1013d-100mhz-tablet-oscilloscope/msg4027378/#msg4027378) says that has 270 Kbits of memory, which would be 33,75KB. However, [EEVblog post](https://www.eevblog.com/forum/fpga/reverse-engineering-anlogic-al3_10-fpga/) says this is Anlogic [AL3A10LG144C7](https://www.eevblog.com/forum/testgear/fnirsi-1013d-100mhz-tablet-oscilloscope/msg3885095/#msg3885095) FPGA. Memory could be used for other purposes as well. Different models and builds are using different FPGA's, which may complicate things.

Based on [reverse-engineered schematics](https://github.com/pecostm32/FNIRSI-1013D-1014D-Hack/blob/main/Schematics/1014D/Scope_1014D_Data_Acquisition.png) FPGA's W25Q80 SPI flash has SPI interface at J2. Unfortunately I haven't had success reading the flash with J-Link ([not supported](https://www.segger.com/supported-devices/winbond/spi)) or [BusPirate](https://learn.sparkfun.com/tutorials/bus-pirate-v36a-hookup-guide/all) (chip-ID 0 when powered off the BusPirate) yet. The JTAG connector on the same page appears to be connected to the FPGA, though not shown on the schematic. This should allow debugging and updating the FPGA on the fly, if facilitated by software.

Another post on the thread notes that reverse-engineering the FPGA revealed it doesn't have [JTAG or ISP core](https://www.altera-price.com/files/51/DC-VIDEO-TVP5146N.pdf) as evident by no connection to the pins from user logic - [MCU doesn't have JTAG or SPI connection to the FPGA](https://github.com/pecostm32/FNIRSI-1013D-1014D-Hack/blob/main/Schematics/1014D/Scope_1014D_Processor.png), so it'd have to be ISP over the shared data link.

[HAL – The Hardware Analyzer](https://github.com/emsec/hal) was really pretty much first match I found for FPGA reverse-engineering, there might be better reverse-engineering tools. It does appear that [pcprogrammer on EEVblog has already finished the job of reversing the netlist for the FPGA into Verilog](https://www.eevblog.com/forum/fpga/reverse-engineering-anlogic-al3_10-fpga/50/#lastPost). There appears to be some remaining problems which might be due to unconstrained clock domain crossings etc.

#### 1014D development, firmware update

[Instructions note there are some custom fields which need to be saved](https://github.com/pecostm32/FNIRSI-1013D-1014D-Hack/tree/main/Test%20code/fnirsi_1014d_firmware_backup), so I wonder if I can use this on the 1014D? Or is everything on the SD/MMC card (which could break, so backup?). In retrospect I think the custom fields must be SPI-flash only. The scope can be mounted as SD-card to run the flash-extractor or whole ARM firmware from the SD card without even opening the scope.

[FEL with DRAM setup](https://github.com/pecostm32/FNIRSI-1013D-1014D-Hack/tree/main/Test%20code/fnirsi_1014d_startup_with_fel), at least could get to boot? See is this differs from 1013d at all.

[SD card emulation](https://github.com/froloffw7/FNIRSI-1013D-1014D-Hack) in a separate fork, according to [explanation](https://www.eevblog.com/forum/testgear/fnirsi-1013d-100mhz-tablet-oscilloscope/msg4326355/#msg4326355) this is SD-card emulations for QEMU emulator allowing testing and reverse-engineering the ARM firmware on a desktop.

The unique important part for 1014D is the [physical knob controller](https://github.com/pecostm32/FNIRSI-1013D-1014D-Hack/blob/main/Schematics/1014D/Scope_1014D_User_Interface.png) which gladly shows all the wiring - looks like it connects with RX/TX USART to the MCU, so first step could be reverse-engineering that protocol.
