# Start Up

In this tutorial, we will provide access to cloud servers, prepare the XiangShan development environment for you, and go through the development workflows incluing simulation, function verification and performance verification. All you need is a computer with an Internet connection and `ssh` tools to do this!

## Demo Instructions

Shell commands are presented in boxes.

```shell
echo "Hello, XiangShan"
echo "Have a nice day" 
```

Description and notes are presented in boxes with prefix `#`.

```
# Please prepare a laptop with an SSH client.
# Next, let's start the demo session！
```

## Prerequisites

Login to the provided cloud server.

**The server is only available during the tutorial.**

For Windows (Windows Terminal / PowerShell is recommended):

```powershell
ssh guest@tutorial.xiangshan.cc
# Password: xiangshan-2025
```

For Linux / Mac:

```shell
ssh guest@tutorial.xiangshan.cc
# Password: xiangshan-2025
pwd
# /home/guest

```

For offline users, please refer to [https://github.com/OpenXiangShan/xs-env/tree/tutorial-new](https://github.com/OpenXiangShan/xs-env/tree/tutorial-new)

```shell
# Copy tutorial environment to your dir based on your name
cp -r /opt/xs-env ~/<YOUR_NAME>

# Enter your dir
cd ~/<YOUR_NAME>

# Set up environment variables
# DO IT AGAIN when opening a new terminal
source env.sh

# SET XS_PROJECT_ROOT:                  /home/guest/YOUR_NAME
# SET NOOP_HOME (XiangShan RTL Home):   $XS_PROJECT_ROOT/XiangShan 
# SET NEMU_HOME:                        $XS_PROJECT_ROOT/NEMU 
# SET AM_HOME:                          $XS_PROJECT_ROOT/nexus-am
# SET TLT_HOME:                         $XS_PROJECT_ROOT/tl-test-new
# SET gem5_home:                        $XS_PROJECT_ROOT/gem5
```

```shell
# Project Structure
tree –d –L 1
# .  
# ├── DRAMsim3
# ├── gem5
# ├── NEMU 
# ├── nexus-am 
# ├── NutShell
# ├── tl-test-new
# ├── tutorial 
# └── XiangShan
  
# Enter XiangShan directory
cd XiangShan
```

## Chisel Compilation

### Compile RTL and build simulator with Verilator

Compilation might take ~20 mins.

```shell
make emu –j4

# Options: 
# CONFIG=MinimalConfig  Configuration of XiangShan
# EMU_THREADS=4  	    Simulation threads
# EMU_TRACE=1 		    Enable waveform dump
# WITH_DRAMSIM=1	    Enable DRAMSim3 for DRAM simulation
# WITH_CHISELDB = 1	    Enable ChiselDB feature
# WITH_CONSTANTIN = 1	Enable Constantin feature
```

### Open Another Terminal

```shell
# login to the cloud server again
ssh guest@tutorial.xiangshan.cc
# Password: xiangshan-2025

# Enter your dir and set up
cd ~/<YOUR_NAME> && source env.sh

# Enter tutorial for following operations
cd tutorial
```

### Run RTL Simulation with Verilator

After building, we can run the simulator.

```shell
# run pre-built simulator
cd p1-basic-func
./emu -i hello.bin --no-diff 2>hello.err

# Some key options: 
-i				            # Workload to run
-C / -I 			        # Max cycles / Max Insts 
--diff=PATH / --no-diff		# Path of Reference Model / disable difftest
```

Great! We have learned the basic simulation process of Xiangshan.
