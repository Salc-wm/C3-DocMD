# Installing C3C
You can use one of the precompiled binaries or build the compiler from source.

## Installing on Windows
1. Download the zip file: _[stable](https://github.com/c3lang/c3c/releases/download/v0.6.1/c3-windows.zip)_ _[debug](https://github.com/c3lang/c3c/releases/download/v0.6.1/c3-windows-debug.zip)_
2. Unzip it into a folder
3. If you don’t have Visual Studio 17 installed you can either install it or follow the next two steps.
4. Run the `msvc_build_libraries.py` Python script which will download the necessary files to compile on Windows.

### optional: set c3c as a global enviornment variable

5. copy the folder
6. navigate to `C:\Program Files`
7. paste the folder here
8. navigate inside the folder you’ve pasted
9. copy the path of the folder
10. search for “edit the system environment variables” on your computer
11. click on the “environment variables” button on the bottom right
12. under “user variables” double click on “path”
13. click on “new” and paste the path to the folder
14. run `c3c` on anywhere in your computer!

```bash
c3c compile ./hello.c3
```

# Installing on Debian
1. Download tar file: https://github.com/c3lang/c3c/releases/download/latest/c3-linux.tar.gz (debug _version [here](https://github.com/c3lang/c3c/releases/download/v0.6.1/c3-linux-debug.tar.gz)_)
2. Unpack executable and standard lib.
3. Run `./c3c.`

# Installing on Mac
1. Make sure you have XCode with command line tools installed.
2. Download the zip file: https://github.com/c3lang/c3c/releases/download/latest/c3-macos.zip (debug _version [here](https://github.com/c3lang/c3c/releases/download/v0.6.1/c3-macos-debug.zip)_)
3. Unzip executable and standard lib.
4. Run `./c3c.`

# Installing on Arch Linux
There is an AUR package for the c3c compiler : __[c3c-git](https://aur.archlinux.org/packages/c3c-git)__.

Due to some issues with the LLVM packaged for Arch Linux, the AUR package will download and use LLVM 16 for Ubuntu-23.04 to compile the c3c compiler.

You can use your AUR package manager:
```bash
paru -S c3c-git
# or yay -S c3c-git
# or aura -A c3c-git
```

Or clone it manually:
```bash
git clone https://aur.archlinux.org/c3c-git.git
cd c3c-git
makepkg -si
```

# Building C3
C3 has precompiled binaries for Windows, MacOS and Ubuntu. For other platforms it should be possible to compile it on any platform LLVM can compile to. You will need CMake installed.

### 1. Install LLVM
See _LLVM_ the [LLVM documentation](https://llvm.org/docs/GettingStarted.html) on how to set up LLVM for development. On MacOS, installing through Homebrew works fine. Using apt-get on Linux should work fine as well. For Windows you can download suitable pre-compiled LLVM binaries from https://github.com/c3lang/win-llvm

### 2. Clone the C3 compiler source code from Github
This should be as simple as doing:
```bash
git clone https://github.com/c3lang/c3c.git
```
… from the command line.

### 3. Build the compiler
Create the build directory:
```bash
MyMachine:c3c$ mkdir build
MyMachine:c3c$ cd build/
```

Use CMake to set up:
```bash
MyMachine:c3c/build$ cmake ../
```

Build the compiler:
```bash
MyMachine:c3c/build$ make
```

### 4. Test it out
```bash
MyMachine:c3c/build$ ./c3c compile-run ../resources/testfragments/helloworld.c3
```

## Building via Docker
You can build `c3c` using either an Ubuntu 18.04 or 20.04 container:
```bash
./build-with-docker.sh 18
```
Replace 18 with 20 to build through Ubuntu 20.04.

For a release build specify:
```bash
./build-with-docker.sh 20 Release
```

A `c3c` executable will be found under `bin/`.

# Building on OS X using Homebrew
1. Install CMake: `brew install cmake`
2. Install LLVM 17+: `brew install llvm`
3. Clone the C3C github repository: `git clone https://github.com/c3lang/c3c.git`
4. Enter the C3C directory `cd c3c`.
5. Create a build directory `mkdir build`
6. Change directory to the build directory `cd build`
7. Set up CMake build for debug: `cmake ..`
8. Build: `cmake --build .`
