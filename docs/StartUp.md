# Start Up

<!-- ## Online -->

<!-- In this tutorial, we will provide access to cloud servers, prepare the XiangShan development environment for you, and go through the development workflows incluing simulation, function verification and performance verification. All you need is a computer with an Internet connection and `ssh` tools to do this! -->

<!-- ## Offline -->

In this tutorial, we provide the procedure to prepare the XiangShan development environment for you, and go through the development workflows incluing simulation, function verification and performance verification.

<!-- If you want to build the Xiangshan environment on your own machine, please refer to the following operations. -->

Please prepare a server with relatively high performance. The following are some configuration requirements for the server:

* Operating system: Ubuntu 22.04 LTS (Other versions have not been tested and are not recommended. **NOTE:** the Xiangshan environment corresponding to Ubuntu 20.04 LTS is no longer maintained.)
* CPU: Not limited. The performance will determine the speed of compilation and generation.
* Memory: At least **32G**. 64G or more is recommended.
* Disk space: 20G or more.
* Network: Please configure a smooth network environment.

For detailed steps, please refer to: [xs-env](https://docs.xiangshan.cc/zh-cn/latest/tools/xsenv/)

**Step 1:** download `riscv-gnu-toolchain`.

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

**Step 2:** clone the environment and install the tools.

```bash
# clone xs-env
git clone https://github.com/OpenXiangShan/xs-env
cd xs-env
git checkout isca2025-tutorial
# Use apt to install dependencies, you may modify it to use different pkg manager
sudo -s ./setup-tools.sh
# Prepare tools, test develop env using a small project
# Please configure a smooth network environment
source setup.sh
# This script will setup XiangShan environment variables
source env.sh
```

**Step 3:** copy the attachments

Some of the pre-built programs in this step only work for Ubuntu-22.04.
If you encountered any issue with other distributions, you can try to compile
corresponding programs yourself.

```bash
cd $XS_PROJECT_ROOT/..
wget https://github.com/OpenXiangShan/xs-env/releases/download/isca2025-tutorial/xs-env.tar.gz
tar -xzf xs-env.tar.gz
```

**Step 4:** install the necessary Python libraries.

```bash
pip install -r $NOOP_HOME/scripts/requirements.txt
```

**Step 5:** Compile RTL and build simulator with Verilator.

Compilation might take ~20 mins.

```shell
# Project Structure
tree -d -L 1
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

# initialize submodule of XiangShan
make init

# compile XiangShan
make emu -j4

# Options: 
# CONFIG=MinimalConfig  Configuration of XiangShan
# EMU_THREADS=4         Simulation threads
# EMU_TRACE=1           Enable waveform dump
# WITH_DRAMSIM=1        Enable DRAMSim3 for DRAM simulation
# WITH_CHISELDB = 1     Enable ChiselDB feature
# WITH_CONSTANTIN = 1   Enable Constantin feature
```

Next step: [Function Verification](FunctionVerification.md)