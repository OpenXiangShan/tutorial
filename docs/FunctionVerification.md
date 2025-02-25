# Function Verification

## Nexus-AM: Bare Metal Runtime

Nexus-AM is a bare metal runtime test generation environment. It is light-weight, and easy to use; implements basic system call interfaces and exception handlers. What'smore,
it supports multiple ISAs and configurations.

Use AM to build a coremark workload.

```shell
bash build-am.sh

# cd $AM_HOME/apps/coremark
# make ARCH=riscv64-xs \
#  ITERATIONS=1 LINUX_GNU_TOOLCHAIN=1 TOTAL_DATA_SIZE=400
# ls -l build
#    coremark-riscv64-xs.bin      Binary image of the program
#    coremark-riscv64-xs.elf      ELF of the program
#    coremark-riscv64-xs.txt      Disassembly of the program

```

## NEMU: ISA REF

We present NEMU, an instruction set simulator, similar to Spike. Through optimization techniques, achieve performance similar to QEMU. Also a set
of APIs is provided to assist XiangShan in comparing and verifying the architectural states.

Build a nemu. NEMU has two main types of configuration.

```shell
bash build-nemu.sh

# cd $NEMU_HOME
# make clean
# make riscv64-xs_defconfig
# make –j         build NEMU as the bare metal machine, can run the Coremark from the previous step
# make clean-softfloat
# make riscv64-xs-ref_defconfig
# make -j              build NEMU as the reference model for XiangShan
```

Run Coremark workload on NEMU.

```shell
bash run-nemu.sh

# cd $NEMU_HOME  
# ./build/riscv64-nemu-interpreter –b \                              run in batch mode, faster
# $AM_HOME/apps/coremark/build/coremark-1-iteration-riscv64-xs.bin   set workspace to Coremark
```

## Difftest: ISA Co-simulation Framework

We present Difftest, an ISA co-simulation framework. The basic flow is: Once our RTL Processor have instructions commit / other states update, the ISA simulator executes the same instructions. Difftest will compare the architectural states between DUT and REF. If there are differences, it will stop running and report the error. Otherwise it continues to run.

Run Coremark workload on XiangShan, difftest with NEMU.

```shell
# Running Coremark takes about 2 minutes
bash run-emu.sh

# ./emu -i \                                             use coremark.bin 
# $AM_HOME/apps/coremark/build/coremark-1-iteration-riscv64-xs.bin \  
# --diff $NEMU_HOME/build/riscv64-nemu-interpreter-so \  difftest with NEMU
# 2>perf.out    \                                        redirect stderr to file

```

Run Coremark workload on a pre-built XiangShan processor injected a bug, difftest with NEMU.

```shell
bash run-emu-diff.sh

# ./emu-bug \
#  -i $AM_HOME/apps/coremark/build/coremark-1-teration-riscv64-xs.bin \ 
# --diff $NEMU_HOME/build/riscv64-nemu-interpreter-so    # difftest with NEMU
```

Dump waveform after Difftest detects the bug.

```shell
bash run-emu-dumpwave.sh
ls $NOOP_HOME/build/*.vcd -lh  # There should be a .vcd waveform

# ./emu-bug \
# -i $AM_HOME/apps/coremark/build/coremark-1-iteration-riscv64-xs.bin \
# --diff $NEMU_HOME/build/riscv64-nemu-interpreter-so \
# -b 11000 \     the begin cycle of the wave, you may find XiangShan trap at 12000 cycles in previous step 
# -e 13000 \     the end cycle of the wave
# --dump-wave \  set this option to dump wave, you will find *.vcd on $NOOP_HOME/build 
```

## LightSSS: Light-weight Simulation SnapShot

During simulation, LightSSS will record snapshots of the process with funtion fork(). When bug is detected, it will be waked up and generate waveform of several cycles from the latest snapshot before the bug occurred.

Use LightSSS to dump waveform.

```shell
bash run-emu-lightsss.sh

# ./emu-bug \    
# -i $AM_HOME/apps/coremark/build/coremark-1-iteration-riscv64-xs.bin \ 
# --diff $NEMU_HOME/build/riscv64-nemu-interpreter-so \
# --enable-fork \        No longer need to use complex parameters
# 2 > lightsss.err
```

LightSSS is working if you see "the oldest checkpoint start to dump wave and dump nemu log...". After that, simulation will start again from the existing latest snapshot.

## ChiselDB: Debug-friendly Structured Database

We present ChiselDB, a debug-friendly structured database. It will insert probes between module interfaces in hardware, and use DPI-C in Chisel code directly to transfer bundle info and data. As for bug analysis, SQL queries are supported so it's much more easy to use than waveform.

To use ChiselDB, first step is creating table.

Here is an example, call this API function, you can create a chiselDB table and define the name and data type.

```scala
// Design source code:  XiangShan/utility/src/main/scala/utility/ChiselDB.scala
// Usage:  Create table

// API: def createTable[T <: Record](tableName: String, hw: T): Table[T] 
import huancun.utils.ChiselDB

class MyBundle extends Bundle { 
	val fieldA = UInt(10.W)
	val fieldB = UInt(20.W) 
} 
val table = ChiselDB.createTable("MyTableName", new MyBundle)

```

The second step is to register the probes.

You can use the function `table.log` to record the signals value when `en` is true.

