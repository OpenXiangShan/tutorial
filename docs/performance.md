# MinJie Performance Verification

The performance verification of Minjie is a crucial part in processor development. Similar to Functional Verification, we have established a cycle consisting of RTL implementation, test running, performance evaluation, performance analysis, and then implement improvements. 

In this section, we will provide a demonstration of the performance evaluation.

## Performance Evaluation: Checkpoint

The Checkpoint technique is widely used to improve the efficiency of performance evaluation. Common slicing techniques include Uniform checkpoint and SimPoint checkpoint:

**Uniform Checkpoint** divides a program into equal segments for parallel simulation with uniform sampling. **SimPoint Checkpoint** selects representative segments through cluster analysis and weighted sampling.

In this section, we will provide a demonstration of generating an agile test suite using SimPoint/Uniform checkpoint techniques.

### Simpoint Checkpoint

Step 0: Prepare NEMU environment for SimPoint checkpoint.

```
cd ../p4-checkpoint && bash simpoint_step0-prepare.sh
```

```
# simpoint_step0_prepare.sh (~23s)

# cd $NEMU_HOME
# git submodule update --init
# cd $NEMU_HOME/resource/simpoint/simpoint_repo
# make clean && make		# generate simpoint generator binary
 
# cd $NEMU_HOME
# make clean
# make riscv64-xs-cpt_defconfig && make -j 8	# compile NEMU

# cd $NEMU_HOME/resource/gcpt_restore
# rm -rf $XS_PROJECT_ROOT/tutorial/part5-checkpoint/gcpt
# make –C $NEMU_HOME/resource/gcpt_restore/ \	# generate gcpt restorer binary
  	O=$XS_PROJECT_ROOT/tutorial/part5-checkpoint/gcpt \ # directory of results
	GCPT_PAYLOAD_PATH=$XS_PROJECT_ROOT/tutorial/part5-	checkpoint/bin/stream_100000.bin

```

Step 1: Execute the workload and collect program behavior.

```
bash simpoint_step1-profiling.sh
```

```
# simpoint_step1_profiling.sh (~18s)
# source simpoint_env.sh			# configure environment variables

# rm –rf $RESULT

# $NEMU ${BBL_PATH}/${workload}.bin \  	# specify workload 
#    -b                             \  	# run with batch mode
#    -D $RESULT                     \  	# directory where the checkpoint is generated
#    -C $profiling_result_name      \  	# name of this task
#    -w stream                      \  	# name of workload
#    --simpoint-profile             \  	# is simpoint profiling
#    --cpt-interval ${interval}     \  	# simpoint interval instructions: 50,000,000
#    > >(tee $log/stream-out.txt) 2> >(tee ${log}/stream-err.txt) # redirect stdout and stderr

```

Step 2: Cluster and obtain multiple representative slices.

```
bash simpoint_step2_cluster.sh
```

```
# simpoint_step2_cluster.sh

# export CLUSTER=$RESULT/cluster/${workload} && mkdir -p $CLUSTER
# mkdir –p $LOG_PATH/cluster_logs/cluster

# random1=`head -20 /dev/urandom | cksum | cut -c 1-6`
# random2=`head -20 /dev/urandom | cksum | cut -c 1-6`

# $NEMU_HOME/resource/simpoint/simpoint_repo/bin/simpoint   \
#    -loadFVFile $PROFILING_RES/${workload}/simpoint_bbv.gz \   # load a frequency vector file
#    -saveSimpoints $CLUSTER/simpoints0                     \   # file to save simpoints
#    -saveSimpointWeights $CLUSTER/weights0                 \   # file to save simpoints weights
#    -inputVectorsGzipped                                   \   # input vectors have been compressed with gzip
#    -maxK 3                \  # maximum number of clusters to use
#    -numInitSeeds 2        \  # times of different random initialization for each run, taking only the best clustering
#    -iters 1000            \  # maximum number of iterations that should perform
#    -seedkm ${random1}     \  # random seed for choosing initial k-means centers
#    -seedproj ${random2}   \  # random seed for random linear projection
#    > >(tee $log/${workload}-out.txt) 2> >(tee $log/${workload}-err.txt) 
			        # redirect stdout and stderr

```

