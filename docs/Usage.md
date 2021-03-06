# Environment 
ESESC runs on Linux.  It has been tested on x86-64 and ARM platforms. The x86-64 system is the default, but ESESC also compiles and runs on ARM. ESESC itself executes native ARM binaries using QEMU as an emulator.

# Requirements 
ESESC is currently tested with Arch Linux and with Ubunut 12.04 LTS.  The following commands list the packages and configuration settings required for each tested OS.  Other Linux version should work as well, but the list of packages will need to be adapted.

## Arch Linux: (64bit)

    pacman -S boost
    pacman -S bison flex
    pacman -S gcc
    pacman -S python2
    pacman -S texinfo
    pacman -S cmake
    pacman -S make
    pacman -S pkgconfig

## Ubuntu 12.04 LTS (64bit)
    sudo apt-get install build-essential
    sudo apt-get install cmake
    sudo apt-get install libboost-dev
    sudo apt-get install bison flex
    sudo apt-get install g++
    sudo apt-get install python
    sudo apt-get install texinfo
    sudo apt-get install libglib2.0-dev
    sudo apt-get install libncurses5-dev

With Ubuntu 12.04 it might also be necessary to set the `mmap_min_addr` setting.
This can be done by editing `/etc/sysctl.conf` and adding the line:

    vm.mmap_min_addr = 4096

To apply this change to the current session run the following command as root:

    echo 4096 > /proc/sys/vm/mmap_min_addr

# Steps to compile and build eSESC

# Compile ESESC to run on an x86_64 system:

These steps assume that the ESESC repo is checked out in the `~/projs/esesc` directory. And also that you are building in the `~/build/debug` or `~/build/release` directory. Change them for your own configuration as needed.

## Debug
    mkdir -p ~/build/debug
    cd ~/build/debug
    cmake -DCMAKE_BUILD_TYPE=Debug ~/projs/esesc
    make VERBOSE=1

# Release
    mkdir ~/build_release
    cd ~/build_release
    cmake ~/projs/esesc -DESESC_QEMU_ISA=armel
    make 

# Compile with QEMU in a arm machine (arch linux chromebook)

cmake -DCMAKE_BUILD_TYPE=Debug -DCMAKE_HOST_ARCH=armel -DESESC_QEMU_ISA=armel ~/projs/esesc

# Compile ESESC with DEBUG and clang compiler

CC=clang CXX=clang++ cmake -DCMAKE_BUILD_TYPE=Debug ~/projs/esesc

# Compile ESESC with DEBUG and emscripten compiler

CC=emcc CXX=em++ cmake -D_CMAKE_TOOLCHAIN_PREFIX=em -DCMAKE_BUILD_TYPE=Debug ~/projs/esesc

# Compile scmain (SCCORE) 

cmake ~/projs/esesc/ -DENABLE_SCQEMU=1 -DESESC_QEMU_ISA=armel -DCMAKE_BUILD_TYPE=Debug

--------------------------------------------------------
# Compile scmain in an arm machine (arch linux chromebook)

cmake ~/projs/esesc/ -DENABLE_SCQEMU=1 -DCMAKE_HOST_ARCH=armel -DESESC_QEMU_ISA=armel -DCMAKE_BUILD_TYPE=Debug


--------------------------------------------------------
## Sample execution for ARM + CUDA

* Enable CUDA with ARM for QEMU (experimental)
* NOTE: THIS DOES NOT WORK WITH ANY OTHER TARGET (i.e. SPARC, MIPS, etc. ) 
* NOTE: This works only if qemu is compiled for a 32 bit host.  

# Required : 
1. CUDA enabled device 
2. CUDA version 3.2 (Check the version you have by typing "nvcc --version")
3. Compatibility Libraries (for 32 bit development) on the host machine.

Compile QEMU and generate the  "qemu-arm" binary
    ./configure --target-list=arm-linux-user --cpu=i386 --disable-vnc-sasl --disable-vnc-tls --enable-esesc_cuda --enable-esesc_user --disable-uuid --disable-sdl  --interp-prefix=/mada/software/benchmarks/armel-root  --extra-cflags="-Wno-unused-but-set-variable" --enable-debug --disable-guest-base
    gmake 

