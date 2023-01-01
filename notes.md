NOTE: The FNIRSI 1013D and 1014D have one two-channel 100 MSPS ADC for each probe. In practical terms this means it has 400 MSPS max. which gives it the sampling performance of 40MHz scope. Bandwidth and software may limit that performance even further. This appears to be true for [all sub-$200EUR](https://www.eevblog.com/forum/testgear/best-portable-scope-under-$200-for-pinball-work/) scopes as of 2023. Such a scope can still be useful for hobbyist projects, and indeed most embedded circuit work. In the case of [FNIRSI 1013D there also exists working open source firmware](https://github.com/pecostm32/FNIRSI_1013D_Firmware) which allows testing new concepts or implementing protocol decoders not found even on more expensive scopes.

### 29-DEC-2022 Forked repo to add some of my own notes, for now.

Many of the claims are currently sourced to [EEVblog discussion](https://www.eevblog.com/forum/testgear/new-bench-scope-fnirsi-1014d-7-1gsas/msg4604008/#msg4604008).

On the basic specs, it was confirmed the FNIRSI 1014D operates at 200MSa/s for fastest time division setting. This is the nominal performance of AD9288-100, which according to the data-sheet has 475 MHz bandwidth and could be operated at least up to 110MSa/s for each of the 2 independent channels at reduced quality.
This is likely actually MXT2088 Chinese clone with identical specs. According to [teardown OWON HSD272S](http://www.kerrywong.com/2021/09/18/teardown-of-an-owon-hds272s-3-in-1-handheld-oscilloscope-dmm-awg-compared-with-hantek-2d72/) is running it at 125MHz per channel. Actual performance of silicon remains to be seen.

Discussion also notes stock firmware switches to sine wave approximation around ~44MHz, with around ~30MHz filter on the signal path. This is to be expected, since according to [Shannon-Nyquist Sampling Theorem](https://www.allaboutcircuits.com/technical-articles/nyquist-shannon-theorem-understanding-sampled-systems/) the maximum resolvable frequency is half of the sample rate, and higher frequencies getting into the ADC alias to look like different frequency. This is kind of a shame as it would still be possible to use the scope to align the phase (highest amplitude) of higher frequency signals of a bus to clock and each other.

FPGA is said to have up to 24KB per channel (4 or 2?), with 2500 displayed of 3000 available with room for 4096? I think this is supposed to be 24KB for 2 channels, as it's supposed to be quadrupled, but still unclear values. Another [FNIRSI 1013D teardown](https://www.cnx-software.com/2022/11/16/fnirsi-1013d-teardown-and-mini-review-a-portable-oscilloscope-based-on-allwinner-cpu-anlogic-fgpa/) shows EF2L45LG144B FPGA and gives its specifications as 35k DistributeRAM (bits, presumably) and 700k eRAM. 

Datasheet for [Altera Cyclone IV EP4CE6 (ep4ce6e22c8n)](https://www.altera-price.com/files/7f/EP4CE40F23C6N.pdf) used in earlier versions of FNIRSI 1013D says that has 270 Kbits of memory, which would be 33,75KB. However, [EEVblog post](https://www.eevblog.com/forum/fpga/reverse-engineering-anlogic-al3_10-fpga/) says this is Anlogic [AL3A10LG144C7](https://www.eevblog.com/forum/testgear/fnirsi-1013d-100mhz-tablet-oscilloscope/msg3885095/#msg3885095) FPGA. Memory could be used for other purposes too. Different models and builds are using different FPGA's, which may complicate things.

Based on [reverse-engineered schematics](https://github.com/pecostm32/FNIRSI-1013D-1014D-Hack/blob/main/Schematics/1014D/Scope_1014D_Data_Acquisition.png) FPGA's W25Q80 SPI flash has SPI interface at J2. Unfortunately I haven't had success reading the flash with J-Link (not supported) or BusPirate (everyting 0) yet. The JTAG connector on the same page appears to be connected to the FPGA, though not shown on the schematic. This should allow debugging and updating the FPGA on the fly, if facilitated by software.

Another post on the thread notes that reverse-engineering the FPGA revealed it doesn't have JTAG or ISP core like https://www.altera-price.com/files/51/DC-VIDEO-TVP5146N.pdf - MCU at https://github.com/pecostm32/FNIRSI-1013D-1014D-Hack/blob/main/Schematics/1014D/Scope_1014D_Processor.png doesn't have JTAG connection to the FPGA, so it'd have to be ISP over the shared data link anyway.
"Another drawback is the vertical sensitivity, which with only 100mV per division true range with a 1x probe setting it is not very good. The advertised 50mV per division is based on software zoom." Have not looked if it could be possible to increase the sensitivity.
https://www.eevblog.com/forum/testgear/fnirsi-1013d-100mhz-tablet-oscilloscope/msg4354150/#msg4354150 has some discussion on the analogic frontend and opamp.

https://github.com/emsec/hal was really pretty much first match I found, there might be better reverse-engineering tools. Also apparently it's already been reverse-engineered into Verilog, which might not have worked due to unconstrained clock domain crossings.

https://github.com/pecostm32/FNIRSI-1013D-1014D-Hack/tree/main/Test%20code/fnirsi_1014d_firmware_backup Instructions note there are some custom fields which need to be saved, so I cna use this? Or is everything on the SD/MMC card (which could break, so backup?)
https://github.com/pecostm32/FNIRSI-1013D-1014D-Hack/tree/main/Test%20code/fnirsi_1014d_startup_with_fel "FEL and DRAM setup", at least could get to boot? See is this differs from 1013d at all.

https://github.com/froloffw7/FNIRSI-1013D-1014D-Hack "SD card emulation", is this the "The new firmware is easy to install, without even writing over the original firmware, so one can easily revert back to it. Some even modified their scope with a switch to be able to go back and forth." referenced? Nope, according to https://www.eevblog.com/forum/testgear/fnirsi-1013d-100mhz-tablet-oscilloscope/msg4326355/#msg4326355 this is SD-card emulations for QEMU emulator.

The unique important part for 1014D is https://github.com/pecostm32/FNIRSI-1013D-1014D-Hack/blob/main/Schematics/1014D/Scope_1014D_User_Interface.png which gladly shows all the wiring - looks like it connects with RX/TX USART to the MCU, so first step could be reverse-engineering that protocol.

https://www.eevblog.com/forum/testgear/fnirsi-1013d-100mhz-tablet-oscilloscope/1250/ seems to identify the FPGA as AL3A10LG144C7 - other posts mention that apparently older 1013D have Altera, possibly with Altera markings. Can we assume all 1014D are Anlogic?

Anlogic IDE: https://dl.sipeed.com/shareURL/TANG/Premier/IDE 5.0.4
Mentioned: https://dl.sipeed.com/shareURL/TANG/Primer/IDE but that's 4.6.4
Reference: https://www.eevblog.com/forum/fpga/latest-version-of-anlogic-ide/
Translations: https://github.com/kprasadvnsi/Anlogic_Doc_English

Wow, https://www.eevblog.com/forum/fpga/reverse-engineering-anlogic-al3_10-fpga/msg4514072/#msg4514072 drops the verilog for the FPGA "some errors still". I need to have a look at that; assuming the FPGA is indeed the same.
