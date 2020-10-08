---
title: 'Academic: the website builder for Hugo'
subtitle: 'Create a beautifully simple website in under 10 minutes :rocket:'
summary: Create a beautifully simple website in under 10 minutes.
authors:
- admin
- 吳恩達
tags:
- Academic
- 开源
categories:
- Demo
- 教程
date: "2016-04-20T00:00:00Z"
lastmod: "2019-04-17T00:00:00Z"
featured: false
draft: false
本文从模型搜索**NAS**的问题出发，整理了最新**ICLR2021**相关投稿论文。 神经网络除了权重**(W)**之外，其通道数，算子类型和网络连接等结构参数需要设定，而模型搜索NAS即是确定结构参数的自动方法。最初**NASNet**中每种结构参数的模型单独训练带来的巨大开销，最近两年基于权重共享的NAS方法中，不同结构参数模型复用权重组成代理模型**(SuperNet)**一起训练，然后评测子模型指标并通过**RL , EA , Random**搜索**(One-shot)**或由参数化离散变连续用梯度下降**(Darts)**从结构参数空间**(A)**求解出最优子结构，最后重训最优子结构得最后需要的模型。整个流程中分为**SuperNet**训练，最优子模型搜索，重训三个阶段，其中搜索阶段时间因为不同的评测方式和指标，快则几秒慢则几天，而**SuperNet**训练周期一般设置成重训阶段相近，因此目前流行的权重共享搜索方法多是单独训练的两倍左右开销。其中结构参数空间如何建模，代理模型评测好坏是否真实(一致性)，以及训练开销是否可以进一步降低，这些问题对应投稿论文整理如下：

