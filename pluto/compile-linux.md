---
title: Compiling on Linux
layout: home
parent: Pluto
nav_order: 2
---

The build system has the following package requirements:
- python3 >= 3.6
- libsdl2-dev
- libglew-dev
- git
- libcurl
- zlib

### Ubuntu
```
sudo apt install build-essential git python3 libglew-dev libsdl2-dev libz-dev libcurl4-openssl-dev libcjson-dev xclip
```
### Arch
```
sudo pacman -S base-devel python sdl2-compat glew zlib-devel libcurl-devel cjson xclip
```
### Fedora
```
sudo dnf install make gcc python3 glew-devel SDL2-devel zlib-devel libcurl-devel cjson xclip
```
***

Clone the source code:
```
git clone https://github.com/Llennpie/Pluto
```

Drop an **unmodified, vanilla USA ROM** into `Pluto/` and rename it to `baserom.us.z64`. Then enter the directory and compile:
```
cd Pluto
make
```

An executable will be created in `Pluto/build/us_pc/`.