Step 3: Generate corresponding Checkpoints based on clustering results.

```
bash simpoint_step3_genspt.sh
```

```
# simpoint_step3_genspt.sh

# export CLUSTER=$RESULT/cluster
# mkdir –p $LOG_PATH/checkpoint_logs

# $NEMU ${BBL_PATH}/${workload}.bin \
#    -b                             \ run with batch mode
#    -D $RESULT                     \ directory where the checkpoint is generated
#    -C checkpoint                  \ name of this task
#    -w stream                      \ name of workload
#    -S $CLUSTER                    \ simpoints results director
#    --cpt-interval $interval       \ simpoint interval instructions: 50,000,000
#    > >(tee $log/stream-out.txt) 2> >(tee $log/stream-err.txt)					    redirect stdout and stderr

```


Step 4: Run the SimPoint checkpoint on NEMU or XiangShan.

Here we take XiangShan as an example.

```
bash simpoint_step4_run_xs.sh
```

```
# simpoint_step4_run_xs.sh (~ 1 min)
# ./emu \
#    -i `find $RESULT/checkpoint/stream -type f -name "*_.gz" | tail -1` \
				 get the path of workload
#   --diff $NOOP_HOME/ready-to-run/riscv64-nemu-interpreter-so \
				Enable the reference standard design path for difftest
#   --max-cycles=50000 \     Maximum execution instruction count
#   2>simpoint.err

```

Step 5: Provide configuration files for batch running on Gem5/XiangShan.

```
python3 simpoint_step5_dumpresult.py 
ls –l simpoint_result/checkpoint
```

### Uniform Simpoint

Step 0: Prepare NEMU environment for checkpoints.

```
cd $XS_PROJECT_ROOT/tutorial/p4-checkpoint 
bash simpoint_step0_prepare.sh
```

Step 1: Generate uniform checkpointsusing NEMU.

```
bash uniform_cpt.sh
```

Step 2: Run uniform checkpoint on NEMU or XiangShan.

Taking NEMU as an example

```
bash uniform_run_nemu.sh
```

## Performance Analysis

After obtaining performance data, we mainly use three tools to conduct efficient performance analysis: **XSPerf**, **Costantin**, and **Top-Down**. 

In this section, we will provide provide the setup process and a demonstration of performance analysis using the aforementioned three tools.

### XSPerf

XSPerf is a performance analysis tool based on chiselDB, used for specific performance data and visualization processing. 

It provides three data processing modes: accumulation, histogram, and rolling.

#### Accumulate & Histogram

Accumulate is the basic processing mode for accumulator counter.

Histogram is counting the distrbution.

How to use:

add `XSPerfAccumulate(‘name’, signal) or XSPerfHistogram(‘name’, signal)  .`

set `DebugOptions(EnablePerfDebug = false/true) `to turn off/on.

Example:

```
cd ../p5-xs-perf
bash xs-perf-log.sh | head –n 20
```

```
# print the log result of p1-basic-func
# cat ${XS_PROJECT_ROOT}/tutorial/p1-basic-func/perf.err

```

#### Rolling

Rolling is an advanced data processsing mode to roll curve colleciton and visualization.

How to use:

add ` XSPerfRolling(‘name’, perfCnt, granularity, clock, reset.`

compile with option `WITH_CHISELDB=1 WITH_ROLLINGDB=1` and run with option `--dump-db`

Example:

step 1: Build emu with rolling  (time consuming, use pre-built emu instead)

```
bash xs-perf-prepare.sh
```

```
# cd ${NOOP_HOME}
# make clean
# make emu EMU_THREADS=4 WITH_CHISELDB=1 WITH_ROLLINGDB=1 -j8 \
#     PGO_WORKLOAD=${NOOP_HOME}/ready-to-run/coremark-2-iteration.bin \
#     PGO_MAX_CYCLE=10000 PGO_EMU_ARGS=--no-diff LLVM_PROFDATA=llvm-profdata
# ./build/emu -i ./ready-to-run/coremark-2-iteration.bin \
#     --diff ./ready-to-run/riscv64-nemu-interpreter-so --dump-db
# cp `find $NOOP_HOME/build/ -type f -name "*.db" | tail -1` \
#     ${XS_PROJECT_ROOT}/tutorial/p5-xs-perf/xs-perf-rolling.db

```

