---
layout: post
author: Grant Nichol
color:  amber
title:  "Bitfields in C"
date:   2019-01-31 12:00:00 -0600
categories: c
---

Apparently there's such a things at _bitfields_ which would allow me to name parts of a byte.
I'm still trying to find out how exactly to apply them, 
but I'm hoping to be able to use them for my microcontroller programming as a way to have more
readable code than doing a bunch of bitwise operations and macros. 

Bitfields can only be defined in a struct so I will do:
```c
struct portB
{
   uint8_t 0: 1;
   uint8_t 1: 1;
   uint8_t 2: 1;
   uint8_t 3: 1;
   uint8_t 4: 1;
   uint8_t 5: 1;
   uint8_t 6: 1;
   uint8_t 7: 1;
}

void main () {
   struct portB outPort;
   outPort.1 ^= 1; // Toggles bit 1
   while(1);
}
```
Unfortunately I can not expect the bitfields to be in any normal order so
I'm going to find a way to use them more reliably.

_I will update this post as I learn more about them_

