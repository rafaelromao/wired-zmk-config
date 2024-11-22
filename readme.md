# Wired ZMK Config

This repo is a simple example of a ZMK Config for a wired split keyboard using serial communication over the UART protocol, in a Seeeduino Xiao RP2040.

## Wired PR

First of all, wired communication between the halves is not yet officially supported by ZMK, so you will need to use an [open PR branch](https://github.com/xudongzheng/zmk/tree/split-serial-pr) to make it work. This work is still in draft at the moment of this writing, so use it at your own risk.

## Keyboard Name

This example defines a custom keyboard named *Wired Diamond*. This is the keyboard I built and used to test this code, but in terms of pinout, it is basically a 3x5+2 board.

## Configuration

The configurations below are limited to the Seeeduino Xiao RP2040, and may differ from other MCUs. It is also assuming the default UART configuration supported by the MCU, so minimal configuration changes are required.

### Enabling split serial

A first step is to tell ZMK that the shields form a split keyboard and that the halves communicate with each other using serial communication.
This is done in the `.defconfig` file:

```
if SHIELD_WIRED_DIAMOND_LEFT || SHIELD_WIRED_DIAMOND_RIGHT

config ZMK_SPLIT
    default y

config ZMK_SPLIT_SERIAL
	default y

config SERIAL
    default y

endif
```

### Enabling serial communication

By default, serial communication is disabled in the MCU so it is also required to enable it:

In the `.dtsi` file:
```
&xiao_serial { status = "okay"; };
```

It is also necessary to enable serial communication in the `.conf` files:

```
CONFIG_SERIAL=y
```

### Enabling UART

The next step is to enable the UART protocol and configure it for serial communication between the halves:

In the `.conf` files:

```
CONFIG_UART_INTERRUPT_DRIVEN=y
CONFIG_UART_CONSOLE=n
```

Note that the logging to the console via UART should also be disabled, since it is enabled by default.

In the `.dtsi` file, you also need to enable split uart, and tell ZMK which pinout configuration will be used.

The easiest way to do it is to use the default UART pins for TX and RX, which in this case is P0/D6 and P1/D7. Assuming that, all that is left is:

```
    chosen {
        ...
        zmk,split-uart = &uart0;
    };
```

**This is all that you need to have wired communication working for this MCU and pinout configuration.**

Continue in the section below to understand what else is necessary but implicitly set.

### Inherited settings

For the settings above to work, the following inherited settings are assumed:

In the ZMK repo file `boards/arm/seeeduino_xiao_rp2040/seeeduino_xiao_rp2040.dts`:

```
&uart0 {
	current-speed = <115200>;
	status = "okay";
	pinctrl-0 = <&uart0_default>;
	pinctrl-names = "default";
};
```

In the ZMK repo file `boards/arm/seeeduino_xiao_rp2040/seeeduino_xiao_rp2040-pinctl.dtsi`:

```
&pinctrl {
	uart0_default: uart0_default {
		group1 {
			pinmux = <UART0_TX_P0>;
		};
		group2 {
			pinmux = <UART0_RX_P1>;
			input-enable;
		};
	};
    ...
}
```

### Pinout

For the physical connection between the halves, I connected the pins 3v3 to 3v3, GND to GND, D6 to D7 and D7 to D6.