Banggood sells under DANIU brand, ELIKLIV and Yeapook on Amazon, [FNIRSI 1013D](https://www.aliexpress.com/af/1013d.html) and [FNIRSI 1014D](https://www.aliexpress.com/af/1014d.html) at AliExpress.

NOTE: The [FNIRSI 1013D](http://fnirsi.cn/productinfo/556152.html) and [1014D](http://fnirsi.cn/productinfo/575891.html) have one dual (2X) 100 MSPS Analog to Digital Converter for each probe. In practical terms this means it has 400 MSPS max total, 200 MSPS per channel (contrary to the 1GSPS spec) which gives it the sampling performance of 40 MHz scope. Bandwidth and software may limit that performance even further. The oscilloscope bandwidth in MHz is technically defined as [-3dB cutoff point with the 10X probe setting](https://www.tek.com/en/support/faqs/how-bandwidth-defined-oscilloscope-0) of the measured signal (Equivalent to 70.7% of the true value). The front-end has a Nyquist filter with corner frequency around 30MHz-35Mhz, and the OpAmp may further limit full range signal swing times. Due to the stock firmware switching to waveform estimation over ~44MHz, the appropriate spec is not clear, but it can certainly be said it doesn't perform well up to 100MHz.

![ScopeBandwidthSelection_092016-1](https://user-images.githubusercontent.com/13755049/210172722-59bac364-45af-469d-aeff-836f72458478.jpg "SIGLENT oscilloscope bandwidth determination")

It will obviously never be okay for *production* use in telecom, automotive, medical etc. work governed by quality, information security or other management systems and regulations, but you should already know that. This is strictly for R&D, primarily in the paying from your own pocket hobbyist scene. (The FNIRSI use-cases list actually says as much without explicitly spelling it out).

![3702266](https://user-images.githubusercontent.com/13755049/210172651-015619f0-44b2-4eb2-a4f5-f898484ce5ba.png "FNIRSI 1013D/1014D oscilloscope use-cases")

These limits appear to be true for [all sub-$200EUR](https://www.eevblog.com/forum/testgear/best-portable-scope-under-$200-for-pinball-work/) scopes as of 2022. Such a scope is still be useful for hobbyist projects, and indeed most embedded circuit work. In the case of the [FNIRSI 1013D there exists working open source firmware](https://github.com/pecostm32/FNIRSI_1013D_Firmware) which allows testing new concepts or implementing protocol decoders not found even on more expensive scopes. 1014D adds basic signal programmable signal generator, professional benchtop look and familiar user-interface for learning, though the open source firmware does not yet work for it.

True 1GSPS would require [ADC08D500 or compatible](https://www.ti.com/product/ADC08D500/part-details/ADC08D500CIYB/NOPB) starting at around 150 dollars piece, or the MXT2002 clone; there's no chance of replacing this piece, you need a new scope like [Hantek DSO2X1X](https://www.eevblog.com/forum/testgear/hacking-the-dso2x1x/) which may be found for prices over 200 EUR in EU (Meaning you will likely have to handle taxes & customs as well). There's no open firmware for it as of yet.

Related works:
* Alpha stage project [FNIRSI Open5012h project](https://github.com/ataradov/open-5012h) is similar but with no FPGA, single channel, and most notably programming requires swapping the MCU. Number of forks suggests there's great interest even for this.
* Class project for [FPGA based oscilloscope](https://github.com/agural/FPGA-Oscilloscope) with claimed 1mV sensitivity.
* [FPGA digital oscilloscope](https://www.fpga4fun.com/digitalscope.html) project, for learning.
* [Open source ScopeFUN project](https://www.scopefun.com/) using KAD5510P-25 10bit 250MSPS ADC.
* [Earth People Technology DSO 100M](https://earthpeopletechnology.com/digital_storage_oscilloscope_dso_100m) complete open source 4ch 80MSPS USB-scope with different OpAmp front-end.
* [Hantek DSO2X1X](https://www.eevblog.com/forum/testgear/hacking-the-dso2x1x/) appear to have same hardware in every version, which could be unlocked. [Booting Linux](https://github.com/AndrewBCN/Hantek_DSO2x1x_Linux) - I can't tell if this can access the DSO hardware yet, and it doesn't quite make the sub-200 category.

Possibly implementing [sigrok](https://sigrok.org/wiki/Main_Page) compatible data-logger/computer interface?

[Equivalent-time sampling](https://www.tek.com/en/documents/application-note/real-time-versus-equivalent-time-sampling) and other waveform reconstruction techniques.

### 29-DEC-2022 Forked repo to add some of my own notes, for now.

Many of the claims are currently sourced to [EEVblog discussion](https://www.eevblog.com/forum/testgear/new-bench-scope-fnirsi-1014d-7-1gsas/msg4604008/#msg4604008).

"Firmware" in this is generally meant to imply both software of the [Allwinner F1C100s](https://www.cnx-software.com/2022/01/27/allwinner-f1c100s-handheld-computer-should-cost-15-to-manufacture/) with ARM9 core, whether from the built-in SD-card or SPI chip, the 1014D instrument panel co-processor, or the FPGA SPI-flash or other programmable components inside the scope. In other words, everything that goes inside the covers of the scope, but doesn't require swapping components to change. Notably, the existing [1013D open firmware](https://github.com/pecostm32/FNIRSI_1013D_Firmware) can be ran without changing any of the firmware, by specially preparing the SD-card inside the scope via its USB connection.

#### Bandwidth and samples per second

On the basic specs, it was confirmed the FNIRSI 1014D operates at 200MSa/s for fastest time division setting. This is the nominal performance of [AD9288-100](https://www.analog.com/media/en/technical-documentation/data-sheets/AD9288.pdf), which according to the data-sheet has 475 MHz bandwidth and could be operated at least up to 110MSa/s for each of the 2 independent channels at reduced quality.

This is likely actually [MXT2088 Chinese clone](http://web.archive.org/web/20211001051828/http://www.mxtronics.com/n107/n124/n181/n184/c692/attr/2630.pdf) with identical specs. For example according to [teardown of OWON HSD272S](http://www.kerrywong.com/2021/09/18/teardown-of-an-owon-hds272s-3-in-1-handheld-oscilloscope-dmm-awg-compared-with-hantek-2d72/) that scope is running it at 125MHz per channel. Actual performance of silicon remains to be seen. Honestly to my eye the [front-end simulation](https://www.eevblog.com/forum/testgear/fnirsi-1013d-100mhz-tablet-oscilloscope/msg3147914/#msg3147914) looks kind... normal. Remmeber you're *expected* to get -3dB/30% drop to the specified bandwidth! It's not quite clear to me what this simulation is showing, though. Where are the probe points located, what are the exact values?

EEVblog discussion also notes stock firmware switches to sine wave approximation around ~44MHz, with a filter of around ~30MHz-35MHz on the signal path. This is to be expected, since according to [Shannon-Nyquist Sampling Theorem](https://www.allaboutcircuits.com/technical-articles/nyquist-shannon-theorem-understanding-sampled-systems/) the maximum resolvable frequency is half of the sample rate, and higher frequencies getting into the ADC alias to look like different frequency. This is kind of a shame as it would still be possible to use the scope to align the phase (highest amplitude) of higher frequency signals of a bus to clock, and each other. A [poster remarks](https://www.eevblog.com/forum/testgear/fnirsi-1013d-100mhz-tablet-oscilloscope/msg3148108/#msg3148108) low-pass filter corner should be no more than 40-50% of Nyquist right after the simulation.

#### Vertical sensitivity, analog frontend

"Another drawback is the vertical sensitivity, which with only 100mV per division true range with a 1x probe setting it is not very good. The advertised 50mV per division is based on software zoom." 1014D analog frontend OpAmp is listed as [Runic Technology RS8751XF](https://www.run-ic.com/upload/web/uploads/file/20210421/wD788577TDFQf917d01M2Jj2zb6SkR8M.pdf). [User marauder on the eevblog forums](https://www.eevblog.com/forum/testgear/fnirsi-1013d-100mhz-tablet-oscilloscope/msg4354150/#msg4354150) reports successfully replacing the OpAmp with [Texas Instruments OPA356](https://www.ti.com/lit/gpn/opa356). This chip has significantly better specs on paper, and might allow increasing the gain rate, although the primary purpose of the OpAmp appears to be acting as a buffer for the ADC.

[This answer](https://itecnotes.com/electrical/electronic-attenuation-oscilloscope-front-end/](https://electronics.stackexchange.com/questions/58115/how-to-achieve-high-impedance-input-on-opamp-without-sacrificing-bandwidth)) goes some way towards explaining high-impendance scope buffer with OpAmp in "non-inverting unity gain buffer configuration". [Texas Instruments Buffer Op Amp to ADC Circuit Collection](https://www.ti.com/lit/pdf/sloa098) doesn't have this exact configuration, in particular due to the AC/DC coupling selection.

##### Noise reduction, signal quality

There is [another discussion](https://www.eevblog.com/forum/testgear/fnirsi-1013d-100mhz-tablet-oscilloscope/msg3148590/#msg3148590) considering the OpAmp as OPA356 and explaining some of the choices/[simulation](https://www.eevblog.com/forum/testgear/fnirsi-1013d-100mhz-tablet-oscilloscope/msg3147914/#msg3147914): "If this is the case then the switching (using the values in the circuit) will follow fairly closely the sequence 5V/div, 2.5V/div, 1V/div, 500mV/div, 200mV/div and 100mV/div.  This leaves the 50mV/div scale and it is quite possible that it is done in software, reducing the resolution to 7 bits, unless there is another gain stage after the OPA356." Conversation needs some parsing to determine if changing components would make sense.

![image](https://user-images.githubusercontent.com/13755049/211763558-76653cc1-bcee-4433-81cb-66f403bd9890.png)

The OpAmps, relays and the rest of the analog front-end reside under the two large tin-cans on the PCB. Another [hack is providing the OpAmps with a dual power supply](https://www.eevblog.com/forum/testgear/fnirsi-1013d-100mhz-tablet-oscilloscope/msg4536116/#msg4536116) to fix ground level when the front USB cable is plugged into computer, though this is perhaps not for the faint of heart.

At least [Hantek 6022BL](https://sigrok.org/wiki/Hantek_6022BL) appears to have same ADC covered in glued-on heatsink, which would be one way to reduce noise and improve performance. Both ADC's and OpAmp's are fairly temperature sensitive. In my testing the ADC's run about 20 degress C above ambient, but the [ms5351m clock generator](https://www.qrp-labs.com/images/synth/ms5351m.pdf) runs whopping 30 degrees above. I can't find information on temperature stability of the chip, but everything seems to assume that it should be running at roughly room temperature. Adding heat sink without mechanical stresses for the clock generator would be hard. Temperature inside the box will always be bit higher than the room temperature.

#### FPGA work, sample depth

Its FPGA is said to have up to 24KB per channel (4 or 2 since both ADC have two channels?), with 2500 displayed of 3000 available with room for 4096? I think this is supposed to be 24KB for 2 channels, as it's supposed to have space to be quadrupled, but still unclear values. Another [FNIRSI 1013D teardown](https://www.cnx-software.com/2022/11/16/fnirsi-1013d-teardown-and-mini-review-a-portable-oscilloscope-based-on-allwinner-cpu-anlogic-fgpa/) shows EF2L45LG144B FPGA and gives its specifications as 35k DistributeRAM (bits, presumably) and 700k eRAM.

1. Anlogic IDE: https://dl.sipeed.com/shareURL/TANG/Premier/IDE 5.0.4
2. Mentioned: https://dl.sipeed.com/shareURL/TANG/Primer/IDE but that's 4.6.4
3. Reference: https://www.eevblog.com/forum/fpga/latest-version-of-anlogic-ide/
4. Translations: https://github.com/kprasadvnsi/Anlogic_Doc_English

Datasheet for [Altera Cyclone IV EP4CE6 (ep4ce6e22c8n)](https://www.altera-price.com/files/7f/EP4CE40F23C6N.pdf) [used in earlier versions of FNIRSI 1013D](https://www.eevblog.com/forum/testgear/fnirsi-1013d-100mhz-tablet-oscilloscope/msg4027378/#msg4027378) says that has 270 Kbits of memory, which would be 33,75KB. However, [EEVblog post](https://www.eevblog.com/forum/fpga/reverse-engineering-anlogic-al3_10-fpga/) says this is Anlogic [AL3A10LG144C7](https://www.eevblog.com/forum/testgear/fnirsi-1013d-100mhz-tablet-oscilloscope/msg3885095/#msg3885095) FPGA. Memory could be used for other purposes as well. Different models and builds are using different FPGA's, which may complicate things.

Based on [reverse-engineered schematics](https://github.com/pecostm32/FNIRSI-1013D-1014D-Hack/blob/main/Schematics/1014D/Scope_1014D_Data_Acquisition.png) FPGA's W25Q80 SPI flash has SPI interface at J2. The SPI flash on my 1014D scope is actually [ZB25VQ80ATIG](https://datasheet.lcsc.com/lcsc/2003141212_Zbit-Semi-ZB25VQ80ATIG_C495747.pdf) which doesn't appear to be directly supported by most flashers. Unfortunately I haven't had success reading the flash with J-Link ([not supported](https://www.segger.com/supported-devices/winbond/spi)). [BusPirate](https://learn.sparkfun.com/tutorials/bus-pirate-v36a-hookup-guide/all) with 3v3 unconnected and board powered on worked with latest 'flashrom' using generic mode worked, however. The JTAG connector on the same page appears to be connected to the FPGA, though not shown on the schematic. This should allow debugging and updating the FPGA on the fly, if facilitated by software.

Interpreting the FPGA bitstream on my 1014D:

f0 (device id packet) 00 (flags) 0006 (6 bytes) 18006c31 (al3_10) 0bb7 (crc) - which happens to be the al3_10 identifier, so yup, same device, which is main thing I wanted to know.

First difference to 1013D sample looks like starting at 0x56 (starting at 0), c1 that is VERSION_UCODE, on mine that is 00e40000, 1013D in repository 001c0000

The rest of the changes are after the "ec f0 0433" identifier, which are the actual FPGA blocks already. 0x0433 = 1075 = number of frames in Anlogic AL3 family.

Another post on the thread notes that reverse-engineering the FPGA revealed it doesn't have [JTAG or ISP core](https://www.altera-price.com/files/51/DC-VIDEO-TVP5146N.pdf) as evident by no connection to the pins from user logic - [MCU doesn't have JTAG or SPI connection to the FPGA](https://github.com/pecostm32/FNIRSI-1013D-1014D-Hack/blob/main/Schematics/1014D/Scope_1014D_Processor.png), so it'd have to be ISP over the shared data link.

[HAL – The Hardware Analyzer](https://github.com/emsec/hal) was really pretty much first match I found for FPGA reverse-engineering, there might be better reverse-engineering tools. It does appear that [pcprogrammer on EEVblog has already finished the job of reversing the netlist for the FPGA into Verilog](https://www.eevblog.com/forum/fpga/reverse-engineering-anlogic-al3_10-fpga/50/#lastPost). There appears to be some remaining problems which might be due to unconstrained clock domain crossings etc.

#### 1014D development, firmware update

[Instructions note there are some custom fields which need to be saved](https://github.com/pecostm32/FNIRSI-1013D-1014D-Hack/tree/main/Test%20code/fnirsi_1014d_firmware_backup), so I wonder if I can use this on the 1014D? In retrospect I think the custom fields must be SPI-flash only. The scope can be mounted as SD-card to run the flash-extractor or whole ARM firmware from the SD card without even opening the scope.

[FEL with DRAM setup](https://github.com/pecostm32/FNIRSI-1013D-1014D-Hack/tree/main/Test%20code/fnirsi_1014d_startup_with_fel) would allow using sunxi-fel binary to load new binaries into memory from USB port without writing SD flash. The sunxi-fel provided firmware doesn't work as it doesn't know how to initialize FNIRSI DRAM.

[SD card emulation](https://github.com/froloffw7/FNIRSI-1013D-1014D-Hack) in a separate fork, according to [explanation](https://www.eevblog.com/forum/testgear/fnirsi-1013d-100mhz-tablet-oscilloscope/msg4326355/#msg4326355) this is SD-card emulations for QEMU emulator allowing testing and reverse-engineering the ARM firmware on a desktop.

1014D uses [ms5351m clock generator](https://www.qrp-labs.com/images/synth/ms5351m.pdf) for generating two FPGA clocks. Up to 200 MHz for the signal generator, and 50 MHz for the scope logic. [Skyworks Si5351 AN619](https://www.skyworksinc.com/-/media/Skyworks/SL/documents/public/application-notes/AN619.pdf) explains the clock generator I2C configuration, although their ClockBuilder Pro may be easiest solution to generate the correct register files.

Another unique important part for 1014D is the [physical knob controller](https://github.com/pecostm32/FNIRSI-1013D-1014D-Hack/blob/main/Schematics/1014D/Scope_1014D_User_Interface.png) which gladly shows all the wiring. Decompiling the initialization shows the UART is configured near 115200 baud rate. This is derived from the system clock, which in 1014D original firmware is set at 575 MHz instead of 600 MHz in the open firmware, thus the divisor latch needed to be adjusted to match.

### NOTES TO SELF

ESPRESSIF ESP32-WROVER-B uses CH343CDC driver. NOT the CH343SER driver, that'll bring down the entire Remote Desktop Infrastructure node, don't do that.
