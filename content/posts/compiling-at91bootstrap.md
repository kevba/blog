# Compiling At91Bootstrap for the Aria G25

## Some context

This device is build using an Aria G25 from corewind. Corewind themselves does offer prebuild files, including at91bootstrap, but those are quite old.

## Why would you do this in the first place?

While trying to update Linux kernel I discovered that u-boot had to be updated as well, since the gcc version has updated. Updating u-boot mde the u-boot binary just slighty larger. This caused a problem with at91bootstrap. at91bootstrap loads bootstrap into memory, so it needs to know how large u-boot is. So a new version of at91bootstrap must be compiled with an increaded size for u-boot.

## Neccesary setup 

The buildroot used can be found [here][0]. This post assumes you have some idea of how this setup works. Still, the commands needed to setup the various containers are described below.

You need a container to store the build results and build cache. It can be created like this:
```
docker run -i --name buildroot_output advancedclimatesystems/buildroot /bin/ echo "Data only."
```

it is also assumed the latest image is build locally with the following command. 
```
docker build -t "advancedclimatesystems/buildroot" .
```
Now thats out of the way, let build a bootloader.

## Menuconfig configuration

To see all existing configs run:

```
./scripts/run.sh make list-defconfigs
```

Since I am using the Aria G25, I am using the `acmesystems_aria_g25_256mb_defconfig` defconfig. 

First lauch menuconfig using the docker container. This an be done something like this:

```
./scripts/run.sh make acmesystems_aria_g25_256mb_defconfig menuconfig
```

This will open builtroots menuconfig. Open the bootloaders menu, from here the bootloaders can be configured. 
The default configuration uses AT91bootstrap3, which is exactly what we need. U-Boot is disabeld however. Since we want to boot to U-boot from at91bootstrap is should be enabled.

## at91bootstrap configuration
at91bootstrap has its own menuconfig. It can be opened using the following command:
```
./scripts/run.sh make at91bootstrap3-menuconfig
``` 

By default at91bootsstrap is configured to start for an SDcard. I want to use the NAND on the aria G25. This menas we need to configure that as well.

Navigate to the `memory selection` submenu. Change the `Flash Memory Technology` to `NAND flash`

To load U-boot into RAM the `Image Loading Strategy` must be changed to   `Load U-Boot into last MBYTE of SDRAM`

Thansk to our previous options, some new options have become available in the `U-Boot Image Storage Setup` menu. These options are needed to load u-boot into memory. The first option is ` Flash Offset for U-Boot`. It is the address on the NAND where u-boot iamge begins. The second option called `U-Boot Image Size` is teh size of the u-boot iamge on the NAND. These two options can difffer per board, or even per u-boot build. In my case, u-boot is stored at `0x00040000`, which is the same as the default.
My u-boot iamge is slightly largerm because I am using a custom build. So the  `U-Boot Image Size` has to be increased. In my case it needed to be set to  `0x00100000`. 

The last option is called `The External Ram Address to Load U-Boot Image`. It tells at91bootstrap to which address in RAM< the u-boot iamge must be loaded. the default option of `0x22000000` is fine.


## Compiling!
Now we are ready to compile! to compile just at91bootstrap run the follwoing command:
```
./scripts/run.sh make at91bootstrap3-build
``` 
Go grab a beverage of your choosing, bacause compiling might take a while. 




[0]: https://github.com/AdvancedClimateSystems/docker-buildroot