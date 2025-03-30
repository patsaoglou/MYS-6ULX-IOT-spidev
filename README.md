# MYS-6ULX-IOT modified device trees
In this repository I share modified device trees from the preconfigured Linux source provided by MYIR Tech for the MYS-6ULX-IOT iMX6ULL based board to enable the SPI bus in order to use userspace SPIDEV under /dev/spidev0.*. I also disable some unnecessary stuff i don't use.

## Prerequisites
1. **Preconfigured Linux source:** available SDK provided by MYIR Tech under https://d.myirtech.com/MYS-6ULX/ - Sources
2. **SPI related linux drivers:** spi plattform related drivers for the IMX6 soc and spidev related stuff should be enabled within menuconfig and compiled as a module or built-in

## Compiling DTS
1. Navigate to the linux source root directory
2. Under the terminal run to preprocess the DTS: 
```shell
    cpp -nostdinc -I include -I arch  -undef -x assembler-with-cpp arch/arm/boot/dts mys-6ull-14x14-gpmi-weim.dts mys-6ull-14x14-gpmi-weim.dts.preprocessed
```
3. Run to get the device tree blob: 
```shell
    dtc -I dts -O dtb -p 0x1000 mys-6ull-14x14-gpmi-weim.dts.preprocessed -o mys-6ull-14x14-gpmi-weim-modified.dtb
```

## Updating DTB under U-BOOT
1. Copy the modified DTB into the boot partition (in my config I boot from sdcard and the bootloader is running from the nand flash)
2. Set DTB environment variable, base address and load it to the bootloader: 
```u-boot
    setenv fdt_file mys-6ull-14x14-gpmi-weim-modified.dtb
    setenv fdt_addr_r 0x83000000     
    load mmc 0:1 ${fdt_addr_r} ${fdt_file}
```
3. If you don't want to change anything else save u-boot configuration and you can boot: 
```u-boot
    saveenv
    boot
```
4. Complete reconfiguration of the bootloader: 
```u-boot
    setenv fdt_file mys-6ull-14x14-gpmi-weim-modified.dtb 
    setenv fdt_addr_r 0x83000000     
    load mmc 0:1 ${fdt_addr_r} ${fdt_file}


    load mmc 0:1 ${loadaddr} zImage

    bootz ${loadaddr} - ${fdt_addr_r}

    setenv bootcmd 'load mmc 0:1 ${loadaddr} zImage; load mmc 0:1 ${fdt_addr_r} ${fdt_file}; bootz ${loadaddr} - ${fdt_addr_r}'

    saveenv
```

## Enabling SPI bus
- Setting 2 chipselect pins
- compatible is set to "rohm,dh2228fv" instead of "spidev" so there is no stack trace error at boot (dmesg)
- pin cs set to active logic 0 - as typical too
```dts
    &ecspi1 {
        fsl,spi-num-chipselects = <2>;
        cs-gpios = <&gpio4 26 0>, <&gpio4 24 0>; 
        pinctrl-names = "default";
        pinctrl-0 = <&pinctrl_ecspi1 &pinctrl_ecspi1_cs>;
        status = "okay";

        spidev@0 {
            compatible = "rohm,dh2228fv";
            spi-max-frequency = <8000000>;
            reg = <0>;
        };

        spidev@1 {
            compatible = "rohm,dh2228fv";
            spi-max-frequency = <8000000>;
            reg = <1>;
        };
    };
```

## Assigning pins
```dts
    pinctrl_ecspi1: ecspi1grp {
		fsl,pins = <
			MX6UL_PAD_CSI_DATA07__ECSPI1_MISO 0x100b1
			MX6UL_PAD_CSI_DATA06__ECSPI1_MOSI 0x100b1
			MX6UL_PAD_CSI_DATA04__ECSPI1_SCLK 0x100b1
		>;
	};
	
	pinctrl_ecspi1_cs: ecspi1cs {
		fsl,pins = <
			MX6UL_PAD_CSI_DATA05__GPIO4_IO26 0x80000000 //CS0
			MX6UL_PAD_CSI_DATA03__GPIO4_IO24 0x80000000 //CS1
		>;
	};
```

## Resources
Resolve stack trace: https://yurovsky.github.io/2016/10/07/spidev-linux-devices.html
Resolve dts compilation error with typical command: https://stackoverflow.com/questions/50658326/device-tree-compiler-not-recognizes-c-syntax-for-include-files 
## Disclaimer
All other necessary files for the board were supplied by MYIR Tech SDK and I share the 
modification to be available as a resource. Wish i had that when i did it.

Use it at your own responsibility.

If any mistakes let me know.


