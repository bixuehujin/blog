---
title: 谈一谈自动化测试的统筹规划
date: 2017-12-16
tags: 
    - 自动化测试
---

谈到自动化测试，大家都会想到单元测试、功能测试等词汇，笔者所在团队也有这样的实践，取得了一定的效果，但却没有让自动化测试发挥最大的价值，一直在思考这背后的原因，有没有办法做的更好，是以形成本文，供读者参考~

## 背景

回顾以前自动化测试编写的经历，主要是以开发者自驱动的方式进行，测试的编写随心而动，没有规划，也没有章法，这样就面临如下的一些问题：

1、测试用例设计不到位，覆盖不全，或者不够高效  
2、因为工期原因压缩自动化测试时间，自动化测试名存实亡   
3、自动化基础设施不完善，某些测试编写成本比较高   
4、缺少完善的测试数据支持，导致测试效果大打折扣

这么多的问题，其实总结起来本质就是一个原因，缺少自动化测试的统筹规划，没有将自动化测试纳入到研发体系中。

## 自动化测试的统筹规划

为了解决这些问题，让自动化测试真正的发挥其最大价值，解放生产力，提高研发效率，让我们从重复的手动测试中解放出来，我们首先要做的就是对自动化测试进行统筹规划，将自动化测试的意义提升一个等级，让每个人都认识到他的价值与意义，包括产品，研发，测试以及高层管理人员。

自动化测试的统筹规划应该是自上而下的，由多个层次构成一整套体系，这个体系应对包含框架，数据、用例和代码四个部分，每个部分有其自己的职责，四者相互协同形成完整的测试体系和闭环。

下面简单介绍一下这套体系。

## 测试体系之测试框架

这里的测试框架是泛指，也可以叫测试基础设施，它存在的目的是为了服务测试相关人员，让他们更加高效便捷的编写测试，执行测试，从而提高效率。可能涉及如下一些工作：

1、选择合适的测试框架，TDD 还是 BDD   
2、环境初始化机制，比如E2E测试如何搭建快速搭各种环境，以及对应数据的初始化  
3、辅助测试工具的开发

做好测试框架的搭建，需要有相关的测试开发人员（未必要有这个职位，但需要这个角色）进行，一个好用完善的测试框架，直接关系到最终测试体验和效果

## 测试体系之测试数据

自动化测试离不开数据的支持，为了测试顺利进行，我们需要准备一套甚至多套测试数据，以便在不同的场景下使用。同时，这些数据不能是杂乱无章的，它应该是有序的，且能够覆盖尽可能多的使用场景，并且需要随着业务的发展不断迭代维护。

假如用户有多个状态，每个状态对应了不同的用户行为，这些用户的测试数据应该同时包含不同状态的用户，以便测试用户在不同状态的行为是否符合预期，当然这只是一个很简单的例子，实际场景会复杂很多。

## 测试体系之测试用例

有了测试框架和测试数据的支撑，就需要我们开始设计测试用例了，测试用例的设计最好是独立于开发环节之外，这样才能更专注的进行测试用例的设计，对于有手动测试的团队，测试用例在自动化测试和手动测试也需要统筹考虑，以便设计最高效的测试用例，平衡测试效果与成本。

同理，如果存在多个层次的测试，比如单元测试、功能测试、E2E测试，他们的测试也要统筹考虑，在最合适的地方做最合适的事情。

## 测试体系之测试代码

有了前面的规划和准备，测试代码的编写应该是水到渠成的事了，有开发者编写对应的测试代码即可。当然，在这个阶段如果遇到测试代码编写的困难，比如某个基础数据很难在测试中复现，可能需要回到 测试框架 中，反过来提升测试框架的能力，形成一个完整的闭环。

## 结束语

本文就是这些了，后续找机会再分享一些具体操作层面的细节，欢迎关注「代码写诗」微信公众号或本专栏获得更新。