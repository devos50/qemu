## Running the iPod Touch 2G Emulator

This file contains the instructions on how to run the iPod Touch 2G emulator using QEMU.
Note that this is an experimental release and the functionality of the device is still limited.
Linux compatibility is currently unstable. Efforts are underway to improve it.

### Getting the Source Code

Clone the repository using the following commands:

```
git clone https://github.com/devos50/qemu-ios.git
cd qemu-ios
```

Now you can proceed with the rest of the instructions for building and running the emulator.

### Building QEMU

Make sure you have the required libraries installed to compile QEMU:

```
# On MacOS
brew install glib ninja pixman pkg-config sdl2

# On Linux (Ubuntu)
sudo apt install make ninja-build pkg-config libssl-dev libsdl2-dev libpixman-1-dev libpixman-1-0 libglib2.0-dev

# On Windows (MSYS2/mingw64)
pacman -S base-devel mingw-w64-x86_64-toolchain git python ninja mingw-w64-x86_64-glib2 mingw-w64-x86_64-pixman python-setuptools mingw-w64-x86_64-SDL2

```
Compile QEMU by running the following commands from the root directory:

```
mkdir build
cd build
# On Intel Macs
./configure --target-list=arm-softmmu --extra-cflags=-I/usr/local/opt/openssl@3/include --extra-ldflags='-L/usr/local/opt/openssl@3/lib -lcrypto'

# On Apple Silicon Macs
./configure --enable-sdl --target-list=arm-softmmu --disable-capstone --disable-pie --disable-slirp --extra-cflags=-I/opt/homebrew/opt/openssl@3/include --extra-ldflags='-L/opt/homebrew/opt/openssl@3/lib -lcrypto'

# On Linux (Ubuntu)
./configure --enable-sdl --disable-cocoa --target-list=arm-softmmu --disable-capstone --disable-slirp --extra-cflags=-I/usr/include/openssl --extra-ldflags='-lcrypto' --disable-werror --enable-pie

# On Microsoft Windows (with MINGW64)
./configure --enable-sdl --disable-cocoa --target-list=arm-softmmu --disable-capstone --disable-slirp --disable-pie --extra-cflags=-I/mingw64/include/openssl --extra-ldflags='-L/mingw64/lib -lcrypto' --disable-stack-protector --disable-werror

make
```

Note that we’re explicitly enabling compilation of the SDL library which is used for interaction with the emulator (e.g., capturing keyboard and mouse events). Also, we only configure and build the ARM emulator.
We are also linking against OpenSSL as the AES/SHA1/PKE engines use some of the library’s cryptographic functions.
Remember to update the include and library paths to the OpenSSL library in case they are located elsewhere.
You can speed up the `make` command by passing the number of available CPU cores with the `-j` flag, e.g., use `make -j6` to compile using six CPU cores.
The compilation process should produce the `qemu-system-arm` binary in the `build/arm-softmmu` directory.

### Downloading the Required Files

We need a few files to successfully boot the iPod Touch emulator to the home screen, which I published [here](https://github.com/devos50/qemu-ios/releases/tag/n72ap_v1) for convenience. You can download all these files from here, and they include the following:
- The S5L8720 bootrom binary, which is started as the very first thing when booting.
- A NOR image that contains various auxillary files used by the bootloader. I provide some instructions on how to generate this image yourself [here](https://github.com/devos50/qemu-ios-generate-nor).
- A NAND image that contains the root file system. I provide some instructions on how to generate this image yourself [here](https://github.com/devos50/qemu-ios-generate-nand).

Download all the required files and save them to a convenient location. You should unzip the `nand_n72ap.zip` file, which contains a single directory named `nand`.

### Running the Emulator

We are now ready to run the emulator from the build directory with the following command:

```
./arm-softmmu/qemu-system-arm -M iPod-Touch,bootrom=<path to bootrom>,nand=<path to NAND directory>,nor=<path to NOR directory> -serial mon:stdio -cpu max -m 2G -d unimp
```

If there are any issues running the above commands, please let me know by [opening an issue](https://github.com/devos50/qemu-ios/issues/new).
