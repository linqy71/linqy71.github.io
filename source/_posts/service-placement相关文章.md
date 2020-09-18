---
title: service-placement相关文章
date: 2020-06-08 15:54:11
tags: service placement; Edge;
categories: 论文阅读
copyright:
---

# service-placement相关文章
> 无头苍蝇似的在各种顶会上寻找研究方向，在浏览了许多论文之后总觉得杂乱无章，收获甚少。终于发现一个较为感兴趣的方向，即service placement 服务放置问题。对这个问题甚是陌生，希望能从找到的这些文章中整理出一些思路。

## 顶会论文简介

| 论文 | 类型 | 研究内容 |
| --- | ---- | -------- |
| 1   | 算法类 | service placement + request scheduling |
| 2   | 算法类 | service placement + request routing |
| 3   | 算法类 | service placement ， user mobility |
| 4   | 算法类 | VNF placement ， budget & capacity constraint |
| 5   | 算法类 | ML job placement ，interference |
| 6   | 算法类 | SFC placement ，throughput |
| 7   | 算法类 | DAG placement |
| 8   | 算法类 | application component placement |

### INFOCOM2019

#### 1. Service Placement and Request Scheduling for Data-intensive Applications in Edge Clouds

##### 作者

Vajiheh Farhadi, Fidan Mehmeti, Ting He, Tom La Porta, Hana Khamfroush, Shiqiang Wang, Kevin S Chan 

Pennsylvania State University, University of Kentucky, IBM, ARL 


##### 摘要
移动边缘计算使无线用户可以利用云计算的功能而不会造成较大的通信延迟。为了从边缘服务于数据密集型应用程序（例如增强现实，视频分析），除了需要CPU周期和用于计算的内存外，我们还需要**用于存储服务器数据的存储资源和用于接收用户提供的数据的网络带宽**。此外，在考虑系统稳定性和运营成本的同时，需要**随时间调整数据放置以适应随时间变化的需求**。我们通过提出一个两阶段规模的框架来解决此问题，该框架在存储，通信，计算和预算约束下共同优化服务（数据和代码）放置和请求调度。通过分析各种情况的难度，我们可以充分说明问题的复杂性。通过将问题转化为集合函数优化，我们开发了多项式时间算法，该算法在某些条件下可实现恒定因子近似。大量的综合和跟踪驱动的仿真表明，该算法可实现90％的最佳性能。

##### 一些概念
-  shared resource pool 共享资源池：
    服务迁移需要考虑service performance和migration cost之间的平衡。当某些边缘节点负载较重时，考虑使用非离用户最近的边缘节点。因此，标准化的开放式边缘计算环境应运而生，使同一地理环境内的边缘云形成一个共享资源池。
- request scheduling 请求调度：
    探究用户请求应调度到哪个服务器上以使得一些参数（例如 cost、completion time）最优的问题。
    它决定是否/在何处调度每个请求，考虑通信、边缘云的计算能力以及其他限制。

- service placement 服务放置：
    它决定如何在每个边缘云的存储容量内复制和放置每个服务（包括服务器代码和数据）

##### 本文工作
我们共同考虑数据密集型应用程序的服务放置和请求调度。 与[20]相比，我们将两个决策的时间尺度分开：**服务放置发生在较大的范围（frames）以防止系统不稳定，而请求调度发生在较小的尺度（slots）以支持实时服务**。 由于服务复制/迁移，我们还施加了预算约束来控制运营成本。 这些更改可在重新配置的成本和服务请求的性能之间实现可控的权衡，同时在根本的优化问题中引发关键的更改。


#### 2. Joint Service Placement and Request Routing in Multi-cell Mobile Edge Computing Networks

##### 作者

Konstantinos Poularakis, Jaime Llorca, Antonia M. Tulino, Ian Taylor, and Leandros Tassiulas 

Department of Electrical Engineering and Institute for Network Science, Yale University, USA 
Nokia Bell Labs, USA 
Department of Electrical Engineering and Information Technologies, University of Naples Federico II, Italy 
School of Computer Science and Informatics, Cardiff University, UK