step 2: check the results and plot the curve of performance segments

```
bash xs-perf-rolling.sh
```

```
# cd ${NOOP_HOME}/scripts/rolling
# python3 rollingplot.py ${XS_PROJECT_ROOT}/tutorial/p6-xs-perf/XsPerfRolling-test.db ipc
# ls ${NOOP_HOME}/scripts/rolling/results

perf.png
```

### Top-Down

Top-Down is a hierarchical performance analysis and optimization method, widely used in efficient performance evaluation processes. 

We have applied the Top-Down framework in both XS-Gem5 and RTL, and optimized configurations specifically for the RISC-V instruction set, including performance counter optimization. 

In this section, we will provide the setup process for Top-Down and a demonstration of agile performance analysis using the Top-Down approach.

Here is the Top-Down Performance Counter Setup Process:

Step 1 : setting performance Counters in RTL code:

Example:

```
# XiangShan/src/main/scala/xiangshan/backend/dispatch/Dispatch.scala

val stallReason = Wire(chiselTypeOf(io.stallReason.reason))
// ...
TopDownCounters.values.foreach(ctr =>
  XSPerfAccumulate(ctr.toString(), PopCount(stallReason.map(_ === ctr.id.U)))
)

```

Step2 : do simulation to collect performance counter data

Step3 : Analyze the performance counter data through Top-Down method

Example:

```
# XiangShan/scripts/top-down/configs.py

xs_coarse_rename_map = {
  'OverrideBubble': 'MergeFrontend',
  'FtqFullStall': 'MergeFrontend',
  'IntDqStall': 'MergeCoreDQStall',  # attribute perf-counters to Top-Down hierarchy
  'FpDqStall': 'MergeCoreDQStall’
}

```

Step 4: Obtain analysis results and make targeted optimizations

Then try the hands-on to analyze the congestion causes in RTL using the Top-Down tool.

```
bash rtl-top-down.sh
```

```
# cd ${NOOP_HOME}/scripts/top-down && \
  python3 top_down.py \
  -s /opt/SPEC06_EmuTasks_topdown \    #path of performance counter results
  -j /opt/SPEC06_EmuTasks_topdown.json #json file of checkpoints configuration

# ls ${NOOP_HOME}/scripts/top-down/results
result.png  results.csv  results-weighted.csv #output of visual analysis results
```

### ConstantIn

ConstantIn is a performance analysis tool based on DPI-C, which allows for obtaining configured constants at runtime. This makes it possible to perform agile performance testing under different parameters without the need for recompilation.

In this section, we will provide the setup process for ConstantIn and a demonstration of agile performance analysis using ConstantIn.

#### Basic Usage

Here is the example to create constant by the API:

```
# XiangShan/coupledL2/src/main/scala/coupledL2/prefetch/TemporalPrefetch.scala
require(cacheParams.hartIds.size == 1)
val hartid = cacheParams.hartIds.head
// 0 / 1: whether to enable temporal prefetcher
private val enableTP = WireInit(Constantin.createRecord("enableTP" + hartid.toString, initValue = 1.U))
// 0 ~ N: throttle cycles for each prefetch request
private val tpThrottleCycles = WireInit(Constantin.createRecord("tp_throttleCycles" + hartid.toString, initValue = 4.U(3.W)))
// 0 / 1: whether request to set as trigger on meta hit
private val hitAsTrigger = WireInit(Constantin.createRecord("tp_hitAsTrigger" + hartid.toString, initValue = 1.U))
// 1 ~ triggerQueueDepth: enqueue threshold for triggerQueue
private val triggerThres = WireInit(Constantin.createRecord("tp_triggerThres" + hartid.toString, initValue = 1.U(3.W)))
// 1 ~ tpEntryMaxLen: record threshold for recorder and sender (storage size will not be affected)
private val recordThres = WireInit(Constantin.createRecord("tp_recordThres" + hartid.toString, initValue = tpEntryMaxLen.U))
// 0 / 1: whether to train on vaddr
private val trainOnVaddr = WireInit(Constantin.createRecord("tp_trainOnVaddr" + hartid.toString, initValue = 0.U))
// 0 / 1: whether to eliminate L1 prefetch request training
private val trainOnL1PF = WireInit(Constantin.createRecord("tp_trainOnL1PF" + hartid.toString, initValue = 0.U))
```

