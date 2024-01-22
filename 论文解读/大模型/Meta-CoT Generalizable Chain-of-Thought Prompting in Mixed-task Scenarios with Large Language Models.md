# Meta-CoT: Generalizable Chain-of-Thought Prompting in Mixed-task Scenarios with Large Language Models

论文链接：https://arxiv.org/abs/2310.06692

GitHub：https://github.com/Anni-Zou/Meta-CoT

## 1.论文背景

大型语言模型 (LLM) 通过利用思维链 (CoT) 提示展现了卓越的推理能力。然而，当前的 CoT 方法要么简单地采用通用提示，例如“let's think step by step”，要么严重依赖手工制作的特定于任务的演示(few-shot)来获得更好的性能，难以在泛化性和性能之间取得一个较好的平衡。

现有研究通常假设提供给模型的问题类型是已知的，并对来自同一数据集的问题进行评估。然而，更现实的情况是在于混合任务场景，其中输入问题的类型是未知的，它们以任意的方式出现。这样的混合任务场景目前还没有什么好的解决办法。

## 2. 论文提出观点

论文提出了 Meta-CoT，这是一种在输入问题类型未知的混合任务场景中的通用 CoT 提示方法。Meta-CoT 首先根据输入问题对场景进行分类，然后自动从相应的数据池中构建不同的演示，最后进行答案的推理。

## 3.论文方法

![](https://github.com/zzysos/LLMsStudy/blob/master/论文解读/pic/Meta-CoT.png)

**1.场景识别**

前置信息：

论文中使用到10个任务集构建mixed data pool，涵盖了算术问题，常识推理以及符号推理三大类别，答案的形式分为short-answer, multiple-choice,  yes-or-no三大类。作者进一步将10个任务集以 <Category, Form> 的形式分为6个大类，称为6个场景。

这样做的原因是经过多次实验，作者发现以 <Category, Form> 的形式分类，LLM能够准确地将一个输入问题分到对应的类别。

例如：给到LLM这6个类别 ：<Arithmetic, MCQ> ，<Symbolic, SAQ> ......

再给出一个输入问题，LLM能准确地得出这个输入问题属于上面6个类中的哪一个。

![](https://github.com/zzysos/LLMsStudy/blob/master/论文解读/pic/分类方法比较.png)

方法流程：

注：mixed data pool中会为每个场景事先准备好少量数据。

从6个场景中分别随机抽取一个问题与其对应的场景组成 <问题，场景>，给到LLM的上下文中，LLM根据这些演示提示最后识别出当前输入问题的类别。

**2.演示选择**

将第一步得到的场景中的所有问题拿出来做一次k-means聚类，对于每个簇，按照与簇中心的距离升序对问题进行排序。遍历排序的问题并将 Zero-Shot-CoT 应用于当前问题，以获得一个推理过程和一个预测答案。遍历的过程是会执行过滤操作的，即只要得到了一组符合要求的推理过程和预测结果，遍历就会终止。所以对于每个簇的话只会得到一个演示结果。

过滤原则：遵循简单的启发式方法，问题token数不超过60，且推理步骤不超过5步。

最终得到k个 <问题，推理过程，预测答案> 的演示。添加到LLM的上下文中。

**3.得出答案**

根据第二步的演示作为提示加入上下文，LLM最终得出当前问题的答案。这次执行的最后的结果会以 <问题，推理过程，预测答案> 的形式更新mixed data pool。 

## 4.实验分析

![](https://github.com/zzysos/LLMsStudy/blob/master/论文解读/pic/10个分布内数据集实验结果.png)

![](https://github.com/zzysos/LLMsStudy/blob/master/论文解读/pic/5个分布外数据集实验结果.png)

## 5.结论分析

这篇论文提出了一种比较新的应用场景，也就是输入问题类型未知的混合任务场景。论文中通过根据输入数据执行场景识别，然后自动构建 ICL 的相应演示的方法，在追求通用性的同时也提高了性能。