##### 摘要
诸如增强现实，网络游戏和自动驾驶等创新移动服务的激增，刺激了人们对低延迟访问计算资源的需求日益增长，而这些延迟只能通过现有的集中式云系统无法满足。通过支持在最终用户附近的网络外围执行计算任务，移动边缘计算（MEC）有望成为满足低延迟服务需求的有效解决方案。尽管许多最新研究已经解决了确定服务任务的执行以及将用户请求路由到相应边缘服务器的问题，但重点主要放在高效利用计算资源上，而忽略了这样一个事实：**需要存储数据以启用服务执行，并且许多新兴服务表现出不对称的带宽要求**。为了填补这一空白，我们研究了在具有多维（存储-计算-通信）约束的支持MEC的mmulti-cell网络中，服务放置和请求路由的联合优化。我们表明，该问题概括了文献中的几个问题，并提出了一种使用随机舍入实现接近最佳性能的算法。评估结果表明，我们的方法可以有效利用可用资源，以最大程度地减少低延迟边缘云服务器所服务的请求数量。

##### 一些概念
- 面对的挑战：
  - allocation of computation resources
  - non-trival amout of data to be prestored at BS
  - communication requirements of services
(不对称的双向通信，占用无线基站上行链路和下行链路带宽容量的不同部分)


#### 3. Adaptive User-managed Service Placement for Mobile Edge Computing: An Online Learning Approach

##### 作者

Tao Ouyang, Rui Li, Xu Chen, Zhi Zhou, Xin Tang 

School of Data and Computer Science, Sun Yat-sen University, Guangzhou, China

##### 摘要
设想为云扩展的移动边缘计算（MEC）将云资源从网络核心推送到网络边缘，从而满足许多新兴的计算密集型移动应用程序的严格服务要求。现有的许多工作都集中在研究系统范围内的MEC服务放置问题，个性化服务性能优化而受到的关注较少。因此，在本文中，我们提出了一种新颖的**自适应用户管理**的服务放置机制，该机制可以**联合优化用户的感知潜伏期和服务迁移成本，并通过用户偏好进行加权**。为了克服未来信息的不可用和未知的系统动力学问题，我们将动态服务放置问题公式化为a contextual multiarmed bandit (MAB) 问题，然后提出一种基于汤普森采样的在线学习算法来探索动态MEC环境，进一步协助用户做出自适应服务放置决策。严格的理论分析和广泛的评估证明了所提出的自适应用户管理的服务放置机制的优越性能。

#### 4. Joint Placement and Allocation of Virtual Network Functions with Budget and Capacity Constraints

##### 作者

Gamal Sallam and Bo Ji 

Department of Computer and Information Sciences, Temple University, Philadelphia, PA, U

##### 摘要
随着网络功能虚拟化（NFV）的出现，传统上可以在专用硬件上运行的网络服务现在可以使用托管在通用商品硬件上的虚拟网络功能（VNF）来实现。这种新的网络范例为网络服务提供商（ISP）提供了极大的灵活性，可以有效地操作其网络（收集网络统计信息，执行管理策略等）。但是，引入NFV要求进行投资以将VNF部署在某些网络节点（称为VNF节点）上，这必须考虑实际的限制，例如部署预算和VNF节点容量。为此，重要的是设计一个**联合的VNF节点放置和容量分配算法，该算法可以在尊重此类实际约束的同时，最大化VNF节点完全处理的网络流量总量**。与通常忽略预算约束或能力约束的大多数先前工作相反，我们明确考虑了两者。我们证明，考虑这些约束会带来一些新的挑战。具体而言，我们证明了所研究的问题不仅是NP难的，而且是非亚模的。为了解决这些挑战，我们引入了一种新颖的松弛方法，以使松弛放置子问题的目标函数变为亚模。利用这种有用的子模性质，我们提出了两种算法，它们分别对原始的非松弛问题实现了1/2（1-1 / e）和1/3（1-1 / e）的近似比。最后，我们使用跟踪驱动的仿真和基于综合网络设置的仿真，通过广泛的评估来证实所提出算法的有效性。


#### 5. Deep Learning-based Job Placement in Distributed Machine Learning Clusters

