# Start Up

## Online

In this tutorial, we will provide access to cloud servers, prepare the XiangShan development environment for you, and go through the development workflows incluing simulation, function verification and performance verification. All you need is a computer with an Internet connection and `ssh` tools to do this!

### Demo Instructions

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

### Prerequisites

Login to the provided cloud server.

**The server is only available during the tutorial.**

Please open a Terminal and run the following commands.

* For Windows User, Windows Terminal with PowerShell is recommended
* For Mac / Linux User, just open "Terminal".

```powershell
ssh guest@t.xiangshan.cc
# Password: xiangshan-2025
```

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

### Chisel Compilation

#### Compile RTL and build simulator with Verilator

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

#### Open Another Terminal

```shell
# login to the cloud server again
ssh guest@t.xiangshan.cc
# Password: xiangshan-2025

# Enter your dir and set up
cd ~/<YOUR_NAME> && source env.sh

# Enter tutorial for following operations
cd tutorial
```

#### Run RTL Simulation with Verilator

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


## Offline

If you want to build the Xiangshan environment on your own server, please refer to the following operations.

Please prepare a server with relatively high performance. The following are some configuration requirements for the server:

* Operating system: Ubuntu 22.04 LTS (Other versions have not been tested and are not recommended. **NOTE:** the Xiangshan environment corresponding to Ubuntu 20.04 LTS is no longer maintained.)
* CPU: Not limited. The performance will determine the speed of compilation and generation.
* Memory: At least 32G. 64G or more is recommended.
* Disk space: 20G or more.
* Network: Please configure a smooth network environment.

For detailed steps, please refer to: [xs-env](https://docs.xiangshan.cc/zh-cn/latest/tools/xsenv/)

**Step 1:** clone the environment and install the tools.

```bash
# clone xs-env
git clone https://github.com/OpenXiangShan/xs-env
cd xs-env
git checkout -b tutorial-2025 origin/tutorial-2025
# use apt to install dependencies, you may modify it to use different pkg manager
sudo -s ./setup-tools.sh
# prepare tools, test develop env using a small project
source setup.sh
```

**Step 2:** download `riscv-gnu-toolchain`.

Please refer to the [toolchain](https://docs.xiangshan.cc/zh-cn/latest/workloads/toolchain/) for details.

```bash
# NOTE: please download the toolchain that is compatible with your environment. Here, I use Ubuntu 22.04.
wget https://github.com/riscv-collab/riscv-gnu-toolchain/releases/download/2025.01.20/riscv64-glibc-ubuntu-22.04-gcc-nightly-2025.01.20-nightly.tar.xz
wget https://github.com/riscv-collab/riscv-gnu-toolchain/releases/download/2025.01.20/riscv64-elf-ubuntu-22.04-gcc-nightly-2025.01.20-nightly.tar.xz
sudo tar -xJf riscv64-glibc-ubuntu-22.04-gcc-nightly-2025.01.20-nightly.tar.xz -C /opt
sudo tar -xJf riscv64-elf-ubuntu-22.04-gcc-nightly-2025.01.20-nightly.tar.xz -C /opt

# set environment variables
vim ~/.bashrc
# add `export PATH=/opt/riscv/bin:$PATH` to the end of the file.

# update the environment variables
source ~/.bashrc
# check the status
riscv64-unknown-linux-gnu-gcc --version
riscv64-unknown-elf-gcc --version
```

**Step 3:** install the necessary Python libraries.
```bash
pip install -r $NOOP_HOME/scripts/requirements.txt
```

**Step 4:** copy the attachments in [release](https://github.com/OpenXiangShan/xs-env/releases) to the corresponding path.
