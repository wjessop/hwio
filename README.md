# hwio

## Introduction

hwio is a library for interfacing with hardware I/O. It is loosely modelled on
the Arduino programming style, but deviating where that doesn't make sense in
go. It makes use of a thin hardware abstraction via an interface so a program
written against the library for say a beaglebone could be easily compiled to run
on a raspberry pi, maybe only changing pin references.

To use hwio, you just need to import it into your go project, initialise pins as
required, and then use functions that manipulate the pins.

Initialising a pin looks like this:

	myPin, err := hwio.GetPin("GPIO4")
	err = hwio.PinMode(myPin, hwio.OUTPUT)

Unlike Arduino, where the pins are directly numbered and you just use the number, in hwio
you get the pin first, by name. This is necessary as different hardware drivers may provide
different pins.

Writing a value to a pin looks like this:

	hwio.DigitalWrite(myPin, hwio.HIGH)

For more information about using the library, see http://stuffwemade.net/hwio which has:

 *	Pin diagrams for supported boards



## BIG SHINY DISCLAIMER

REALLY IMPORTANT THINGS TO KNOW ABOUT THIS ABOUT THIS LIBRARY:

 *	It is under development. If you're lucky, it might work. It should be considered
	Alpha.
 *	Currently it is limited to GPIO on BeagleBone and Raspberry Pi.
 *	Only GPIO is tested. Negative interactions with other system functions
	is not tested.
 *	If you don't want to risk frying your board, you can still run the
 	unit tests ;-)


## Board Support

Currently there are 3 drivers:

  *	BeagleBoneFSDriver - for BeagleBone boards running linux kernel 3.8 or higher, including
  	BeagleBone black
  *	BeagleBoneDriver - for BeagleBone boards running older SRM kernels (pre-device tree)
  *	RaspberryPiDriver - for Raspberry Pi

### BeagleBoneFSDriver

This driver accesses hardware via the device interfaces exposed in the file system on linux kernels 3.8 or higher, where
device tree is mandated. This should be a robust driver as the hardware access is maintained by device driver authors,
but is likely to be not as fast as direct memory I/O to the hardware as there is file system overhead.

Status:

  * In development
  * New driver, not yet properly untested
  * Some GPIOs are known to work on writes, unknown on reads
  * USR0-USR3 don't work and cannot be accessed as GPIO, as the LED driver reserves them.
  * Analog not yet supported


### BeagleBoneDriver

This driver accesses hardware directly using memory mapped I/O. It will only work on older SRM kernals (pre 3.8). It is generally
pretty quick as all non-setup functions access devices directly.

This driver is deprecated in favour of BeagleBoneFSDriver, as it is only useful for older kernels. Also, I don't have a BeagleBone with the older
kernel for testing any more, so I'm not in a position to test changes to this driver.

Status:

  * Alpha
  *	GPIO on all pins known to work for output and all input modes (pull variations)


### RaspberryPiDriver

This driver accesses hardware directly using memory mapped I.O.

Status:

  * Beta
  * GPIO on all pins known to work for input and output


## How it Works

Hardware drivers implement the interface that allows different devices to
implement the I/O handling as appropriate. This allows for drivers that use
different techniques to access the I/O. e.g. some drivers may use GPIO (a file
system approach), whereas for maximum performance another driver might use
direct memory access to I/O registers, but is less portable.

Some general principles the library attempts to adhere to include:

 *	Pin references are logical, and are mapped to hardware pins by the driver.
 *	Drivers provide pin names, so you can look them up by meaningful names
	instead of relying on device specific numbers.
 *	The library does not implement Arduino functions for their own sake if go's
	framework naturally supports them better, unless we can provide a simpler interface
 	to those functions and keep close to the Arduino semantics.
 *	Make no assumption about the state of a pin whose mode has not been set.
 	Specifically, pins that don't have mode set may not on a particular hardware
 	configuration even be configured as general purpose I/O. For example, many
 	beaglebone pins have overloaded functions set using a multiplexer. Any pin
 	whose mode is set by PinMode can be assumed to be general purpose I/O, and
 	likewise if it is not set, it could have any multiplexed behaviour assigned
 	to it. A consequence is that unlike Arduino, PinMode *must* be called before
 	a pin is used.
 *	The library should be as fast as possible so that applications that require
 	very high speed I/O should achieve maximal throughput, given an appropriate
 	driver.
 *	The library should be tolerant towards beginners, and give meaningful errors
 	when the hardware is requested to do things it can't. But this checking
 	should be able to be disabled for maximum performance.
 *	Make simple stuff simple, and harder stuff possible. In particular, while
 	Arduino-like methods have uniform interface and semantics across drivers,
 	we don't hide the driver itself, so special features of a driver can still
 	be used, albeit non-portably.
 *	Sub packages can be added as required that approximately parallel Arduino
 	libraries (e.g. perhaps an SD card package). Where possible, these
 	implementations should be generic across I/O pins. e.g. not assuming
 	specific pin behaviour as happens with Arduino.


### Pins

Pins are logical representation of physical pins on the hardware. To provide
some abstraction, pins are numbered, much like on an Arduino. Unlike Arduino,
there is no single mapping to hardware pins - this is done by the hardware
driver. To make it easier to work with, drivers can give one or more names to
a pin, and you can use GetPin to get a reference to the pin by one of those
names.

Each driver must implement a method that defines the mapping from logical pins
to physical pins as understood by that piece of hardware. Additionally, the
driver also publishes the capabilities of each pin, so that hwio can ensure
that constraints of the hardware are met. For example, if a pin implements PWM
in hardware, that is a capability. A method (e.g. motor driver) that requires
hardware PWM can ensure that a pin has the appropriate capability. Because each
pin can have a different set of capabilities, there is no distinction between
analog and digital pins as there is in Arduino; there is one set of pins, which
may support digital and/or analog capabilities.

The caller generally works with logical pin numbers retrieved by GetPin.


## Things to be done

 *	PWM output support (BeagleBone, R-Pi)
 *	Analog input (BeagleBone)
 *	Interupts (lib, BeagleBone and R-Pi)
 *	Serial support for UART pins (lib, BeagleBone and R-Pi)
 *	SPI support; consider augmenting ShiftIn and ShiftOut to use hardware pins
 	if appropriate (Beaglebone and R-Pi)
 *	define a way to model multi-pin capabilities. e.g LCD or MMC on BeagleBone
 *	LCD, particularly HD44780 (lib)
 *	Servo (lib)
 *	Stepper (lib)
 *	I2C (lib)
 *	TLC5940 (lib)
