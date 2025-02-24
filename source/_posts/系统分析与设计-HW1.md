---
title: 系统分析与设计-HW1
date: 2019-03-14 21:38:28
tags: 系统分析与设计
categories: 课程作业
copyright: true
---


## 软件工程的定义

> Software engineering is “(1) the application of a systematic, disciplined, quantifiable approach to the development, operation, and maintenance of software, that is, the application of engineering to software,” and “(2) the study of approaches as in (1).” –– IEEE Standard 610.12
    
软件工程是：
(1)将系统的、有条理的、可量化的方法应用于软件的开发、操作和维护，即工程在软件上的应用
(2) 对(1)中所提到的方法的研究。

## 解释导致 software crisis 本质原因、表现，述说克服软件危机的方法

### 软件危机本质原因
> The software crisis was due to the rapid increases in computer power and the complexity of the problems that could not be tackled. With the increase in the complexity of the software, many software problems arose because existing methods were insufficient. -- Wikipedia

软件危机是由于计算机能力的迅速增长和无法解决的问题的复杂性造成的。随着软件复杂度的增加，由于现有方法的不足，出现了很多软件问题。

### 软件危机的表现
> Projects running over-budget
> Projects running over-time
> Software was very inefficient
> Software was of low quality
> Software often did not meet requirements
> Projects were unmanageable and code difficult to maintain
> Software was never delivered 

- 项目运行超预算
- 项目运行时间过长
- 软件效率很低
- 软件质量很差
- 软件通常不符合要求
- 项目难以管理，代码难以维护
- 软件从未交付过

### 克服软件危机的方法
> Various processes and methodologies have been developed over the last few decades to improve software quality management such as procedural programming and object-oriented programming. However software projects that are large, complicated, poorly specified, and involve unfamiliar aspects, are still vulnerable to large, unanticipated problems.

计算能力按摩尔定律发展，软件处理的问题也越来越广、越来约复杂，因此软件生产不只是编程技术问题，需要有系统化、与时俱进的软件工程方法，才能规避 software crisis ！

软件危机的产生是因为计算机能力的提高超过了程序员利用这些能力的能力。在过去的几十年中，已经出现了各种过程和方法来改进软件质量管理，例如过程编程和面向对象的编程。然而，大型，复杂，指定不当以及涉及不熟悉方面的软件项目仍然容易受到大的，未预料到的问题的影响。


## 软件生命周期
> 软件生命周期一般包括可行性分析与计划、需求分析、设计(概要 设计和详细设计)、编码实现、测试、运行与维护等活动。这些活动应当以适当的方式分配到不同的阶段去完成

软件生命周期（Life Cycle）：在时间维度，对软件项目任务进行划分，又成为软件开发过程。常见有瀑布模型、螺旋模型、敏捷的模型等。