```scala
// Usage:  Register probes

/* APIs
def log(data: T, en: Bool, site: String = "", clock: Clock, reset: Reset)
def log(data: Valid[T], site: String, clock: Clock, reset: Reset): Unit
def log(data: DecoupledIO[T], site: String, clock: Clock, reset: Reset): Unit
*/
table.log(
  data = my_data /* hardware of type T */,
  en = my_cond,   site = "MyCallSite",
  clock = clock,   reset = reset
)

```

This is a complete example. Here we record L2TLB’s missqueue’s input and output request.

```scala
// Example:  XiangShan/src/main/scala/xiangshan/cache/mmu/L2TLB.scala

class L2TlbMissQueueDB(implicit p: Parameters) extends TlbBundle {
   val vpn = UInt(vpnLen.W)
}

val L2TlbMissQueueInDB, L2TlbMissQueueOutDB = Wire(new L2TlbMissQueueDB)
L2TlbMissQueueInDB.vpn := missQueue.io.in.bits.vpn
L2TlbMissQueueOutDB.vpn := missQueue.io.out.bits.vpn

val L2TlbMissQueueTable = ChiselDB.createTable(
	"L2TlbMissQueue_hart" + p(XSCoreParamsKey).HartId.toString, new L2TlbMissQueueDB)

L2TlbMissQueueTable.log(L2TlbMissQueueInDB, missQueue.io.in.fire, "L2TlbMissQueueIn", clock, reset)
L2TlbMissQueueTable.log(L2TlbMissQueueOutDB, missQueue.io.out.fire, "L2TlbMissQueueOut", clock, reset)

```

And you can see the result of ChiselDB when you record bus transactions between caches to SQLite3 database, we call it TL-Log, because we use Tilelink bus protocol. It support offline bug detect and performance analysis.

Let's inject a bug to show how to use the ChiselDB. We provide an already-compiled emu with a bug that all releasedata from L2 to L3 cache are constant value.

```shell
cd ../p2-chiseldb && cat cdb_err.patch

# --- a/src/main/scala/huancun/noninclusive/SinkC.scala
# +++ b/src/main/scala/huancun/noninclusive/SinkC.scala

# @@ -99,7 +99,7 @@ class SinkC(implicit p: Parameters) extends BaseSinkC {
# -      buffer(insertIdx)(count) := c.bits.data
# +      buffer(insertIdx)(count) := 0xABCDEF.U
```

Run the command below you can simulate the pre-built emu. There is a new parameter `–dump-db` to dump the data base.

```shell
# simulate using pre-built emu (ChiselDB enabled)
bash chiseldb_step1_run.sh

# ./emu-cdb-err -i $NOOP_HOME/ready-to-run/linux.bin \
# --diff $NOOP_HOME/ready-to-run/riscv64-nemu-interpreter-so \
# --dump-db   # set this argument to dump data base
# 2>linux.err
```

Difftest detect refill test failed at this address 0x80040580. So we can analyze it. Run analyze script to query for all transactions on the error addr, and we have script to parse the TL-Log.

```shell
# Analyze
bash chiseldb_step2_analyze.sh

# 1. sqlite query for all transactions on Eaddr
# 2. use script to parse TLLog
# sqlite3 $NOOP_HOME/build/*.db \
# "select * from TLLOG where ADDRESS=0x80040580" | \
# sh $NOOP_HOME/scripts/utils/parseTLLog.sh
```

Now, you can see the query result. We can see which transaction is wrong, and the precise cycle number.

## TL-Test: Cache Verification Framework

TL-Test is a Unit Level Verification Framework for cache system. As an infrastructure for cache verification, it support Tilelink protocolandcache coherence check. And TL-Test can generate testcases or constrained random test quickly.

What's more, TL-Test support converting real cache access trace into testbench, so that we can dump trace from the whole XS-Core and run Cache system alone as a trace-driven model. The tl-trace mode can be used for fast reproduction of functional bugs and analysis of performance problems.

Change to the TL-Test tutorial dir.

```shell
# Project Structure
cd $TLT_HOME && tree -d -L 1

# .
# ├── assets
# ├── configs
# ├── dut
# ├── main
# ├── run
# ├── scripts
# └── tutorial

# Enter TL-Test tutorial
cd ../tutorial/p3-tl-test
```

Analysis of Cache Coherence Violation.

```shell
# Inject bug – We wrongly shift the Grant Data (L2->L1)
cat tlt_err.patch

--- a/tl-test-new/dut/CoupledL2/src/main/scala/coupledL2/GrantBuffer.scala
+++ b/tl-test-new/dut/CoupledL2/src/main/scala/coupledL2/GrantBuffer.scala
@@ -94,7 +94,7 @@ class GrantBuffer(implicit p: Parameters) extends LazyModule {
-      d.data := data
+      d.data := data << 8
```

We also provide a pre-compiled TL-Test Run the TL-Test.

```shell
bash tltest_step1_run.sh

# cd $TLT_HOME/run && ./tltest_v3lt 2>&1 | tee tltest_v3lt.log
```

TL-Test will generate constrained random testcase, and find something wrong with our cache design at this address 0x4000.

Now we can analyze the bug by chiselDB mentioned before.

```shell
# Analyze
bash tltest_step2_analyze.sh

# 1. grep all transactions on Eaddr(0x4000)
# 2. use TL-Log to help debugging

# grep “addr: 0x4000” $TLT_HOME/run/tltest_v3lt.log
```
