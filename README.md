# Arduino_UNO_R4_SPI_Speedup
Patch the Arduino SPI library, true 16 bit transfers and add loop-friendly API

This repo provides a version of the SPI libarary for Arduino board such as the UNO R4 with the  Renesas processor

In the standard distribution, 16 bit transfers are done as back to back byte transfers, with a 1.2 usec setup time and 1.2 usec lost between transfers.  The transfer takes 4.8 usec to complete

In this version, the SPI.transfer16(uint16_t data) is implemented as a single 16 bit transfer.  Setup is still 1.2usec, but the transfer takes 3.6 usec to complete.

For transfers inside a loop, there are three new calls,  SPI.transfer16_setup(), SPI.transfer16_transfer(uint16_t data), and SPI.tansfer16_cleanup().

For the loop-friendly calls, setup inside the loop is reduced to 600 nsec and each transfer takes 2.4 usec to complete.

Simple installation, just copy SPI.cpp and SPI.h into your directory  packages/arduino/hardware/renesas_uno/1.2.0/libraries/SPI/

You may want to rename the files that are already there to something lile SPI.cpp_keep, SPI.h_keep

