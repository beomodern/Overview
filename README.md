# Overview

This whole thing started when I saw first time this [Beosound 9000](https://www.beoworld.org/prod_details.asp?pid=954) CD player designed and manufactured by Bang & Olufsen (BeO). At that time it looked amazingly, like some sort of spaceship (at least to me). 

Moving forward, few years back I manage to buy this player! It was faulty and I spent some time learning peculiarities that BeO engineers introduced inside this unit. In the end, I manage to bring it back to life and it is now fully working, every day used unit. But during that process of working on this unit, something changed… I caught a bug for BeO audio equipment. I especially started to like products designed by BeO designer Jacob Jensen. If you are interested, have a look [here](https://jacobjensendesign.com/bo) and [here](https://www.bang-olufsen.com/en/story/jacob-jensen-design-icon). That gentleman described his own design philosophy as “different but not strange” and I agree with him 100%. 

One of his designs, which was BeO response to far-east stackable sound systems was called BeoSystem 6500 (more [here](https://beocentral.com/beosystem6500) and [here](https://www.beoworld.org/prod_details.asp?pid=687)). It’s beautifully simple interface with spectacularly looking remote controller really makes an impression on everyone. I bought one of those (faulty again) and brought it back to life as well. It is great but it is a system designed in late eighties. Tapes are no longer popular sound carriers. Records definitely have characters but system on its own needed upgrade with access to more recent audio features. Keep in mind that Bluetooth interface was not even thought about when this unit was manufactured. I decided to something about it. 

Entering BeoModern concept. 
First, name. 
I was thinking about something that might work well with rest of the system. I talked with my wife what she thinks about it and she mention term “modern” I thought this is great. It would also be a tribute to exhibition that BeO held at NY MOMA in seventies displaying Jensen designs (more [here](https://www.moma.org/calendar/exhibitions/1786)). BeoModern name was born.

Second, what it is?
In terms of details what is supposed to be and do. I wanted this unit to:
1.	Be able to store and easily play mp3 and flac files that I collected over the years,
2.	Have ability to receive and play DAB radio stations,
3.	Play internet radio stations,
4.	Receive and play audio sent over Bluetooth (using latest BT codecs)
5.	Transmit audio played by audio system over Bluetooth (using latest BT codecs)
6.	Display FM radio station names and FM RDS information,
7.	Display clock. 
8.	Blend and look identical as rest of the system therefore preserving eighties look.
9.	Work seamlessly with existing unit and its remote controller interfaces.
When I thought about all of it I had vague idea of the challenge but it ended up actually much more difficult than I thought. I definitely learned great deal of different things while playing with it. At this point in time I’m probably 70% done with clear vision how to finish it. Now, it is just a matter of more time invested into it.  

In terms of what it already done.
On hardware side:
1.	Power supply solution. 
Since unit is quite multifunctional and build from different blocks (different EVBs and other pieces I had laying around), not all its blocks needs to be ON all the time. I designed PSU solution that is software controllable and enabled only blocks that are needed for particular mode of operation. 
2.	Raspberry Pi and DAB radio module. 
I had RPi 2. I also decided to use [uGreen DAB](https://ugreen.eu/product/ugreen-dab-board/) solution . Since it utilizes newer RPi interface I had to produce little interposer board in-between DAB module and RPi. I used that interposer to get access to I2S signals from DAB as well as for UART interface and GPIO signal to control RPi. On that board there is also small LDO that powers external LNA only that is only needed when DAB or FM radio is in use (again to save power).
3.	Audio interfaces.
I found this great EVB from ADI. It contains SigmaDSP uP [ADAU1452](https://www.analog.com/en/products/adau1452.html) as well as actually quite good audio codec [AD1938](https://www.analog.com/en/products/ad1938.html). Both will come very handy in this project. In general I want this EVB to handle all audio processing with analog signal being digested and produced by audiocodec and SigmaDSP to handle all digital audio (I2S and SPDIF). Unit will works as a standalone solution utilizing its on-board EEPROM to start up in pre-programmed state. 
4.	Bluetooth module.
Instead of writing my own code for some BT module I bought for few Euro, unit from eBay and interface its push-button inputs and LED status outputs with main uP. It works quite well. 
5.	Main uP.
I found at the bottom of my drawer few Cypress PSoC EVBs. They are old one, 4200 series but they are perfect for what I’m looking for. I hooked them up to control RPi, PSU distribution, Bluetooth module, SigmaDSP. On main PCB, I also added I2C buffer to deal with digital DATALINK interface that BeO products utilize to talk to each other.

On software side:
1.	Code for RPi 
I spent some time developing Python code that runs on RPi. Before code was written I gave a bit of thoughts how to architect it. There are some flow diagrams [here](https://github.com/beomodern/RPi_Python_code/blob/master/RPi_flowcharts.pdf)  I ended up utilizing state machine approach. This code is responsible for:
a.	scanning and navigation thru folder structure on SD card that stores mp3 and flac files,
b.	playing mp3 and flac files using python mpd,
c.	parsing uGreen software output files and controlling DAB radio system,
d.	playing and displaying status from internet radio stations (mpd utilized here as well),
e.	displaying FM station names and plan to listen to RDS from FM stations,
For all of that I put simple protocol in place that main uP uses to control and interact with RPi. It is located [here](https://github.com/beomodern/RPi_Python_code/blob/master/UART%20interface%20protocol%20between%20RPi%20and%20main%20uP.pdf)
On RPi, there is a SAMBA server running. General everyday interaction with system (adding new files, adding/modifying radio stations) is done by simple copy/paste approach. There is also SSH interface open for debug stuff. RPi is equipped with two Wi-Fi cards (2.4GHz and 5GHz) as well as Ethernet port. Code can be found on github [here](https://github.com/beomodern/RPi_Python_code) 
2.	Code for main uP
Main uP code is responsible for system housekeeping. There is a collection of flowcharts that attempts to explain its functionality available on github [here](https://github.com/beomodern/Main_uP/blob/master/Main_uP_flowcharts.pdf) In general it maintains system state, reacts to push buttons and remote control commands as well as sent information to display module. It interacts with SigmaDSP, RPi, BT module and PSU module. It also monitors and decoded BeO DATALINK commands. The Main_uP.JPG shows hardware interface implemented in PSoC. There is a whole auto generated datasheet [here](https://github.com/beomodern/Main_uP/blob/master/Main_uP_code_datasheet.pdf) Code can be found on github [here](https://github.com/beomodern/Main_uP) 



In terms of what still needs to be done.
On hardware side:
1.	Design PCB for display module. That PCB will house:
a.	3 x eight digit alphanumeric LED displays (from nineties to keep in line with the overall style). 
b.	number of LEDs to display system state – to keep in line with other BeoSystem designs.
c.	I2C controlled, LED driver,
d.	RTC module and DCF77 atomic clock AM signal receiver to produce accurate clock information
e.	photoresistor to maintain display brightness based on ambient conditions. 
f.	another PSoC 4200 series to handle all above
2.	Housing for whole system
The idea here is to use the same physical housing as any other BeoSystem unit. Best candidate is housing from Beomaster as it does not have any cuts in front for CD or anything like that. I bought one of those in the past. It was broken and I assumed I’ll just use chassis from it. I manage to fix it easily enough and now I don’t have guts to brake it down just for chassis. Hunt for properly broken unit continues.  

On software side:
1.	SigmaDSP code:
Write code to handle all audio routing properly (at the moment is simply in to out). Once written code should ended up in on-board EEPROM and be controlled by main uP over SPI interface. Control will include input selection and reading back status for spectrum and signal analyser display ideas. Frames for that code are on github [here](https://github.com/beomodern/SigmaDSP)
2.	Display uP code.
Write code that will handle:
-	parallel interfaces to 3 alphanumeric display modules. 
-	interact with RTC and DCF77 modules and code clock operation with them. 
-	interact over I2C interface with LED controller. 
-	handle photoresistor and use its readings to control brightness of both alphanumeric and LEDs displays. 
Display will be slaved and react to commands sent over SPI from main uP. Github placeholder for this code is [here](https://github.com/beomodern/Display_uP)

Once those bits are done, unit should be ready to go as a companion with other BeoSystem blocks. I hope Jacob Jensen and BeO engineers would be happy with it :)