**瀑布模型**
![](https://i.loli.net/2019/03/14/5c8a63770e51e.png)


## SWEBoK 的 15 个知识域

> Knowledge Areas Characterizing the Practice of Software Engineering（表征软件工程实践的知识领域）

1. 软件需求 Software Requirements
软件要求知识域关注软件需求的启发，协商，分析，规范和验证。软件需求表达了对软件产品的需求和限制，这些需求和约束有助于解决一些现实问题。

2. 软件设计 Software Design
设计被定义为定义体系结构、组件、接口和系统或组件的其他特性以及该过程的结果的过程（IEEE 1991）。软件设计知识域涵盖了设计过程和最终产品。

3. 软件构建 Software Construction
软件构建是指通过结合详细设计，编码，单元测试，集成测试，调试和验证来详细创建工作软件。软件构建知识域包括与满足其要求和设计约束的软件程序开发相关的主题。

4. 软件测试 Software Testing
测试是一项旨在评估产品质量并通过识别缺陷来改进产品质量的活动。软件测试涉及在有限的测试用例集上针对预期行为动态验证程序的行为。这些测试用例是从（通常非常大的）执行域中选择的。

5. 软件维护 Software Maintenance
软件维护包括增强现有功能，调整软件以在新的和修改的操作环境中运行，以及纠正缺陷。这些类别称为完善，自适应和纠正性软件维护。

6. 软件配置管理 Software Configuration Management
软件配置管理（SCM）是在不同时间点识别系统配置的规则，用于系统地控制配置的改变，以及在整个软件生命周期中维持配置的完整性和可追溯性。

7. 软件工程管理 Software Engineering Management
软件工程管理涉及规划，协调，测量，报告和控制项目或程序，以确保软件的开发和维护是系统化的，规范化的和量化的。

8. 软件工程过程 Software Engineering Process
软件工程知识域关注软件生命周期过程的定义，实施，评估，测量，管理和改进。

9. 软件工程模型和方法 Software Engineering Models and Methods
软件工程模型和方法知识域解决了涵盖多个生命周期阶段的方法; 其他知识域涵盖特定生命周期阶段的特定方法。

10. 软件质量 Software Quality
软件质量是许多SWEBOK V3 KAs中普遍存在的软件生命周期问题。

11.  软件工程专业实践 Software Engineering Professional Practice
软件工程专业实践关注软件工程师必须具备的专业，负责和道德的软件工程知识，技能和态度。

> Knowledge Areas Characterizing the Educational Requirements of Software Engineering(表征软件工程教育要求的知识领域)

12. 软件工程经济学 Software Engineering Economics
软件工程经济学知识域关注的是在业务环境中做出决策，以使技术决策与组织的业务目标保持一致。

13. 计算基础 Computing Foundations
计算基础知识域涵盖了提供软件工程实践所需的计算背景的基础主题。

14. 数学基础 Mathematical Foundations
数学基础知识域涵盖了提供软件工程实践所必需的数学背景的基础主题。

15.  工程基础 Engineering Foundations
工程基础知识域涵盖了提供软件工程实践所必需的工程背景的基础主题。

## 简单解释 CMMI 的五个级别
- Level 1 - Initial：无序，自发生产模式。
- Level 2 - Managed: 管理制度化，建立了基本的管理制度和规程，管理工作有章可循.
- Level 3 - Defined: 开发过程，包括技术工作和管理工作，均已实现标准化、文档化。
- Level 4 - Quantitatively Managed: 产品和过程已建立了定量的质量目标。可预测过程和产品质量趋势。
- Level 5 - Optimizing：可集中精力改进过程，采用新技术、新方法。


## 用自己语言简述 SWEBok 或 CMMI

软件能力成熟度模型集成（CMMI）是一个过程级别的改进培训和评估计划。其目的是帮助软件企业对软件工程过程进行管理和改进，增强开发与改进能力，从而能按时地、不超预算地开发出高质量的软件。其所依据的想法是：只要集中精力持续努力去建立有效的软件工程过程的基础结构，不断进行管理的实践和过程的改进，就可以克服软件开发中的困难。CMMI为改进一个组织的各种过程提供了一个单一的集成化框架，新的集成模型框架消除了各个模型的不一致性，减少了模型间的重复，增加透明度和理解，建立了一个自动的、可扩展的框架。因而能够从总体上改进组织的质量和效率。CMMI主要关注点就是成本效益、明确重点、过程集中和灵活性四个方面。




## 参考
- [https://en.wikipedia.org/wiki/Software_crisis](https://en.wikipedia.org/wiki/Software_crisis)
- [https://en.wikipedia.org/wiki/Capability_Maturity_Model_Integration](https://en.wikipedia.org/wiki/Capability_Maturity_Model_Integration)
- [https://www.sebokwiki.org/wiki/An_Overview_of_the_SWEBOK_Guide](https://www.sebokwiki.org/wiki/An_Overview_of_the_SWEBOK_Guide)
- [https://baike.baidu.com/item/CMMI](https://baike.baidu.com/item/CMMI)