##### 作者

Yixin Bao, Yanghua Peng, Chuan Wu 

Department of Computer Science, The University of Hong Kong

##### 摘要
生产机器学习（ML）集群通常托管各种分布式ML工作负载，例如语音识别，机器翻译。虽然作业之间的服务器共享可以提高资源利用率，但是位于同一地点的ML作业之间的干扰会导致性能严重下降。现有的群集调度程序（例如，Mesos）在其工作位置上是无干扰的，从而导致资源效率不佳。文献中已经研究了具有干扰意识的工作位置，但是使用详细的工作量分析和干扰建模进行了处理，这不是一般的解决方案。本文介绍了Harmony，这是一种深度学习驱动的ML群集调度程序，它**以最小化干扰和最大化性能（即训练完成时间）的方式放置训练作业**。Harmony基于精心设计的深度强化学习（DRL）框架，并补充了奖励建模。 DRL采用最先进的技术来稳定训练并提高收敛性，其中包括演员批评算法，作业意识动作空间探索和体验重播。鉴于通常缺乏对应于不同安置决策的奖励样本，我们建立了一个辅助奖励预测模型，该模型使用历史样本进行训练，并用于为看不见的安置产生奖励。在由6个GPU服务器组成的Kubernetes集群中使用实际ML工作负载进行的实验表明，就平均作业完成时间而言，Harmony的性能比典型的调度程序高25％。

##### 一些概念
- 本文所描述的interference：
  创建在同一server上的不同类型的ML jobs，例如CPU-intensive, I/O intensive, network bandwith-intensive, 由于底层资源的共享而导致的干扰。

#### 6. Octans: Optimal Placement of Service Function Chains in Many-Core Systems

##### 作者

Zhilong Zheng, Jun Bi, Heng Yu, Haiping Wang, Chen Sun, Hongxin Hu, Jianping Wu

Institute for Network Sciences and Cyberspace, Tsinghua University 
Department of Computer Science, Tsinghua University
Beijing National Research Center for Information Science and Technology
School of Computing, Clemson University


##### 摘要
网络功能虚拟化（NFV）可以通过在具有多个核心的商用服务器上运行服务功能链（SFC），来提供服务交付灵活性并降低总体成本。用于将SFC放置在一台服务器中的现有解决方案将所有CPU内核视为相等，并将隔离的CPU内核分配给不同的网络功能（NF）。但是，高级服务器通常采用非统一内存访问（NUMA）架构来提高多核系统的可伸缩性。 CPU内核分为多个节点，由于**跨节点内存访问和节点内资源争用而导致性能瓶颈**。我们的评估表明，与最佳放置解决方案相比，随机选择将NF放置在SFC中的核心可能会降低39.2％的吞吐量。在本文中，我们提出了Octan，它是NFV协调器，可在多核系统中实现所有SFC的最大总吞吐量。 Octans首先将优化问题表述为非线性整数规划（NLIP）模型。然后，我们确定解决问题的关键因素是评估同一SFC或不同SFC中其他NF导致的NF吞吐量下降，即性能下降指数，并基于系统级性能指标提出正式而精确的预测模型。最后，我们提出了一种有效的启发式算法来快速找到接近最佳的放置解决方案。我们已经实现了Octans的原型。广泛的评估表明，与两种最先进的放置机制相比，Octan显着提高了总吞吐量26.7％〜51.8％，SFC性能的预测误差非常低（平均偏差为2.6％）。而且，Octans可以快速找到具有最佳间隙（1.2％〜3.5％）的接近最佳的放置解决方案。

##### 一些概念
- NFV 网络功能虚拟化：对传统在专有硬件上运行的网络服务（例如路由器、防火墙和负载平衡器）进行虚拟化的方法。这些服务被打包为商用硬件上的虚拟机（VM），这样服务提供商便可以在标准服务器（而非专有服务器）上运行其网络。
- SFC 服务功能链：网络包以network functions sequence的形式被处理，从而形成SFC。

