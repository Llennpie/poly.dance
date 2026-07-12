---
title: Compiling on Windows
layout: home
parent: Pluto
nav_order: 1
---

**Pluto** does not contain a built-in asset extractor and must be **compiled from source code**, like many other SM64 PC ports.

You will need to obtain an **unmodified, vanilla US ROM** of *Super Mario 64*.

- TOC
{:toc}

***

Download and install the latest version of [MSYS2](https://www.msys2.org/).

Launch the `MINGW64` executable (`MINGW32`/32-bit is untested). *If you can't find it, search `MINGW` in the start menu
and select the app with the blue icon.*

Enter `pacman -Syuu` in the prompt and hit enter. Press `Y` whenever it asks you to update packages.

Next run this command in the terminal to install our necessary packages:
```
pacman -S unzip make git mingw-w64-x86_64-gcc mingw-w64-x86_64-glew mingw-w64-x86_64-SDL2 mingw-w64-x86_64-cjson python3
```

Clone the source code and enter its folder:
```
git clone https://github.com/Llennpie/Pluto && cd Pluto
```

Next, run `explorer .` to open a new File Explorer window at Pluto's root directory. Drop your ROM in here and rename
it to `baserom.us.z64`.

Begin compiling:
```
make
```
*(You can optionally run `make -j` for a faster compile but at the cost of CPU usage.)*

When completed, an executable will be created in `Pluto\build\us_pc\`.
