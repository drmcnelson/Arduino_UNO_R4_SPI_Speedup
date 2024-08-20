# Arduino_UNO_R4_SPI_Speedup
Patch the Arduino SPI library, true 16 bit transfers and add loop-friendly API

This repo provides a version of the SPI libarary for the Arduio UNO R4, which uses the  Renesas RA4M1 processor.

In the standard distribution, 16 bit transfers are done as back to back byte transfers, with a 1.2 usec setup time and 1.2 usec lost between transfers.  The transfer takes 4.8 usec to complete

In this version, the SPI.transfer16(uint16_t data) is implemented as a single 16 bit transfer.  Setup is still 1.2usec, but the transfer takes 3.6 usec to complete.

For transfers inside a loop, there are three new calls,  SPI.transfer16_setup(), SPI.transfer16_transfer(uint16_t data), and SPI.tansfer16_cleanup().  The transfer starts (the clock starts) at 600 nsec into the call, and each transfer takes 2.4 usec to complete.

If you want the old transfer16(), which transferred 16 bits as two one byte transfers,  it is still there.  It is now called SPI.transfer16_asbytes().

## Some things to be aware of.

### File names
The source files in the repo are named SPI.cpp and SPI.h.   These are the same as those in the standard SPI library in the Arduino distribution.   The intention is that these take the place of those for the UNO R4.  You may want to remove or rename the originals for the UNO R4 in your arduino tree.  On my computer, Fedora 40 with Cinnamon desktop, they are located at ~/.arduino15/packages/arduino/hardware/renesas_uno/1.2.0/libraries/SPI,  The version number may be different on your Linux machine.

### MOSI idle
In the original library, the idle state for MOSI was set to low.  That is corrected in this library.  It is now high when idle.   However, accessing the control register to setup 16 bit transfers, seems to cause MOSI to go low for a moment.  If you need to avoid that, you can try the three cals for loop friendly transfers; call SPI.transfer16_setup() and then call SPI.transfer16_transfer() for your transfers. When you are done, if you want to reset to 8 bit, call SPI.transfer16_cleanup().

## Comparison of old and new 16 bit transfers

### Orginal SPI.transfer16() as two one byte transfers.
Here is a scope trace of the original SPI.tansfer16().  The top trace is the SPI clock, the lower trace is a digital pin set high before the call and low immediately after.  As can be seen there is an extra 1.2 usec incurred by sending the data as two bytes.  The time after the clock stops is the time it takes for the SPI interface to return the received data to the calling routine.

<img src="https://github.com/drmcnelson/Arduino_UNO_R4_SPI_Speedup/blob/main/Images/UNO_R4_SPI_original.JPG" width="50%">

### New implementation as a single 16 bit transfer
Here is a scope trace of the new SPI.tansfer16(), now implemented as a single 16 bit transfer.  Again the top trace is the SPI clock, the lower trace is a digital pin set high before the call and low immediately after.  As can be seen the transfer not takes 3.6usec to complete.

<img src="https://github.com/drmcnelson/Arduino_UNO_R4_SPI_Speedup/blob/main/Images/UNOR4_SPI_speedup.JPG" width="50%">

### New loop-friendly API, true 16 bit transfers with minimal overhead inside the loop.
Here is a scope trace of the new API,  SPI.transfer16_setup() is called once to setup 16 bit transfers.
Inside the loop, SPI.tansfer16_transfer() is called for each transfer. 
At the conclusion, SPI.transfer16_cleanup() can be called to restore 8 bit operation.
Again the top trace is the SPI clock, the lower trace is a digital pin set high before the call and low immediately after.
As can be seen the setup inside the loop is down to 600 nsecs and the overall transfer takes 2.6 usec per word.

<img src="https://github.com/drmcnelson/Arduino_UNO_R4_SPI_Speedup/blob/main/Images/UNO_R4_SPI_loop.JPG" width="50%">


