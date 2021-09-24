
# Table of Content
- [Table of Content](#table-of-content)
- [Compiler](#compiler)
- [Setting up OpenOCD in a project](#setting-up-openocd-in-a-project)
  - [Creating the configuration file](#creating-the-configuration-file)
  - [Setting Clion up to run our program](#setting-clion-up-to-run-our-program)


# Compiler
It goes without saying that to run code on an STM32, you need a compiler capable of building code for ARM.

Since Clion does not come with a built-in compiler, you need to provide our own. This means that every compilers are supported, even MSVC if you are a fanboy!

**However, it is extremely important to note that when working on embedded projects using Windows, Clion does not support Cygwin, it only supports Mingw!!!**

# Setting up OpenOCD in a project
The default board configurations provided by OpenOCD may not be exactly what we need in a project that uses a custom board design. For this, we need to create a custom configuration file that integrates both the target and interface informations.

Normally, when invoking OpenOCD, you need to specify both the target you are using and the interface used to connect and communicate with that target, for example, if the target is a STM32F405RGT6, and the interface is a st-link v2, the invoking command would be `openocd.exe -f interfaces/stlink.cfg -f target/stm32f4xx.cfg`.

In Clion however, we can only specify a single configuration file. This means we need to make a configuration for our needs, based on the existing configuration files.

## Creating the configuration file
Creating a configuration file for any combination of target/interface is simple and straight forward. 
In this example, we will use a STM32F405RGT6 as our target and a ST-Link v2 as our interface.

In a file, first add your interface, followed by your target, as shown bellow:

```
source [find interface/stlink.cfg]
source [find target/stm32f4xx.cfg]
```

And that's it! You can find all targets and interfaces supported by OpenOCD by browsing the configuration files provided by the distribution, which can be found in `share/openocd/scripts`.
Furthermore, other parameters can be added to our new configuration file, such as transport layer, connection speed, etc. Please refer to [section 6 of OpenOCD's manual](https://openocd.org/doc/pdf/openocd.pdf) for more informations.

## Setting Clion up to run our program
For Clion to be able to run our program on our board, we must first have a configuration to tell it how it should do it, what file to use, how to connect to the board, and other important things to configure.
The target, executable and GDB fields should already be filed. If they aren't:
- `target`: The `elf` file to use
- `executable`: Same as `target`
- `GDB`: Path of GDB. Of course, since we are using an STM32, we need a build of GDB for ARM.
Next, specify the [board configuration file](#creating-the-configuration-file) that we just created.

You can then run and debug your program as you wish!