Run a test application "simple-kernel-arm" which takes an array of 10 floats and squares them on the GPU. and returns the value back to the host
    cp ~/projs/esesc/conf/simple-kernel-arm ~/projs/esesc/emul/qemu/arm-linux-user/.
    cd ~/projs/esesc/emul/qemu/arm-linux-user/
    ./qemu-arm simple-kernel-arm

If you see the following error message 
    "cannot restore segment prot after reloc: Permission denied"
 you will need to disable SELINUX. 

To Temporarily disable enforcement on a running system
(requires su access)
    /usr/sbin/setenforce 0

To permanently disable enforcement during a system startup
change "enforcing" to "disabled" in ''/etc/selinux/config'' and reboot.

--------------------------------------------------------


Enable CUDA with ARM, Run a sample benchmark with eSESC (currently without traces)
* NOTE: THIS DOES NOT WORK WITH ANY OTHER TARGET (i.e. SPARC, MIPS, etc. ) 
* NOTE: This works only if qemu is compiled for a 32 bit host.  
* NOTE: eSESC will be compiled in a 32 bit mode. 
* NOTE: Not tested Linux binfmt mode.

1. Compile CUDA with ARM for QEMU
    cd <ESESC_DIR>/emul/qemu
    ./configure --cpu=i386 --target-list=arm-linux-user --disable-vnc-sasl --disable-vnc-tls --enable-esesc_cuda --enable-esesc_user --disable-uuid --disable-sdl --disable-guest-base

2. Run cmake to generate the makefiles in the build directory with the flags set
    cd <BUILD_DIR>
    cmake -DCMAKE_BUILD_TYPE=Release -DESESC_QEMU_ISA=armel -DCMAKE_HOST_ARCH=i386 -DENABLE_CUDA=1 <ESESC_DIR>
    cmake -DCMAKE_BUILD_TYPE=Debug -DESESC_QEMU_ISA=armel -DCMAKE_HOST_ARCH=i386 -DENABLE_CUDA=1 <ESESC_DIR>

    gmake
3. Copy the configuration files to the folder. Change the "benchName" to point to a CUDA application compiled for ARM. (Specify the path of the binary relative to the conf file)
    e.g. benchName  = "./exe/matrixMul-CUDA3.0-arm"
4. Run qemumain from the directory where the conf files are located. 
    <ESESC_DIR>/emul/libqemuint/qemumain

    This simply calls the CUDA program through qemumain, None of the traces are passed onto the eSESC yet. 
       
--------------------------------------------------------

eSESC generates a dot file named memory-arch.dot, which is a description of the memory hierarchy as eSesc sees, after parsing shared.conf.
To view this in as a png, run the following command. 

    dot memory-arch.dot -Tpng -o memory-arch.png

--------------------------------------------------------
#Power

To enable power, set `enablePower = true` in `esesc.conf`

#Temperature

To enable temperature modeling, both the following should be set: 
    enablePower = true 
    doTherm = true  # in therm.conf

Temperature modeling needs a floorplan as well. There are several floorplan for
different configurations already in flp.conf. To use a floorplan, update
the corresponding parameters in therm.conf. For example:

    floorplan[0] = 'floorplan_4c'
    layoutDescr[0] = 'layoutDescr_4c'

both floorplan_4c and layoutDescr_4c are sections defined in flp.conf.
Each floorplan is usually specified with a name mangle, _4c in this example.


To generate a new floorplan, use floorplan.rb script:

    esesc/conf/floorplan.rb BuildDirPath SrcDirPath RunDirPath nameMingle

BuildDirPath: Path to the esesc build directory
SrcDirPath: Path to the esesc source directory
RunPath: Path to the directory that configuration files reside for run. This is the same as build directory if you are running esesc from build directory.
nameMangle: A string to attach to `floorplan` and `layoutDescr` sections when saved in `flp.conf`.

This will generate the floorplan information and power breakdown of blocks,
and run the floorplanning tool to generate a new floorplan, save in in `flp.conf`, and 
update `therm.conf` to point to it.

