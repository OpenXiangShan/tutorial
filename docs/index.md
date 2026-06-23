# XiangShan Tutorial @ ISCA'26

We are excited to host a tutorial on XiangShan at ISCA 2026! We look forward to seeing you at Raleigh Convention Center on Sunday, June 28, 2026.

## Highlights

We continue to optimize the tutorial content based on our latest progress and audience feedback from previous tutorials. Compared to our previous tutorial@HPCA'26, the highlights of this tutorial include:

- The latest in-development KMH-V3 microarchitecture design philosophy, insights and design details.
- The introduction and latest progress of our XSAI project, which is a unified microarchitecture for general-purpose and AI-inference uses.

## Agenda

Location: Room 306A, Raleigh Convention Center, North Carolina, USA

Time: **Sunday afternoon, June 28, 2026, 13:30 - 17:00**

| Time          | Topic                                                                                                                   | Slides                                                |
| ------------- | ----------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------- |
| 13:30 - 14:00 | [Introduction](./topic_details.md#introduction)                                                                         | [PDF](./slides/Introduction.pdf)                      |
| 14:00 - 14:50 | [Microarchitecture Design Philosophy of XiangShan](./topic_details.md#microarchitecture-design-philosophy-of-xiangshan) | [PDF](./slides/Microarchitecture.pdf)                 |
| 14:50 - 15:30 | [XSAI: From XiangShan to an AI-Ready CPU](./topic_details.md#xsai-from-xiangshan-to-an-ai-ready-cpu)                    | To be done                                            |
| 15:30 - 16:00 | Coffee Break                                                                                                            | ☕                                                     |
| 16:00 - 16:30 | [XS-GEM5 and Hands-on](./topic_details.md#xs-gem5-and-hands-on)                                                         | [PDF](./slides/GEM5.pdf)                              |
| 16:30 - 17:00 | [Development Workflows and Hands-on](./topic_details.md#development-workflows-and-hands-on)                             | [bootcamp](https://github.com/OpenXiangShan/bootcamp) |

- There will be QA sessions at the end of each topic, this is included in the time slots above.
- Also note that we're in early prepare stage, so:
  - This program is tentative and subject to change.
  - Currently, slides are inherited from HPCA'26, we'll update them later


## Overview

Over the past decade, agile and open-source hardware has gained increasing attention in both academia and industry. In 2019, the SIGARCH Visioning Workshop “Agile and Open Hardware for Next-Generation Computing” in conjunction with MICRO invited eleven experts to present their visions on this direction. We believe that open-source hardware design, and more importantly, free and open development infrastructure, has the opportunity to bring more convenience to architecture research and stimulate innovations.

A prominent example in this domain is XiangShan, an open-source, high-performance RISC-V processor that competes at an industrial level. Since its release in 2021, XiangShan has pushed the boundaries of publicly available processors and established a competitive foundation for future research in computer architecture. In this tutorial, we will share the latest advancements in XiangShan and the agile development platform MinJie, with special emphasis on XS-GEM5, a software simulator calibrated with XiangShan. We will also demonstrate how XiangShan and MinJie enable researchers agilely implement their ideas and obtain reliable evaluation results. Our work was previously published at the MICRO'22 conference, and was selected as an IEEE Micro Top Pick from the 2022 Computer Architecture Conferences.

The major goal of the tutorial is to demonstrate how the XiangShan project can make architecture research more convenient and solid. XiangShan has been developing on an agile hardware development platform called MinJie. We believe MinJie has the potential to become one of the most important infrastructures for computer architecture researchers.

In the hands-on part of this tutorial, we will guide our audience to setup XiangShan and make customization, to do research on XiangShan or XS-GEM5 agilely, and to obtain accurate and convincing evaluation results. Targeted audience includes researchers on architecture design, agile development, etc.

## Next Steps

- [Topic details](./topic_details.md)
- [Hands-on Setup Instructions](./hands_on/setup.md)
    - [Notebook preview](./hands_on/tutorial-en.ipynb)
    - [bootcamp on GitHub](https://github.com/OpenXiangShan/bootcamp)
