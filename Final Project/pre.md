# 1. 引言

线性代数操作，特别是涉及稀疏矩阵的操作，是科学计算、大数据分析和人工智能领域中至关重要的计算核心。在稀疏矩阵的计算中，稀疏矩阵与密集矩阵的乘法（SpMM）在诸如区块迭代求解器、动态仿真、图神经网络（GNN）和深度神经网络（DNN）训练等场景中有着广泛的应用。由于稀疏矩阵的特性，它们可以显著减少计算成本，但如何高效利用这种稀疏性来最大化并行资源的使用，仍然是一个挑战。

在并行计算中，通信成本通常比计算成本更为昂贵，因此减少通信成本成为提高并行稀疏矩阵-向量乘法（SpMV）和稀疏矩阵-多向量乘法（SpMM）性能的关键。本文通过分析SpMM的并行化设计空间，提出了一种优化并行化方案的算法，旨在通过减少通信开销来提高SpMM的并行性能。

# 2. 背景

SpMM操作是并行计算中不可或缺的组件，广泛用于科学计算和机器学习中。在大多数应用中，稀疏矩阵的行或列往往由不规则的非零元素构成，如何有效地对这些元素进行分区以便分配给不同的进程，是实现并行化的核心问题。尽管现有的许多并行SpMV算法可以直接复用于SpMM，但SpMM的额外并行性（即密集向量之间的独立性）为进一步优化通信成本提供了机会。

传统的SpMM并行化通常沿用SpMV的分区策略，即仅对稀疏矩阵的行进行分区。然而，本文提出的算法通过同时对稀疏矩阵和密集向量进行分区，利用更粗粒度的并行化来减少通信量，并提出了一个自适应的进程网格几何优化算法以实现更高的并行性能。

# 3. 设计空间探索

在并行化SpMM的设计空间中，最常见的并行化方法分为一维（1D）、二维（2D）和三维（3D）三种方式：

- **1D并行化**：只对矩阵的一个维度进行分区，例如对稀疏矩阵的行（m维度）或列（k维度）进行分区。这是最基础的并行化方法，通信量较小，但未充分利用多向量带来的额外并行性。
- **2D并行化**：同时对稀疏矩阵和密集向量进行分区，能够平衡工作负载，并在大规模分布式计算中减少通信开销。
- **3D并行化**：进一步利用更复杂的分区方案来提高并行性，但代价是需要更复杂的通信模式。

本文详细分析了这些并行化方案，并通过数学模型量化了不同方案的通信成本。在此基础上，作者提出了一种基于通信成本模型的算法，能够为给定进程数选择最优的进程网格几何结构，从而在多个并行层次上减少通信开销。

# 4. CRP-SpMM算法

为了优化并行化过程，本文提出了一种**通信减少的并行SpMM（CRP-SpMM）算法**。该算法基于前述的通信成本模型，通过调整进程网格的几何结构来减少通信量。CRP-SpMM算法的核心步骤包括：

1. **初始1D分区**：首先通过图划分等方法对稀疏矩阵进行初始的1D分区，为后续的2D或3D并行化打下基础。
2. **进程网格优化**：通过贪心算法探索不同的进程网格组合，并根据通信成本模型计算每个组合的通信开销，从中选取最优方案。
3. **并行化执行**：根据优化后的进程网格结构执行并行SpMM操作，并利用现有的1D并行算法进行局部计算。

CRP-SpMM算法的一个优势在于它能够与现有的硬件优化手段（如缓存优化、向量化）兼容，从而进一步提升性能。

# 5. 数值实验

## 5.1 实验设置

数值实验在Georgia Tech PACE-Phoenix集群上进行，测试了19个来自不同领域的稀疏矩阵。每个节点有两个12核处理器，节点之间通过100Gbps InfiniBand网络连接。实验使用了1D和2D并行化算法对不同规模的SpMM操作进行测试，重点分析通信成本和计算性能。

## 5.2 通信量分析

实验首先比较了1D行并行化和2D并行化的通信量，结果表明，2D并行化能够显著减少通信开销，尤其是在处理非结构化网格矩阵时（如社交网络图矩阵）。对于来自有限元方法（FEM）的矩阵，尽管使用了高质量的图划分算法（如METIS）进行1D分区，2D并行化仍然表现出更好的通信效率。

## 5.3 性能比较

实验对比了多个分布式SpMM算法的性能，包括CombBLAS库中的1.5D和2D算法、作者提出的1D行并行化算法以及CRP-SpMM算法。实验结果显示：

- CRP-SpMM在几乎所有的测试矩阵上表现优于CombBLAS库的算法，尤其是在处理大规模非结构化矩阵时，CRP-SpMM的加速效果尤为明显。
- 在强缩放实验中（即增加计算节点数时），CRP-SpMM的通信成本优势更加突出，尤其在处理大规模输入向量时。

## 5.4 运行时间分解

进一步的实验分析了各个并行算法的运行时间分解，显示出CRP-SpMM在减少通信时间上的显著优势。在一些矩阵上（如com-Orkut和ss），CRP-SpMM通过减少通信负载不平衡，显著提升了整体性能。

# 6. 结论与未来工作

本文提出的CRP-SpMM算法通过优化进程网格结构，减少了分布式并行SpMM的通信成本，在多个稀疏矩阵测试中展现了显著的性能提升。数值实验表明，在处理大规模输入向量和非结构化稀疏矩阵时，2D并行化能够大幅减少通信量，提高计算效率。

未来的研究方向包括：

1. 设计更快速的算法来为不同的进程网格选择1D分区，从而进一步减少通信成本。
2. 扩展CRP-SpMM算法以支持3D并行化，探索更加复杂的并行化方案。
3. 通过底层优化（如MPI派生数据类型和一侧通信操作）进一步提升并行SpMM的性能。


