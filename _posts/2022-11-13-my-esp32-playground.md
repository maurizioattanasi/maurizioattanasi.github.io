---
layout: post
title:  "My ESP32 Playground"
tags:
-  ESP32
-  Expressif
-  IDF
-  SoC
-  Proogramming
author: Maurizio Attanasi
---

After a long period of inactivity, I decided to pull two microcontrollers from the ESP32 family out of the drawer and play with them in my spare time.

The first (and the oldest one) is an ESP-WROOM-32 development board.

<p align='center'>
  <img src='/assets/images/2022-11-13-my-esp32-playground/esp-tree.png' alt='My ESP32s' style="max-width:100%">
</p>

<hr>

The other one is an ESP32-CAM.

There are a few ways to write code runnable on an ESP3. 

one based on the Python programming language

- [Micropython]The other two are C/C++ based:

- [Arduino ESP32](https://docs.espressif.com/projects/arduino-esp32/en/latest/getting_started.html);
- [Espressif IoT Development platform](https://idf.espressif.com/) also known as IDF;

My first device programming experiments will be with the latter of the two, IDF, firstly because it's the one with its implementation of [FreeRTOS](https://www.freertos.org/).

<br>

<p align='center'>
 <img src='/assets/images/2022-11-13-my-esp32-playground/logo-freertos.jpeg' alt='FreeRTOS' style="max-width:100%">
</p>

## Environment setup

### IDF IoT Development Framework

Following the guidelines available in the [official](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/get-started/index.html#installation-step-by-step) documentation](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/get-started/index.html#installation-step-by-step), it's easy enough to set up a ready-to-use development framework for any major operating system type.

After the setup process, something like that should appear on the file system.

<p align='center'>
 <img src='/assets/images/2022-11-13-my-esp32-playground/esp-tree.png' alt='folder-structure' style="max-width:100%">
</p>

### Code editor

Now that the environment is ready, all we need is a text editor to write our code. There are many choices [Notepadd++](https://notepad-plus-plus.org/) on Windows, [Sublime Text](https://www.sublimetext.com/) on macOS, [Vim](https://www.vim.org/) on Unix.
My choice is [Visual Studio Code](https://code.visualstudio.com/) on every platform.

### Terminal

We have the environment, and the code editor last, but not least, we need a terminal app to build, flash and monitor our applications. Every OS has a default one, Windows has **Windows Console** app, and [Windows Teminal](https://apps.microsoft.com/store/detail/windows-terminal/9N0DX20HK701?hl=it-it&gl=it), macOS has the default Terminal app, and every distribution of Unix has its own. My favourites are:

- [iTerm2](https://iterm2.com/) combined with [Z shell](https://zsh.sourceforge.io/) on macOs;
- [Windows Teminal](https://apps.microsoft.com/store/detail/windows-terminal/9N0DX20HK701?hl=it-it&gl=it) with [Powershell 7](https://github.com/PowerShell/PowerShell) shell;
- The default [GNOME TErminal](https://github.com/GNOME/gnome-terminal) on Ubuntu or Debian.
- The Visual Studio Code integrated terminal;

## First Project

Following the official documentation, we can simply copy one of the sample projects we downloaded in the esp installation folder but, after all, we only need to configure a simple [CMake](https://cmake.org/) project with the following folder structure.


```bash
└── 01.project-template
    ├── CMakeLists.txt
    └── main
        ├── CMakeLists.txt        
        ├── Makefile        
        └── main.cpp
```

In the root folder of the project, we have the first _CMakeLists.txt_ configuration project containing a set of directives and instructions describing the project's source files and targets (executable, library, or both).

```cpp
cmake_minimum_required(VERSION 3.5)

include($ENV{IDF_PATH}/tools/cmake/project.cmake)
project(esp32-playground)
```

In the folder *main* we'll find another _CMakeLists.txt_ configuration with the following content:

```cpp
idf_component_register(SRCS "main.cpp"
                       INCLUDE_DIRS ".")
```

a _Makefile_ 

```cpp
#
# This is a project Makefile. It is assumed the directory this Makefile resides in is a
# project subdirectory.
#

PROJECT_NAME := esp32-playground

include $(IDF_PATH)/make/project.mk
```

and, finally, the _main.cpp_ source file

```cpp
#include <iostream>

extern "C" void app_main(void)
{
    std::cout << "Have fun!" << std::endl;
}
```

### IDF Configuration

We're almost there. The last step to build and flash our first app is to open a shell and set up the IDF's environment variables with the following command:

```cmd
~ . $HOME/esp/esp-idf/export.sh
```

then we need to tell IDF the target platform,

```cmd
~ cd /src/01.hello-world
~ idf.py set-target esp32
```

build the project,

```cmd
~ idf.py build
```

flash it on the controller

```cmd
~ idf -p <port> flash
```

in my case and, finally, open a serial connection and check the result:

```cmd
~ idf -p /dev/cu.usbserial-0001 monitor
```

<p align='center'>
 <img src='/assets/images/2022-11-13-my-esp32-playground/hello-world.jpg' alt='Hello World' style="max-width:100%">
</p>

That's it. Enjoy :up: