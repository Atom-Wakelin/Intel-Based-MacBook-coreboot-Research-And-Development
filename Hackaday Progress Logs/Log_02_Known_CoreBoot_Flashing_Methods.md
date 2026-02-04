<p>While I was setting up the MacBook, I was also simultaneously conducting research on the various flashing methods for the A1278. There were a handful of similar models between 2010-2012 that, due to differences in chip layout, had very different known access vectors for Coreboot.</p>

<p><strong>For example, while the A1278 (my target machine) is considered difficult to flash externally, users have been able to reliably externally flash the A1466, which was released around the same period.</strong></p>


<p>An extremely helpful reference that I used in my research was the thorough MacBook Coreboot documentation uploaded to<strong> ch1p.io</strong> written by <strong>Evgeny Sorokin</strong> (Also known as "gch1p" on GitHub) and partly by&nbsp;<strong>Sebastian Grzywna</strong>, (AKA swiftgeek), a Coreboot contributor.</p>


<p><strong>If you are planning to Coreboot your MacBook, I implore you to read the references below:</strong><br></p>


<p><a href="https://ch1p.io/coreboot-macbook-support/" target="_blank">On the current state of coreboot on MacBooks</a></p>


<p><a href="https://ch1p.io/coreboot-macbook-internal-flashing/" target="_blank">Flashing coreboot on MacBooks without external programmer using IFD hack</a></p>


<p>This documentation was extremely helpful and provided a great amount of insight into dos and don'ts of the flashing process. It stopped me from attempting an external flash that would have damaged the Logic Board beyond repair and potentially bricked the laptop. This project would probably not be possible without the documentation as a guide.</p>


<p>In this log, I will give a brief summary of each known flashing method's status on the Macbook A1278, using information from <strong>ch1p.io</strong> and from various internet sources.</p>


<p><strong><em>MMGA</em></strong></p>


<p><a href="https://github.com/gch1p/mmga">MMGA was a bash script written by Evgeny Sorokin,</a> designed to be used to automate internal flashing of Coreboot without attaching an external programmer to the motherboard. <strong>Sorokin has stated multiple times both on GitHub and on ch1p.io that he does not recommend its use,</strong> particularly because many users saw it as a simple script that would automate the entire Coreboot process for them, and didn't pay attention to the rest of the warnings in his documentation and information needed to make it work properly. As a result, some people who used MMGA may have bricked their laptops because they trusted MMGA to automate everything for them. <strong>It is for this reason that Sorokin has recommended MANUALLY running the commands in MMGA ourselves through <em>Internal Flashing.</em></strong></p>


<p><strong><em>INTERNAL FLASHING</em></strong></p>


<p><strong>Internal Flashing</strong> on MacBooks is essentially flashing Coreboot to the motherboard without having to take it apart and attach an external programmer. According to Sorokin, it can be accomplished on some MacBooks (including the A1278, my target machine) with <strong>just Linux installed and root access.&nbsp;</strong>In Sorokin's process, the <strong>Intel_ME</strong> region on the target firmware chip is read using a bash application called "FlashROM", then shrunken from 8 Megabytes to only 90 Kilobytes using a <strong>Python script</strong> called ME_Cleaner. From there, the remaining space can be used to flash Coreboot to the chip.</p>


<p><strong><em>EXTERNAL FLASHING</em></strong></p>


<p>External flashing typically involves taking apart the computer itself to gain access to the motherboard (or Logic Board, as Apple calls their motherboards) and attaching an external programmer. (example: Raspberry Pi) Typically, an external programmer can be connected to a motherboard's BIOS/UEFI chip or the specific target chip with an SOIC clip that latches onto the chip legs to ensure a strong connection to the chip. From there, the clip can have jumper wires connected to it that run into an external programmer (there are lots out there) or a Raspberry Pi, to flash the payload onto the chip.</p>


<p><img alt="5250 Pomona Electronics | Test and Measurement | DigiKey" style="width: 281px; height: 281px;" width="281" height="281" title="5250 Pomona Electronics | Test and Measurement | DigiKey" class="float-left lazy" src="https://mm.digikey.com/Volume0/opasdata/d220001/medias/images/922/5250.JPG?hidebanner=true"></p>


<figcaption>(LEFT) An example SOIC-8 Clip from Pomona Electronics. The small teeth in the jaw have contacts to latch onto the chip legs, while the jumper ports on the backside are used to attach wires.<br><br></figcaption>


<p><img alt="Flashing coreboot on MacBook Pro 10,1 using external SPI programmer" style="width: 311px; height: 206.448px;" width="311" height="206.448" title="Flashing coreboot on MacBook Pro 10,1 using external SPI programmer" class="float-right lazy" src="https://f.ch1p.io/l4bjmeSn/pro101_clip.jpg"><br></p>


<figcaption>(RIGHT) A Raspberry Pi being used as an external programmer, with an SOIC clip connecting it to the chip.</figcaption>


<p><strong><br></strong></p>


<p><strong><br></strong></p>


<h1>What this means for the MacBook A1278:</h1>


<p>For my target machine, the MacBook A1278, <strong>external flashing is very difficult and not recommended.</strong> The components surrounding the target BIOS/UEFI chip are extremely fragile, which means it is very easy to damage them when attaching a clip. Also, there is a small surface-mount chip that prevents attaching a clip entirely without cutting off a corner of the plastic on the clip.&nbsp;Most importantly, flashing the chip while it is still attached to the board will power the surroundings in ways that were not intended by Apple, so it could permanently damage the hardware. <strong>Sorokin did hint that desoldering the chip to isolate it from the rest of the board, flashing, and then resoldering it back onto the board was possible, but this is still a very cumbersome process, especially because the chip is so small.</strong></p>


<figure><img style="width: 428px; height: 410px;" width="428" height="410" class="lazy" src="https://cdn.hackaday.io/images/2108301765264333966.png"></figure>


<p>For me, all of these factors heavily discouraged me (for good reason) from attempting an external flash on my machine.</p>


<p>This left me with an obvious first choice of flashing method: <strong>Internal Flashing.&nbsp;</strong>Internal flashing has been tested previously on the MacBook A1278 and a handful of other models from that era, and it seems to be the method with the highest rate of success. In a worst-case scenario, the chip will have to be recovered through external flashing if all else fails.</p>


<p><strong></strong>In summary,&nbsp;<strong>Internal Flashing</strong> will be the first Coreboot installation method that I will be attempting in this project, because of the available documentation, support for my specific target machine model, and it does not require external hardware. (just Linux and root access)</p>


<p>I will be posting logs on a semi-frequent basis as I move into the flashing stage. I think I will post a few more detailing the set-up process as well. (and some hardware do's and don'ts)</p>