Then try the hands-on to pass constant via standard input stream

```
cd ../p6-constantin
bash step0-build.sh
# please input total constant number
2
# please input each constant ([constant name] [value])
DelayQueueLatencyvbop 175
DelayQueueLatencypbop 175
```

#### AutoSolving

ConstantIn also supports the auto-solving feature, which is used to automatically find the best constant values.

Here is the set-up process of autosolving:

Step 0: Enable `AutoSolving` & prepare signals (like Basic Use) you need

Step 1: Compile the emu with `WITH_CONSTANTIN=1`

Step 2: Run basic demonstration

Step 3: Set parameters for `AutoSolvingin` a configuration file

Example:

```
# Automatical parameter solver configuration file
cat my_constantin.json 
```

Step 4: Run the script to find automatically

```
# Run auto solver, output will be the optimal constant currently found
bash step2-solve.sh
```

```
# ～2min
# The solver generates optimal parameters
# opt constant in this round is  [['DelayQueueLatencyvbop', 8], ['DelayQueueLatencypbop', 72]]  fitness is  54674
# opt constant for gene algrithom is  [['DelayQueueLatencyvbop', 9], ['DelayQueueLatencypbop', 73]]  fitness 54674

```

## Performance simulation

Verilator and Dramsim3 are merely methods to accelerate RTL emulation. However, the compilation and execution times of the aforementioned methods are still quite lengthy. Therefore, we employ the gem5 tool to expedite performance simulation, which is utilized for routine performance exploration.

In this section, we will provide a demonstration of performance simulation using the XS-Gem5.

### Compile and build

The following instructions give a simpile example to compile and build XS-Gem5:

```
cd ../p7-xs-gem5 # Enter  XS-Gem5  tutorial directory
bash 0-gem5_prepare.sh
cd ~/pre/tutorial/p5-xs-gem5 && export gem5_home=`~/pre/gem5`
```

### Run and analyze

Step 1 run bare-metal workload on XS-Gem5

```
bash 1-gem5_run_coremark.sh
```

```
# pushd $gem5_home && \
# export GCBV_REF_SO=$NEMU_HOME/build/riscv64-nemu-gem5-ref-so && \
# mkdir -p util/xs_scripts/coremark && \ # Setup CoreMark working directory
# cd util/xs_scripts/coremark && \
# $gem5_home/build/RISCV/gem5.opt $gem5_home/configs/example/xiangshan.py \
# --raw-cpt \
# --generic-rv-cpt=$NOOP_HOME/ready-to-run/coremark-2-iteration.bin && \
# popd
```

Step 2 analyze the performance counters

```
bash 2-gem5_counter.sh | head -n 20
```

```
cat $gem5_home/util/xs_scripts/coremark/m5out/stats.txt
```

Step 3 run the Cache MPKI analysis script

```
bash 3-gem5_cache.sh
```

```
# pushd gem5_data_proc && \
# mkdir -p results && \
# export PYTHONPATH= pwd && \
# python3 batch.py \
# -s /home/share/xs-model-l1bank \
# -o gem5-cache-example.csv \
# --cache && \
# python3 simpoint_cpt/compute_weighted.py \
# -r gem5-cache-example.csv \
# -j /home/share/xs-model-l1bank/cluster-0-0.json \
# -o weighted.csv

```

Step 4 run the Top-Down analysis script

```
bash 4-gem_topdown.sh
```

```
# pushd gem5_data_proc && \
# python3 batch.py \
# -s $gem5_home/util/xs_scripts/coremark \
# -t --topdown-raw && \
# popd
```

Step 5 run the SPEC CPU score calculation script

```
bash 5-gem5_spec06_score.sh
```

```
# pushd gem5_data_proc && \
# mkdir -p results && \
# export PYTHONPATH= pwd && \
# python3 batch.py -s ../data/xs-model-l1bank \
# -o gem5-score-example.csv && \
# python3 simpoint_cpt/compute_weighted.py \
# -r gem5-score-example.csv \
# -j ../data/xs-model-l1bank/cluster-0-0.json \
# --score score.csv

```
