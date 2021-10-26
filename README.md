
# Table of Content
- [Table of Content](#table-of-content)
- [Compiler](#compiler)
  - [Finding a compiler for STM32 on Windows](#finding-a-compiler-for-stm32-on-windows)
  - [Configuring the toolchain in Clion](#configuring-the-toolchain-in-clion)
- [Setting up OpenOCD in a project](#setting-up-openocd-in-a-project)
  - [Creating the configuration file](#creating-the-configuration-file)
  - [Setting Clion up to run our program](#setting-clion-up-to-run-our-program)
- [Importing a project from STM32CubeIDE](#importing-a-project-from-stm32cubeide)
- [Adding NilaiTFO to a project](#adding-nilaitfo-to-a-project)


# Compiler
It goes without saying that to run code on an STM32, you need a compiler capable of building code for ARM.

Since Clion does not come with a built-in compiler, you need to provide our own.

**However, it is extremely important to note that when working on embedded projects using Windows, Clion does not support Cygwin, it only supports Mingw!!!**

## Finding a compiler for STM32 on Windows
Since we must use GCC for ARM in Clion, we need to find a version of GCC that does that, which is an adventure on Windows. We will thus use a shortcut and "borrow" STM32CubeIDE's GCC.

- Navigate to the installation folder of STM32CubeIDE. Mine is located at `C:/ST/STM32CubeIDE_1.0.2/STM32CubeIDE`
- In the `plugins` directory, find the `com.st.stm32cube.ide.mcu.externaltools.gnu-tools-for-stm32.7-2018-q2-uipdate.win32_XXXXX` directory. You may have it multiple times with multiple different version, just open the most recent one.
- Copy the `tools` folder located within it somewhere else on your computer, feel free to rename it
- Add the directory to your system's PATH, Clion needs it to find GCC.
- There you go! GCC compiler supporting C++17 on Windows with no trouble at all! :) 

## Configuring the toolchain in Clion
For Clion to be able to compile a STM32 project, we need to specify a toolchain that is able to generate code for ARM processors.

- Open up Clion's settings (File -> Settings)
- Under `Build, Execution, Deployment`, select `Toolchains`
- Click the `+` icon to create a new toolchain configuration and select your favorite environment.
    **NOTE:** *On Windows, only MinGW is supported for embedded projects!*
- If desired, set this toolchain as the default one.
- Let CMake detect the compilers, but specify the path to your GDB for ARM

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

# Importing a project from STM32CubeIDE
**It is highly recommended to migrate to Clion into a new branch, to avoid risks of breaking the repository!**

To import a project from STM32CubeIDE into Clion, you must first modify the `.ioc` file so that it generates the project files in a format that Clion understands.
To do that:
  - Create a copy of the `.ioc` file
  - In Clion, create a new STM32 project (File -> New Project -> STM32CubeMX), at the location of your project
  - A prompt will appear saying that the directory is not empty. Select `Create from Existing Sources`.
  - Clion will now have create a new `.ioc` file bearing the name of the project. Delete this file, and rename your copy of the `.ioc` to its original name

# Adding NilaiTFO to a project
- Clone [NilaiTFO](https://github.com/smartel99/NilaiTFO) into your project
- In CMakeLists_template.txt (**Not CMakeLists.txt**), add the following:
```
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -include [path/to/NilaiTFOConfig.h] -include [path/to/nilai/defines/compilerDefines.h]")
include_directories(${includes} [root/of/nilai])
file(GLOB_RECURSE SOURCES ${sources} "[path/to/nilai]/*.*")
```