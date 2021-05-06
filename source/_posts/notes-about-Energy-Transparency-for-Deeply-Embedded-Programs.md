---
title: 论文阅读笔记 ：Energy Transparency for Deeply Embedded Programs
date: 2019-03-31 14:29:35
tags: IoT energy LLVM-IR 
categories: 论文阅读
copyright: true
---

> 阅读了KyriakosGeorgiou等人的[Energy Transparency for Deeply Embedded Programs](
        linqy71.github.io/2019/03/31/notes-about-Energy-Transparency-for-Deeply-Embedded-Programs/georgiou2017.pdf
      )，过程中作了一些笔记。

### Abstract 
- 开发软件层面的能耗估计来代替硬件层面的能耗估计。
- 预估处于ISA层之上，LLVM 优化器之下，即LLVM IR层的能耗
- 使得对于LLVM优化器来说，能耗是可见的，从而可以做出一些减少能耗的调整

### Instruction
- 系统栈的抽象层使得程序员很难理解代码和数据结构如何影响程序执行时的能耗
- 使用电池的嵌入式设备需要经常更换电池，超低能量的内嵌设备（ultralow energy embedded devices）：unreliable；not enough energy for many IoT applications.
- 需要：让程序员了解潜在的硬件层面的energy saving.
- 提出：呈现用于一些嵌入式系统的软件的实际能耗以及能耗的界限，以便于提出一些节约能耗的方案。
- 物理测量（physical measurement）：not easily accessable; 能量监测计数器(energy-monitoring counters): number and acailability limited; 总之，简单的物理层面的能耗检测存在各种不足。
- WCET 最坏情况执行时间；→ 方法： IPET(implicit path enumerate technique): 隐式路径枚举技术；& an energy model 估计最坏情况能耗。 
- Xcore：adopt IPET in SRA（静态资源分析）
- Xcore：时间确定性指令执行的多线程深度嵌入式处理器。 simplier；performance ↑，consumption ↓。 可扩展成多核系统。
- novel mapping technique：energy model in ISA-level → LLVM IR level（从指令集层面到LLVM 中间代表层）。 LLVM IR 层：更多程序信息，如 类型 和 循环结构。
- 适用于任何使用LLVM 优化器的编译器。
- 比基于指令集模拟的估计（ISS-based estimations）更好。根据算法的复杂度以及最终程序的大小，表现出与ISS-based estimation 相同甚至更好的性能。  适用于迭代优化。

### Xcore 体系结构以及能量模型
#### Xcore体系结构
- Xcore：可使得硬件接口在软件层面实现的深度嵌入式处理器。
- Xcore：提供线程间的交流以及ISA层面的I/O端口控制
- 适合IoT应用：时间确定性执行、最快响应。
- 整数，没有浮点数。无需：流水线阶段的转发、推测性指令问题、分支预测
- 线程：轮询调度算法。需要4个以上的线程在线（active）
- Xcore：多核可扩展。同核或异核的线程间的交流：基于同步信道的消息传递。

#### Xcore 多线程指令集体系架构能量模型
- 在ISA层面捕捉；指令序列能耗；模型主要捕捉：硬件协调一系列的仿形测试和测量过程中产生的线程调度花费；
- 公式：总的E = 每个指令的能耗+空闲时能耗
- E = （静态能量 + 动态等待能量）* 动态等待时间 + 求和（每个指令能耗）
- 每个指令能耗 = （静态能量+某些特定指令额外的动态能量 * 扩展因子 * 指令交错平均开销）/ 活跃线程数 * 4个时钟周期
- 大于4个活跃线程，每个线程的指令发射率↓
- 一个线程的指令发射延迟=max(Nt, 4)*Tclk
- 这些计时规则被用于 基于模拟的能量估计，SRA，以及本文提到的profiling
- 这些计时规则可能无效：指令数量有限
- 关心：完全可预测的指令
#### 将能量模型用于能耗估计
- need：提取每个线程的控制流图（CFG）以及识别线程间的交替 → 获得Np(进程数) & Tidl(等待时间)
- 对于SRA，将IPET用于CFG可以提取出划分程序能耗界限的路径（4.1详细描述）
- 对任意多线程程序使用静态分析去估计能耗很难→使用2种并发模式：任务场、流水线
- 任务场、流水线：使用的线程数=整个程序的活跃线程数(常数)
- 影响能耗：FNOP。不定时发生。使用指令缓存模型来确定发生FNOPs的位置（Georgiou中详细描述）

### 将ISA代码映射到LLVM IR 以及LLVM IR的能量特征
- LLVM mapping technique：关注编译器行为、CFG结构。可转移、目标不可知。
#### mapping的格式说明
- 一堆公式。
- LLVM IR上的一条指令可以对应多条ISA指令，IR上一条指令的能耗=对应的所有ISA指令的能耗的和。
#### Xcore映射实例化和协调
- 3个阶段：
  - LLVM IR 标注：
    - 编译过程产生一些symbol协助debug，这些符号传播到中间代码层和ISA代码层。XMOS编译器的前端产生这些符号。转换成LLVM metadata 然后于附在LLVM IR上。
    - LLVM IR代码由编译器的后端转化成ISA代码，此时debug信息以DWARF标准格式存储在LLVM IR代码中。此时：源代码→n条LLVM IR指令，源代码→m条ISA指令。
    - 转换过程4条原则：消除→消除；转换→转移；合并→取first one；一条变多条→新指令采用它们原本的location。
    - 2种例外：无法对应&多变多。解决方法：取adjacent ISA 指令的debug lacation。
  - 映射和能量特征：1 LLVM IR → 多 ISA；1 ISA → 1 LLVM IR；有些LLVM IR指令会消失，但是因为1 ISA → 1 LLVM IR，所以不影响energy计算。
  - 协调以及基本块（BB）能量特征
    - 协调：LLVM IR层的SRA预测与ISA层存在误差
    - LLVM IR 层无法表示FNOP，与ISA拒绝指令一样，放在同个BB的下个ISA指令位。
    - 协调方法：[phi-nodes](https://www.cnblogs.com/ilocker/p/4897325.html)
- 缺陷：
  - case1 ：LLVM IR 的 BB1边缘的指令映射到了ISA层不同的BB
  - case2 ：ISA层BB的突然出现或消失导致CFG结构的不同
- 总结：ISA层比LLVM IR层更靠近硬件，它的能耗是可以预测的，通过从ISA到LLVM IR的映射，可以求得LLVM IR层的能耗

### 能耗预估方法
#### 静态资源分析（SRA）
- low-level analysis: 从SRA层到LLVM IR层的映射，Xcore处理器行为
- 控制流分析
- computation
#### profiling-based 能耗预估
- 计算能耗的准确值而不是bounds
- profiling阶段（记录BB执行轨迹）
  - 给每个BB一个单独的ID
  - 收集BB执行轨迹
  - 将execution counts与每个BB关联起来
- energy estimation阶段
  - 根据每个BB的能耗特征
  - BB 执行计数（execution counts）
  - 计算出程序的能耗

### 多线程程序的分析
- 平行多线程
  - 静态。
  - WCET ← IPET
- 同步，流水线
- 无需计算活跃线程数，只需记录BB执行计数

### 实验评估
- SRA结果
- 基于profiling的分析结果

### 结论
- 能耗界限评估（bounds）：IPET-based SRA（适用于单线程）
- 准确值评估：无目标的profiling technique
- 基于SRA的分析，以及WECT分析，适用于predictable architecture