##### 挑战
现有解决方案将所有CPU内核视为相等，将隔离的CPU内核分配给不同的网络功能。
- 随机选择支持SFC的CPU内核可能会严重损害吞吐量：（一个节点由多个CPU cores组成）
  - 跨节点内存访问引起的吞吐量瓶颈
  - 节点内资源争用

##### 本文工作
- 用非线性整数编程模型解决问题
- 为了评估由于资源争用和链接而导致的性能下降指标，我们引入了一种正式的方法来查找性能指标，并提出了一种用于性能下降预测的准确模型。

### SEC 
#### 7. Linearize, Predict and Place: Minimizing the Makespan for Edge-based Stream Processing of Directed Acyclic Graphs （2019）

##### 作者

Shweta Khare, Hongyang Sun, Julien Gascon-Samson, Kaiwen Zhang, Aniruddha Gokhale, Yogesh Barve, Anirban Bhattacharjee, and Xenofon Koutsoukos.

##### 摘要
在诸如智能电网之类的网络物理系统中发现的许多物联网应用，都必须对关键事件（例如供需不匹配）采取控制措施，这需要对流数据进行低延迟处理以进行快速事件检测和异常纠正。这些流应用程序通常采用有向非循环图（DAG）的形式，其中顶点表示运算符，边表示这些运算符之间的数据流。边缘计算由于能够在数据源附近提供低延迟处理的能力而作为一种很容易满足延迟关键的物联网应用程序需求的方法，最近引起了广泛的关注。为了获得边缘计算的好处，这些应用程序的组成运营商必须以一种合理的方式放置，通过**权衡运营商之间的通信成本与由于运营商在同一资源受限的边缘上的同一地点而产生的干扰成本**。为了解决这些挑战并大幅**简化任意大小和拓扑的DAG的放置问题**，我们提出了一种算法，该算法首先**将任意流处理DAG转换为一组近似的线性链**。随后，用于共处一地的线性链的数据驱动的等待时间预测模型用于通知操作员的位置，以使定义为DAG中所有路径的**最大等待时间的延展时间最小化**。我们使用Beagle Bone集群上代表边缘计算环境的各种DAG放置方案，通过经验评估我们的算法。

(DAG placement)

#### 8. Efficient placement of multi-component applications in edge computing systems （2017）

##### 作者

Tayebeh Bahreini and Daniel Grosu

Dept. of Computer Science, Wayne State University

##### 摘要
移动边缘计算（MEC）是一种新的范例，已被引入以解决移动云计算技术的低效率问题。 MEC背后的关键思想是通过将应用程序的计算转发到网络边缘而不是云数据中心来增强移动设备的功能。 MEC的主要挑战之一是确定移动应用程序组件在边缘服务器上的有效放置，从而最大程度地减少运行应用程序时产生的成本。在本文中，我们通过设计一种有效的启发式在线算法来解决边缘计算中的多组件应用程序放置问题。我们还提出了考虑到**用户位置和网络功能的动态特性**的多组件应用程序放置问题的混合整数线性规划公式。我们进行了广泛的实验，以评估该算法的性能。实验结果表明，该算法执行时间极短，并获得了最佳解。

##### 一些概念
- challenging issues in MEC：用户移动性导致的应用组件放置问题
- application graphs: 节点表示应用组件
- application placement problem：可以视为两种图之间的映射问题

##### 本文工作
- 将多组件应用程序放置问题的onine版本公式化为混合整数线性程序（MILP）
- 基于简单算法技术（例如匹配和局部搜索）的算法，并避免使用复杂的方法（例如基于马尔可夫决策过程的方法）
- 算法设计的主要思想是确定应用程序组件与边缘/核心服务器之间的最佳匹配，而无需考虑组件之间的通信需求，然后考虑组件之间的通信需求并使用本地搜索过程改善解决方案

### 总结

- service placement问题是指考虑如何在每个边缘云的存储容量内复制和放置每个服务的问题。

- 挑战：
  - 通常考虑优化服务器在storage、computation、communication方面的资源限制，以及用户的移动性
  - 由于在同一资源受限的edge server上部署不同的service而导致的问题

- 最新研究
  - 与请求调度、网络功能虚拟化（NFV）结合进行研究