- **One-shot方法中SuperNet训练以及一致性？**  One-shot方法中大多以megvii 的Singlepath为framework，之后的改进工作主要集中在采样方式和具体训练策略上，GreedyNAS 和 AngleNAS分别用droppath和dropnode改进采样方式，Once for all和BigNAS利用训练策略使得SuperNet中子模型性能变强而省去了重训步骤，文【1】是年初的文章总结了训练细节比如分组BN等影响。 One-shot框架中其他问题如代理模型评测误差等也都有ICLR2021投稿工作研究；
- **Darts方法的训练和优化方式？**  Darts方法用bi-level优化轮替优化权重(W)和结构参数(A)，因为softmax以及无参数OP影响导致模型坍塌以及结果不稳定，文【2】通过引入辅助分支改善训练，文【3】加noise改善了模型坍塌，文【4】和文【5】都是解耦连接和OP的搜索。由于bi-level求解存在优化误差文，文【6】和文【7】采用single-level优化方法，希望这两篇工作能结束魔改Darts的局面；
- **结构参数空间怎么建模？**比如文【8】按照分布建模，文【9】按照流形建模，文【10】按照邻接关系建模，随着NAS [benchmarks](https://github.com/automl/nas_benchmarks)101等出现，直接用结构参数作为输入X，提前测试好的对应精度作为Y, 学习X到Y映射关系可以看作对搜索空间建模，这样工作有基于GCN, LSTM等的预测器以及排序器等，如文【6】，文【7】和文【11】等；
- **NAS without training?** 文【13】和 文【14】类似，以及我们所更早的工作ModuleNet，都是通过精心设计指标取代精度来避免权重训练的开销，虽然结果图有相关趋势，但是经实际评测一致性数值较低，其他如随机权重的工作基本只能解决很简单任务难以实用；
- **新的NAS** [**benchmark**](https://github.com/automl/nas_benchmarks)**?** 如 [TransNAS-Bench-101](https://openreview.net/forum?id=HUd2wQ0j200)，[NAS-Bench-301](https://openreview.net/forum?id=1flmvXGGJaa)，[HW-NAS-Bench](https://openreview.net/forum?id=_0kaDkv3dVf)等，都是推动领域大大的好工作，respect！
- **NAS的下游任务应用？**检测上应用如文【16】, 量化上应用如文【17】等。

**总结：**48篇文章中既有延续之前解决One-shot和Darts框架中训练不稳定以及减少评测误差等方面问题的文章，也有single-level优化，流形结构建模等反思前提假设的硬核文章，更多的NAS benchmark以及泛化应用推动NAS研究的进步与落地。诚然，模型搜索方面的研究大厦似乎已经落成，部分新的改进工作因为评测数据集简单(如cifar等)和不公平对比而难以判断良莠，期待大佬组以及大厂能够持续开坑以及贡献更多solid工作。

**ICLR2021中NAS方面投稿文章整理：**

**1.** [**How to Train Your Super-Net: An Analysis of Training Heuristics i**](https://openreview.net/forum?id=txC1ObHJ0wB)**n Weight-Sharing NAS**

**2.** [**DARTS-: Robustly Stepping out of Performance Collapse Without Indicators**](https://openreview.net/forum?id=KLH36ELmwIB)

**3.** [**Noisy Differentiable Architecture Search**](https://openreview.net/forum?id=JUgC3lqn6r2) 

**4.** [**FTSO: Effective NAS via First Topology Second Operator**](https://openreview.net/forum?id=7Z29QbHxIL)

Our method, named FTSO, reduces NAS's search time from days to 0.68 seconds while achieving 76.42% testing accuracy on ImageNet and 97.77% testing accuracy on CIFAR10 via searching for network topology and operators separately

**5.** [**DOTS: Decoupling Operation and Topology in Differentiable Architecture Search**](https://openreview.net/forum?id=y6IlNbrKcwG)

We improve DARTS by discoupling the topology representation from the operation weights and make explicit topology search.

**4.** [**Geometry-Aware Gradient Algorithms for Neural Architecture Search**](https://openreview.net/forum?id=MuSYkd1hxRP)

Studying the right single-level optimization geometry yields state-of-the-art methods for NAS.

**5.** [**GOLD-NAS: Gradual, One-Level, Differentiable**](https://openreview.net/forum?id=DsbhGImWjF)

A new differentiable NAS framework incorporating one-level optimization and gradual pruning, working on large search spaces.

**6.** [**Weak NAS Predictor Is All You Need**](https://openreview.net/forum?id=kic8cng35wX)

We present a novel method to estimate weak predictors progressively in predictor-based neural architecture search. By coarse-to-fine iteration, the ranking of sampling space is refined gradually which helps find the optimal architectures eventually.

**7.** [**Differentiable Graph Optimization for Neural Architecture Search**](https://openreview.net/forum?id=NqWY3s0SILo)

we learn a differentiable graph neural network as a surrogate model to rank candidate architectures.

**8 .** [**DrNAS: Dirichlet Neural Architecture Search**](https://openreview.net/forum?id=9FWas6YbmB3) 

we propose a simple yet effective progressive learning scheme that enables searching directly on large-scale tasks, eliminating the gap between search and evaluation phases. Extensive experiments demonstrate the effectiveness of our method. 

**9 .** [**Neural Architecture Search of SPD Manifold Networks**](https://openreview.net/forum?id=1toB0Fo9CZy)

we first introduce a geometrically rich and diverse SPD neural architecture search space for an efficient SPD cell design. Further, we model our new NAS problem using the supernet strategy which models the architecture search problem as a one-shot training process of a single supernet.

**10 .** [**Neighborhood-Aware Neural Architecture Search**](https://openreview.net/forum?id=KBWK5Y92BRh)

We propose a neighborhood-aware formulation for neural architecture search to find flat minima in the search space that can generalize better to new settings.

**11 .** [**A Surgery of the Neural Architecture Evaluators**](https://openreview.net/forum?id=xBoKLdKrZd)

This paper assesses current fast neural architecture evaluators with multiple direct criteria, under controlled settings.

**12 .** [**Exploring single-path Architecture Search ranking correlations**](https://openreview.net/forum?id=J40FkbdldTX)

An empirical study of how several method variations affect the quality of the architecture ranking prediction.

**13 .** [**Neural Architecture Search without Training**](https://openreview.net/forum?id=g4E6SAAvACo)

**14 .** [**Zero-Cost Proxies for Lightweight NAS**](https://openreview.net/forum?id=0cmMMy8J5q)

A single minibatch of data is used to score neural networks for NAS instead of performing full training.

**15 .** [**Improving Zero-Shot Neural Architecture Search with Parameters Scoring**](https://openreview.net/forum?id=4QpDyzCoH01) 

A score can be designed taking into account the jacobian in parameter space, that is highly predictive of final performance in a task.

**16 .** [**Multi-scale Network Architecture Search for Object Detection**](https://openreview.net/forum?id=mo3Uqtnvz_)

**17.** [**Triple-Search: Differentiable Joint-Search of Networks, Precision, and Accelerators**](https://openreview.net/forum?id=OLOr1K5zbDu&noteId=OLOr1K5zbDu)

We propose the Triple-Search framework to jointly search network structure, precision and hardware architecture in a differentiable manner.

**18 .** [**TransNAS-Bench-101: Improving Transferrability and Generalizability of Cross-Task Neural Architecture Search**](https://openreview.net/forum?id=HUd2wQ0j200) 

**19 .** [**Searching for Convolutions and a More Ambitious NAS**](https://openreview.net/forum?id=ascdLuNQY4J) 

**20 .** [**EnTranNAS: Towards Closing the Gap between the Architectures in Search and Evaluation**](https://openreview.net/forum?id=qzqBl_nOeAQ) 

We show how effective dimensionality can shed light on a number of phenomena in modern deep learning including double descent, width-depth trade-offs, and subspace inference, while providing a straightforward and compelling generalization metric.

**21 . Efficient Graph Neural Architecture Search**

 By designing a novel and expressive search space, an efficient one-shot NAS method based on stochastic relaxation and natural gradient is proposed. 

**22 .** [**Searching for Convolutions and a More Ambitious NAS**](https://openreview.net/forum?id=ascdLuNQY4J) 

A general-purpose search space for neural architecture search that enables discovering operations that beat convolutions on image data.

**23 .** [**Exploring single-path Architecture Search ranking correlations**](https://openreview.net/forum?id=J40FkbdldTX)

An empirical study of how several method variations affect the quality of the architecture ranking prediction.

**24 .** [**Network Architecture Search for Domain Adaptation**](https://openreview.net/forum?id=4q8qGBf4Zxb)

**25.** [**HW-NAS-Bench: Hardware-Aware Neural Architecture Search Benchmark**](https://openreview.net/forum?id=_0kaDkv3dVf)

**26.** [**Neural Architecture Search on ImageNet in Four GPU Hours: A Theoretically Inspired Perspective**](https://openreview.net/forum?id=Cnon5ezMHtu)

Our TE-NAS framework analyzes the spectrum of the neural tangent kernel (NTK) and the number of linear regions in the input space, achieving high-quality architecture search while dramatically reducing the search cost to four hours on ImageNet.

**27.** [**Stabilizing DARTS with Amended Gradient Estimation on Architectural Parameters**](https://openreview.net/forum?id=67ChnrC0ybo)

Fixing errors in gradient estimation of architectural parameters for stabilizing the DARTS algorithm.

**28.** [**NAS-Bench-301 and the Case for Surrogate Benchmarks for Neural Architecture Search**](https://openreview.net/forum?id=1flmvXGGJaa) 

**29.** [**NASOA: Towards Faster Task-oriented Online Fine-tuning**](https://openreview.net/forum?id=NqPW1ZJjXDJ)

We propose a Neural Architecture Search and Online Adaption framework named NASOA towards a faster task-oriented fine-tuning upon the request of users.

**28.** [**Model-based Asynchronous Hyperparameter and Neural Architecture Search**](https://openreview.net/forum?id=a2rFihIU7i) 

We present a new, asynchronous multi-fidelty Bayesian optimization method to efficiently search for hyperparameters and architectures of neural networks.

**29.** [**Searching for Convolutions and a More Ambitious NAS**](https://openreview.net/forum?id=ascdLuNQY4J)

A general-purpose search space for neural architecture search that enables discovering operations that beat convolutions on image data.

**30.** [**A Gradient-based Kernel Approach for Efficient Network Architecture Search**](https://openreview.net/forum?id=5fJ0qcwBNr0)

We first  formulate these two terms into a unified gradient-based kernel and then select architectures with the largest kernels at initialization as the final networks.  The new approach replaces the expensive "train-then-test'' evaluation paradigm.

**31.** [**Fast MNAS: Uncertainty-aware Neural Architecture Search with Lifelong Learning**](https://openreview.net/forum?id=IPGZ6S3LDdw) 

We proposed FNAS which accelerates standard RL based NAS process by 10x and guarantees better performance on various vision tasks.

**32.** [**Explicit Learning Topology for Differentiable Neural Architecture Search**](https://openreview.net/forum?id=AFm2njNEE1)

**33.** [**NASLib: A Modular and Flexible Neural Architecture Search Library**](https://openreview.net/forum?id=EohGx2HgNsA) 

**34.** [**TransNAS-Bench-101: Improving Transferrability and Generalizability of Cross-Task Neural Architecture Search**](https://openreview.net/forum?id=HUd2wQ0j200)

**35.** [**Rethinking Architecture Selection in Differentiable NAS**](https://openreview.net/forum?id=PKubaeJkw3) 

**36.** [**Neural Architecture Search on ImageNet in Four GPU Hours: A Theoretically Inspired Perspective**](https://openreview.net/forum?id=Cnon5ezMHtu)

**37.** [**Rapid Neural Architecture Search by Learning to Generate Graphs from Datasets**](https://openreview.net/forum?id=rkQuFUmUOg3)

We propose an efficient NAS framework that is trained once on a database consisting of datasets and pretrained networks and can rapidly generate a neural architecture for a novel dataset.

**38.** [**Interpretable Neural Architecture Search via Bayesian Optimisation with Weisfeiler-Lehman Kernels**](https://openreview.net/forum?id=j9Rv7qdXjd)

We propose a NAS method that is sample-efficient, highly performant and interpretable.

**39.** [**AutoHAS: Efficient Hyperparameter and Architecture Search**](https://openreview.net/forum?id=ykCRDlfxmk)

**40.** [**EnTranNAS: Towards Closing the Gap between the Architectures in Search and Evaluation**](https://openreview.net/forum?id=qzqBl_nOeAQ) 

**42.** [**Differentiable Graph Optimization for Neural Architecture Search**](https://openreview.net/forum?id=NqWY3s0SILo) 

we learn a differentiable graph neural network as a surrogate model to rank candidate architectures.

**41.** [**Width transfer: on the (in)variance of width optimization**](https://openreview.net/forum?id=2aw7TEq5jo)

we control the training configurations, i.e., network architectures and training data, for three existing width optimization algorithms and find that the optimized widths are largely transferable across settings. 

**42.** [**NAHAS: Neural Architecture and Hardware Accelerator Search**](https://openreview.net/forum?id=fgpXAu8puGj) 

We propose NAHAS, a latency-driven software/hardware co-optimizer that jointly optimize the design of neural architectures and a mobile edge processor.

**43.** [**Neural Network Surgery: Combining Training with Topology Optimization**](https://openreview.net/forum?id=3JI45wPuReY)

We demonstrate a hybrid approach for combining neural network training with a genetic-algorithm based architecture optimization.

**44.** [**Efficient Architecture Search for Continual Learning**](https://openreview.net/forum?id=uUX49ez8P06)

Our proposed CLEAS works closely with neural architecture search (NAS) which leverages reinforcement learning techniques to search for the best neural architecture that fits a new task.

**45.** [**Auto Seg-Loss: Searching Metric Surrogates for Semantic Segmentation**](https://openreview.net/forum?id=MJAqnaC2vO1)

Auto Seg-Loss is the first general framework for searching surrogate losses for mainstream semantic segmentation metrics. 

**46.** [**Improving Random-Sampling Neural Architecture Search by Evolving the Proxy Search Space**](https://openreview.net/forum?id=qk0FE399OJ)

**47.** [**SEDONA: Search for Decoupled Neural Networks toward Greedy Block-wise Learning**](https://openreview.net/forum?id=XLfdzwNKzch)

Our approach is the first attempt to automate decoupling neural networks for greedy block-wise learning and outperforms both end-to-end backprop and state-of-the-art greedy-learning methods on CIFAR-10, Tiny-ImageNet and ImageNet classification.

**48.** [**Intra-layer Neural Architecture Search**](https://openreview.net/forum?id=510f7KAPmYR) 

Neural architecture search at the level of individual weight parameters..
