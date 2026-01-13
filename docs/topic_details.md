# XiangShan Tutorial Topic Details @ HPCA'26

## Introduction

In June 2020, we launched OpenXiangShan project. We have developed three major generations of codenamed Yanqihu (YQH), Nanhu (NH) and Kunminghu (KMH) respectively. KMH-V2 achieves 15/GHz in SPEC06 benchmarks, which is the highest performance among open-source RISC-V processors to the best of our knowledge. Currently, we are working on KMH-V3, which targets at 22/GHz in SPEC06 at the end of 2026.

XiangShan has garnered attention in both industry and academia. It serves as the basis for custom-developed RISC-V processors by numerous companies, including SpacemiT, Lanxin Computing, Innosilicon, etc.. It has also been used in many academic research published in top conferences like MICRO and ISCA.

In this session, we will show why you should consider using XiangShan in your architecture research, covering:

- Why XiangShan chose to go open source.
- The development history of XiangShan.
- The infrastructure and toolchain XiangShan can provide for your research.
- The prosperous open-source community.

## Microarchitecture Design Philosophy of XiangShan

XiangShan KMH-V3 is a superscalar out-of-order RISC-V processor with full support for RVA23 profile. It features high-throughput frontend with advanced branch predictor, 8-issue out-of-order execution engine, high-bandwidth load/store unit and highly configurable cache system.

Instead of just giving a flat description of the microarchitecture of XiangShan, we will share our design philosophy and trade-offs in various aspects, hoping to provide insights for architecture researchers.

## Open-Source Tools and Open Problems in Agile Chip Development Infrastructure

XiangShan is developed with the belief that agile development principles are essential for modern high-performance processor design, especially in an open-source setting.
We will introduce the agile development workflow of XiangShan, including microarchitecture design, functional verification, and performance evaluation/exploration.
We further discuss key challenges unique to agile hardware development of high-performance CPUs and highlight open research problems.

- Overall chip development workflow
- Tools and methodologies used in the XiangShan project
- Open research problems

## Invited Talk: "XSCC: A High-Performance Compiler for RISC-V"

We're excited to have an invited talk from the XSCC team. In this session, they will introduce:

- The motivation and challenges in developing a high-performance compiler for RISC-V.
- The overview of XSCC.
- Key optimizations and current performance results of XSCC.

## XS-GEM5 and hands-on

XiangShan now has a calibrated software simulator XS-GEM5, as part of an integrated toolchain designed to support architecture exploration and development. In this session, we will cover:

- Introduction and motivation of XS-GEM5.
- The architecture of XS-GEM5 and how it is calibrated with XiangShan.
- The development tools we used to improve development efficiency.
- A hands-on demonstration of how to use XS-GEM5 for architecture research.

Please also check [hands-on setup instructions](./hands_on/setup.md) to take part in the hands-on session.

## Development workflows and hands-on

To accelerate the development, and the functional or performance verification process, of XiangShan, we have built a set of open-source development infrastructure called MinJie platform. In this session, we will be using XiangShan-bootcamp built with Code-Server and Jupyter Notebook, to give a easy-to-use hands-on demonstration of the development workflows based on XiangShan and MinJie platform. You will be able to try the following steps by yourself:

- Setting up the development environment of XiangShan.
- Configure and build XiangShan.
- Run XiangShan emulation with verilator.
- Debug XiangShan with MinJie, including NEMU, difftest, lightSSS, etc..
- Evaluate the performance of XiangShan with MinJie, including XSPerf, Top-down, etc..

We will also give motivations and key points of each tool in MinJie platform. We will show that based on XiangShan and MinJie platform, many architectural works can be reproduced and accelerate the interactions between academia and industry.

Please also check [hands-on setup instructions](./hands_on/setup.md) to take part in the hands-on session.
