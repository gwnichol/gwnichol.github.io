---
layout: post
author: Grant Nichol
color:  indigo 
title:  "PSG Music Player"
date:   2019-09-28 20:50:00 -0600
cover: /img/2019-09-28-psg-music-player/cover.png
parmalink: /psg/
categories: ["Projects"]
---

I created a device that can read note and timing information from an SD card in SPI mode and control
a Programmable Sound Generator (PSG) to play video game music.

The video below has many examples of songs that I was able to add. This project used an AY-3-8912 PSG and an HCS08 microcontroller. This was done as a final project for my ECE431 microcontrollers class and was written entirely in assembly. There were many complex challenges to overcome and I ran into many problems that required unique solutions. I will not be posting the full code of the project publicly due to concerns of cheating, however you can contact me at [gwnichol@ksu.edu](mailto:gwnichol@ksu.edu) and I will gladly share it with you.


<iframe width="560" height="315" src="https://www.youtube.com/embed/z2n2TdfK7-o" frameborder="0" allow="accelerometer; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Full text

### Overview

I created a device that can read note and timing information from an SD card in SPI mode and control
a Programmable Sound Generator (PSG) to play the specified notes. I also created a supporting program
to convert from MIDI to the file format used by the device. Ultimately, I wanted to be able to play video
game music from the consoles I had growing up. I got the inspiration to do this project from the Nintendo
Entertainment System (NES) that I often play. I could have used a very similar set-up to run a digital
to analog converter, however, I liked the idea of using a PSG. Unfortunately, the sound generators that
consoles like this used are often integrated directly with the processing unit so I was not able to use the
exact generator that the NES uses. I ended up using the AY-3-8912 from General Instruments since it is one
of the few PSG chips still being sold from retailers. Also, its packaging made it easier to use on a breadboard.

### Design Strategy

Before buying anything, I poured through all the datasheets I had for my main components to make
sure that the project would even work. After determining that it would probably work, I bought the AY-3-
8912 PSG and a 2MHz TTL-compatible clock to provide timing to the PSG. I also bought an extra SD card
just in case I broke one. Luckily, I already had all the of passives needed for the amplifier circuit to drive a
speaker. I was able to use an amplifier circuit that I found in the programmers manual for the PSG.

The process of designing this project consisted of three main parts: writing to the PSG, reading from
the SD card, and putting it all together.

### Writing to the PSG

The primary challenge when writing to the PSG is the non-standard communication protocol it uses.
It has a simplified control of two control pins and eight data pins. To write to a specific register on the PSG,
I need to first send it the NACT command. After that, I send the INTAK command and put the register
address on the data lines. After that, I put it back in NACT mode. Next, I place the new value of that
register on the data lines and send the DWS command when I?m done. After some delay, I put it back in
NACT mode.

The datasheets explain what register does what so knowing what to write to the PSG is simply a matter
of what notes I want to play. The PSG allows me to set the period and amplitude of three different square
wave channels. It also allows me to write the period and amplitude of a noise generator for _drum_ effects.
One of the complications I had when programming a test code for the PSG was that all the registers are
listed in octal on the datasheet. I made the assumption that they would be decimal and it took me a long
time to correct that mistake..

I finished some code to write a chord to the PSG and verified that it sounded about correct before
continuing on to the SD card.

### Initializing and Reading from the SD Card

I would say that the most complicated portion of this whole project was initializing the SD card. I
needed to use SPI to communicate with the SD card because it does not have very strict timing requirements.
That allows me to essentially read the SD card one byte at a time even though it is designed to be read in
512 byte increments. However, there are some hurdles to cross when initializing the card. It requires a very
specific sequence of commands and I need to send it _dummy_ bytes before any command so it can utilize
1the SPI clock. I need to do various things to make sure the card is working correctly and that it is a HCSD
so that I can give it addresses as a block number. The specific protocol is listed in the SD specification.

One of the first mistakes I made when testing the SD card was using a breakout designed for a 5 V
micro-controller and giving it 3.3 V. I was occasionally able to initialize the SD card, but it was unreliable.
SD card are typically 3.3 V devices, so I ended up soldering wires to a full sized SD adapter so I could plug
a micro-sd into the breadboard as seen in the video. That successfully resolved the initialization issue.

Another complication was the SPI peripheral on the HCS08. I wanted to use the built in chip select
to talk to the SD card, however, I needed the chip select to be disabled while sending the card _dummy_
bytes. I learned that the SPI module raises the transmit buffer empty flag before it is actually done sending
because of the way it is double buffered. Because of that, the only way I found to reliably know the timing
of the transmit action was to clear the receive buffer full flag before sending, then wait for that flag to raise
high after the transmit. This is possible because the receive buffer fills as the SCLK pulses. By waiting for
it to go high, I know that eight clock pulses happened on SCLK and that eight bits were sent. (Incidentally,
this helped combine the transmit and receive actions into the same subroutine.)

Reading from the SD card needs to happen in 512 byte increments and I need to start the read of a
block with the CMD17 command. I created a subroutine that keeps track of all the reading using a four
byte _SD next block_ variable and a two byte _SD next byte_ variable that it uses to know when to start
new block reads. This subroutine level allows me to not have to worry about it in the rest of the code and
simply call that subroutine whenever I want a new byte.

### Putting it all Together

At first, I figured I would manually load the shift register after reading from the SD card. However, I
realized that I was directly copying the exact values read to the shift register. That allowed me to load the
shift register at the same time as reading from the SD card, saving a lot of time. I, then, needed to change
my file format to facilitate that. A _file_, as written in this paper, does not correspond to a file system but
is actually just a collection of bytes placed at predefined offsets in the SD card. Each of the sixteen songs
that can be played are placed at 256 block offsets. Each song corresponds to the third byte of the block
number and the fourth byte corresponds to a block in the song.

The file format is declared as follows:

The first two bytes are the number of _notes_ that should be expected in the file. The next two bytes
are the settings of MTIMCLK PS and MTIMMOD respectively. The timing values in the file allow me to
change timing as desired in the file. Following the first four bytes are the collection of notes. Each note
starts with the start byte 0xA4. Following that is the one-byte number of PSG register updates. Each update
consists of the one byte register number then the data to be placed in the register. All those updates are
3written sequentially to the PSG as they are read from the SD. The next two bytes after the updates is the
number of MTIM overflows until the next _note_

I added a button to control starting and stopping the song. This button is debounced in the MTIM
overflow interrupt with it only triggering as it goes from un-pushed to pushed. I also added four switches
that are used to determine which song to play. This allows the aforementioned 16 available to play.

The complete project was put on a breadboard as seen in Figure 3. Various other components were
needed in the final circuit such as logic-level-shifters and the voltage supply.

### Testing and Verification

Testing was the fun part. To properly test actual songs without me needing to manually set each byte
in a hex editor, I created a Cpp program to convert from MIDI to the file format I created. The program I
made was very limited in the sense that it could only understand MIDI files that were separated into three
tracks with one note at a time per track. I can use a fourth track for the _drums_ but I found it should be
used very sparingly. Verifying that it works correctly was simply whether the music sounds correct. As long
as the limitations of my converter were met, I was able to play songs that sounded good. The best way to
show that it works would be with video, so it is available above.

### Wrapping Up

Overall, I believe that this project was successful and that I achieved what I set out to do. If I had
more time I may have figured out more that I could do with the envelope generator on the PSG. With it, I
could have made waveforms more complex than simple square waves. 


