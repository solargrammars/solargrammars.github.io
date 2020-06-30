---
layout: post
title: "Macropad Building Instructions"
author: "Solar Grammars"
categories: notes
tags: [notes]
image: blog/learn-color-reps/gre_set10.png
color: "#000000"
---

[SU120](https://github.com/e3w2q/su120-keyboard) appears to be an extremely 
flexible PCB for unleashing our creativity in terms of mechanical 
keyboard crafting. Here I will 
quickly summarize the steps I followed to build a very basic
macropad, just scratching the surface of what seems to be possible.

I got the necessary components from [TALP](https://talpkeyboard.stores.jp/), 
which included the SU120 PCB, a Pro Micro, diodes, Kaih PCB sockets, a rotary 
encoder and a tact switch button.


![all](/assets/img/blog/macropad/all.jpg)


The first step is to solder the diodes. 

![diode1](/assets/img/blog/macropad/diode1.jpg)


Try to blend them in a uniform way.
![diode2](/assets/img/blog/macropad/diode2.jpg)

![diode3](/assets/img/blog/macropad/diode3.jpg)

The black mark of the diode needs to be aligned to the diagram in the PCB. 
![diode4](/assets/img/blog/macropad/diode4.jpg)

Add some tape to fix them before soldering. 
![diode5](/assets/img/blog/macropad/diode5.jpg)

Solder them and cut (here they are still long, as I didnt know how much you could go).
![diode6](/assets/img/blog/macropad/diode6.jpg)

Next, we solder the Kaih sockets. These will allow to make the PCB hot swappable so we 
can replace the switches more easlily.

![kaih1](/assets/img/blog/macropad/kaih1.jpg)

![kaih2](/assets/img/blog/macropad/kaih2.jpg)

When soldering, try to keep it as aligned as possible, otherwise it will impact onthe arragement
of the switches.
![kaih3](/assets/img/blog/macropad/kaih3.jpg)

Next,the tact switch button, which will be used as reset button for the PCB. No much to do here.
![reset1](/assets/img/blog/macropad/reset1.jpg)
![reset2](/assets/img/blog/macropad/reset2.jpg)

Almost there, the Pro Micro.

![micro1](/assets/img/blog/macropad/micro1.jpg)
![micro2](/assets/img/blog/macropad/micro2.jpg)
![micro3](/assets/img/blog/macropad/micro3.jpg)

The rotary encoder has its own position, and can be soldered in the same way a normal switch

![rotary1](/assets/img/blog/macropad/rotary1.jpg)
![rotary2](/assets/img/blog/macropad/rotary2.jpg)

Now the switches. For this one, I combined some Creams with Gateron reds 
![sk1](/assets/img/blog/macropad/sk1.jpg)
![sk2](/assets/img/blog/macropad/sk2.jpg)

	
And finally some keycaps
![caps1](/assets/img/blog/macropad/caps1.jpg)

To protect the PCB, you can put some rubber feet
![rubber](/assets/img/blog/macropad/rubber.jpg)

Or, like in my case, make a quick wooden case 
![wood](/assets/img/blog/macropad/wood.jpg)

Having finshed the hardware part, let's move to cofigure the
actions associated to each key. For this, we will rely on [QMK](https://docs.qmk.fm/#/).

On a mac, installation is mostly reduced to 

```bash
brew install qmk/qmk/qmkeee

qmk setup
```
...


On the `rules.mk`, make sure to add support for the rotary encoder:

```
...
ENCODER_ENABLE = yes
...
```
Another important setting needs to be done on the `config.h` file, where we need to define 
the position of the encoder:

```
#define ENCODERS_PAD_A { F5, B5 }
#define ENCODERS_PAD_B { F4, B4 }
```
and also set the resolution :

```
#define ENCODER_RESOLUTION 4
```

As a basic functionality for the encoder, lets associate it with the sound volume. To do that,
we need to define a custom function in the `keymaps/keymap.c` file:

```
void encoder_update_user(uint8_t index, bool clockwise) {
	if (index == 0) { // Considers one encoder, you can add more
		if (clockwise) {
			tap_code(KC_VOLU);
		} else {
			tap_code(KC_VOLD);
		}
	}
}

```
	

Once you are done with the settings, it is time to compile and write. Execute the following command:

```
make handwired/su120/rev1_4knob:avrdude
```

You should see some output like this:


```
(base) ➜  qmk_firmware git:(master) ✗ make handwired/su120/rev1_4knob:avrdude
QMK Firmware 0.9.10
Making handwired/su120/rev1_4knob with keymap default and target avrdude

avr-gcc (Homebrew AVR GCC 8.4.0) 8.4.0
Copyright (C) 2018 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

Size before:
   text   data    bss    dec    hexfilename
         0  24770      0  24770   60c2.build/handwired_su120_rev1_4knob_default.hex

	 Compiling: keyboards/handwired/su120/rev1_4knob/rev1_4knob.c                                        [OK]
	 Compiling: keyboards/handwired/su120/rev1_4knob/keymaps/default/keymap.c                            [OK]
	 Compiling: quantum/quantum.c                                                                        [OK]
	 Compiling: quantum/keymap_common.c                                                                  [OK]
	 Compiling: quantum/keycode_config.c                                                                 [OK]
	 Compiling: quantum/matrix_common.c                                                                  [OK]
	 Compiling: quantum/matrix.c                                                                         [OK]
	 Compiling: quantum/debounce/sym_g.c                                                                 [OK]
	 Compiling: quantum/color.c                                                                          [OK]
	 Compiling: quantum/rgblight.c                                                                       [OK]
	 Compiling: quantum/process_keycode/process_rgb.c                                                    [OK]
	 Compiling: drivers/avr/ws2812.c                                                                     [OK]
	 Compiling: quantum/led_tables.c                                                                     [OK]
	 Compiling: quantum/encoder.c                                                                        [OK]
	 Compiling: quantum/process_keycode/process_unicodemap.c                                             [OK]
	 Compiling: quantum/process_keycode/process_unicode_common.c                                         [OK]
	 Compiling: quantum/process_keycode/process_space_cadet.c                                            [OK]
	 Compiling: quantum/process_keycode/process_magic.c                                                  [OK]
	 Compiling: quantum/process_keycode/process_grave_esc.c                                              [OK]
	 Compiling: tmk_core/common/host.c                                                                   [OK]
	 Compiling: tmk_core/common/keyboard.c                                                               [OK]
	 Compiling: tmk_core/common/action.c                                                                 [OK]
	 Compiling: tmk_core/common/action_tapping.c                                                         [OK]
	 Compiling: tmk_core/common/action_macro.c                                                           [OK]
	 Compiling: tmk_core/common/action_layer.c                                                           [OK]
	 Compiling: tmk_core/common/action_util.c                                                            [OK]
	 Compiling: tmk_core/common/print.c                                                                  [OK]
	 Compiling: tmk_core/common/debug.c                                                                  [OK]
	 Compiling: tmk_core/common/sendchar_null.c                                                          [OK]
	 Compiling: tmk_core/common/util.c                                                                   [OK]
	 Compiling: tmk_core/common/eeconfig.c                                                               [OK]
	 Compiling: tmk_core/common/report.c                                                                 [OK]
	 Compiling: tmk_core/common/avr/suspend.c                                                            [OK]
	 Compiling: tmk_core/common/avr/timer.c                                                              [OK]
	 Compiling: tmk_core/common/avr/bootloader.c                                                         [OK]
	 Assembling: tmk_core/common/avr/xprintf.S                                                           [OK]
	 Compiling: tmk_core/common/magic.c                                                                  [OK]
	 Compiling: tmk_core/common/command.c                                                                [OK]
	 Compiling: tmk_core/protocol/lufa/lufa.c                                                            [OK]
	 Compiling: tmk_core/protocol/usb_descriptor.c                                                       [OK]
	 Compiling: tmk_core/protocol/lufa/outputselect.c                                                    [OK]
	 Compiling: lib/lufa/LUFA/Drivers/USB/Class/Common/HIDParser.c                                       [OK]
	 Compiling: lib/lufa/LUFA/Drivers/USB/Core/AVR8/Device_AVR8.c                                        [OK]
	 Compiling: lib/lufa/LUFA/Drivers/USB/Core/AVR8/EndpointStream_AVR8.c                                [OK]
	 Compiling: lib/lufa/LUFA/Drivers/USB/Core/AVR8/Endpoint_AVR8.c                                      [OK]
	 Compiling: lib/lufa/LUFA/Drivers/USB/Core/AVR8/Host_AVR8.c                                          [OK]
	 Compiling: lib/lufa/LUFA/Drivers/USB/Core/AVR8/PipeStream_AVR8.c                                    [OK]
	 Compiling: lib/lufa/LUFA/Drivers/USB/Core/AVR8/Pipe_AVR8.c                                          [OK]
	 Compiling: lib/lufa/LUFA/Drivers/USB/Core/AVR8/USBController_AVR8.c                                 [OK]
	 Compiling: lib/lufa/LUFA/Drivers/USB/Core/AVR8/USBInterrupt_AVR8.c                                  [OK]
	 Compiling: lib/lufa/LUFA/Drivers/USB/Core/ConfigDescriptors.c                                       [OK]
	 Compiling: lib/lufa/LUFA/Drivers/USB/Core/DeviceStandardReq.c                                       [OK]
	 Compiling: lib/lufa/LUFA/Drivers/USB/Core/Events.c                                                  [OK]
	 Compiling: lib/lufa/LUFA/Drivers/USB/Core/HostStandardReq.c                                         [OK]
	 Compiling: lib/lufa/LUFA/Drivers/USB/Core/USBTask.c                                                 [OK]
	 Linking: .build/handwired_su120_rev1_4knob_default.elf                                              [OK]
	 Creating load file for flashing: .build/handwired_su120_rev1_4knob_default.hex                      [OK]
	 Copying handwired_su120_rev1_4knob_default.hex to qmk_firmware folder                               [OK]
	 Checking file size of handwired_su120_rev1_4knob_default.hex                                        [OK]
	  * The firmware size is fine - 25098/28672 (87%, 3574 bytes free)
	  Detecting USB port, reset your controller now...................
	  Device /dev/tty.usbmodem1421 has appeared; assuming it is the controller.
	  Waiting for /dev/tty.usbmodem1421 to become writable.

	  Connecting to programmer: .
	  Found programmer: Id = "CATERIN"; type = S
	      Software Version = 1.0; No Hardware Version given.
	      Programmer supports auto addr increment.
	      Programmer supports buffered memory access with buffersize=128 bytes.

	      Programmer supports the following devices:
	          Device code: 0x44

		  avrdude: AVR device initialized and ready to accept instructions

		  Reading | ################################################## | 100% 0.00s

		  avrdude: Device signature = 0x1e9587 (probably m32u4)
		  avrdude: NOTE: "flash" memory has been specified, an erase cycle will be performed
		           To disable this feature, specify the -D option.
			   avrdude: erasing chip
			   avrdude: reading input file ".build/handwired_su120_rev1_4knob_default.hex"
			   avrdude: input file .build/handwired_su120_rev1_4knob_default.hex auto detected as Intel Hex
			   avrdude: writing flash (25098 bytes):

			   Writing | ################################################## | 100% 1.93s

			   avrdude: 25098 bytes of flash written
			   avrdude: verifying flash memory against .build/handwired_su120_rev1_4knob_default.hex:
			   avrdude: load data flash data from input file .build/handwired_su120_rev1_4knob_default.hex:
			   avrdude: input file .build/handwired_su120_rev1_4knob_default.hex auto detected as Intel Hex
			   avrdude: input file .build/handwired_su120_rev1_4knob_default.hex contains 25098 bytes
			   avrdude: reading on-chip flash data:

			   Reading | ################################################## | 100% 0.24s

			   avrdude: verifying ...
			   avrdude: 25098 bytes of flash verified

			   avrdude: safemode: Fuses OK (E:FB, H:D8, L:FF)

			   avrdude done.  Thank you.

```

