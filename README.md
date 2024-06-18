# ZMK VIK module

This is a ZMK module that allows you to add [VIK support](https://github.com/sadekbaroudi/vik) to a keyboard and/or use [VIK modules](https://github.com/sadekbaroudi/vik/tree/master/pcb) within ZMK.

## Using this module

To include the module, you'll need to add it to your zmk-config `config/west.yml`

It should look something like what's shown below. I've commented out things that you probably already have in the `config/west.yml`. You just need to add the parts that are uncommented.

```
manifest:
  remotes:
    # You'll likely have this already with your preferred ZMK version
    # - name: zmkfirmware
    #   url-base: https://github.com/zmkfirmware
    - name: sadekbaroudi
      url-base: https://github.com/sadekbaroudi
  projects:
    # You'll likely have this already with your preferred ZMK version
    # - name: zmk
    #   remote: zmkfirmware
    #   revision: main
    #   import: app/west.yml
    - name: zmk-fingerpunch-vik
      remote: sadekbaroudi
      revision: main
      import: config/deps.yml
```

Once you've done that, you can use any of the VIK modules that you have connected to it. For example, here is an example `build.yml`:

```
---
include:
  - board: <YOUR_CONTROLLER>
    shield: <YOUR_KEYBOARD> vik_cirque_spi
```

The above contents will build your keyboard with an attached cirque trackpad.

### VIK SPI module usage

If you are using any shield that uses SPI, you will need to also use the `vik_spi_adapter` shield in front of the VIK module shield

For local builds, example below. Notice the `vik_spi_adapter` that comes before the `vik_cirque_spi`:  
`west build --pristine -b "pinkies_out_v3@3.0.0" -- -DSHIELD="vik_spi_adapter vik_cirque_spi" -DZMK_EXTRA_MODULES='/home/sadek/zmk-fingerpunch-keyboards;/home/sadek/zmk-fingerpunch-controllers;/home/sadek/zmk-fingerpunch-vik;/home/sadek/cirque-input-module'`

For github actions builds, examples below:
```
  - board: xivik@0.1.0
    shield: arachnophobe vik_spi_adapter vik_cirque_spi
  - board: vulpes_majora_v1
    shield: vik_spi_adapter vik_pmw3360_per56
```

## Examples

For a whole list of examples of this, please see the [zmk-fingerpunch-keyboards](https://github.com/sadekbaroudi/zmk-fingerpunch-keyboards) repository.

## Adding VIK support to a keyboard

In order to add VIK support to a keyboard, you need to define the `vik_connector`, along with `vik_spi`, `vik_i2c`, and (optionally) `vik_spi_pmw3610` if you want to support the [VIK pmw3610 trackball module](https://github.com/sadekbaroudi/vik/tree/master/pcb/pmw3610)

Here is an example vik connector definition for vikoto. In case this readme is out of date, you can take a look at the [vik_connector.dtsi](https://github.com/sadekbaroudi/zmk-fingerpunch-controllers/blob/main/boards/arm/vikoto/vik_connector.dtsi).

```
/ {
	vik_conn: vik_connector {
		compatible = "sadekbaroudi,vik-connector";
		#gpio-cells = <2>;
		gpio-map-mask = <0xffffffff 0xffffffc0>;
		gpio-map-pass-thru = <0 0x3f>;
		gpio-map
			= <0 0 &gpio0 23 0>		/* vik SDA */
			, <1 0 &gpio0 19 0>		/* vik SCL */
			, <2 0 &gpio0  7 0>		/* vik RGB Data */
			, <3 0 &gpio0  4 0>		/* vik AD_1 */
			, <4 0 &gpio0 27 0>		/* vik MOSI */
			, <5 0 &gpio0  5 0>		/* vik AD_2 */
			, <6 0 &gpio0  6 0>		/* vik CS */
			, <7 0 &gpio1  9 0>		/* vik MISO */
			, <8 0 &gpio0 15 0>		/* vik SCLK */
			;
	};
};

vik_i2c: &i2c1 {};
vik_spi: &spi3 {};
vik_spi_pmw3610: &spi1 {};
```

In the example above, it's based on a controller that has an integrated VIK connector. However, you can also add VIK support to a keyboard like [vulpes minora](https://github.com/sadekbaroudi/vulpes-minora). In this case, the keyboard is wired to work directly from a nice!nano pinout, to a VIK connector on the PCB.

There is also an example of the VIK connector definition, as shown in the [nice_nano_v2.overlay](https://github.com/sadekbaroudi/zmk-fingerpunch-keyboards/blob/main/boards/shields/vulpes_minora/boards/nice_nano_v2.overlay) for that board.

### VIK SPI support

**IMPORTANT NOTE:**  
When adding SPI support, if you have an existing SPI device on your keyboard, you need to let the VIK module know where to start the register (`reg` value), and you need to give it the existing `cs-gpios`.

For the spi bus that is being used by VIK, you can do it as follows:
```
#define VIK_SPI_REG_START 1
#define VIK_SPI_CS_PREFIX <&gpio0 21 GPIO_ACTIVE_LOW>
```

For a full example, see the `pinkies_out_v3` board [here](https://github.com/sadekbaroudi/zmk-fingerpunch-keyboards/blob/main/boards/arm/pinkies_out_v3/pinkies_out_v3.dts), also pasted below.

```
// Need to define this as 1, since we have an existing SPI device on this board
// Starting point is normally 0, so we need to tell the vik_spi_adapter to start at 1 since
// there is already a 0 on this board for the same bus
#define VIK_SPI_REG_START 1
// Need to specify the cs-gpios for this bus, as the same bus is used for VIK, so any SPI
// VIK modules will need to know the existing CS gpios so it can append to them
#define VIK_SPI_CS_PREFIX <&gpio0 21 GPIO_ACTIVE_LOW>

// If this ever gets updated to a different SPI bus, update the corresponding "vik_spi: &spi0 {};"
// in 3_0_0/vik_connector.dtsi and 3_1_0/vik_connector.dtsi
&spi0 {
    status = "okay";
    cs-gpios = VIK_SPI_CS_PREFIX;
    pinctrl-0 = <&spi0_default>;
    pinctrl-names = "default";
    clock-frequency = <DT_FREQ_M(2)>;

    shift_reg: 595@0 {
        compatible = "zmk,gpio-595";
        status = "okay";
        gpio-controller;
        spi-max-frequency = <200000>;
        reg = <0>;
        #gpio-cells = <2>;
        ngpios = <8>;
    };
};
```