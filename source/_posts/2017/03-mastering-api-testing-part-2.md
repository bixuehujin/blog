---
title: 做好自动化 API 测试（2）- 如何高效编写测试
date: 2017-03-26
tags: 
    - 自动化测试
    - API 测试
---

在上一篇文章 「[做好自动化 API 测试（1）- 从一个好用的 Json Validator 开始][1]」 中我们说到缺少自动化 API 测试存在的三个问题：

1\. API 实现与设计不符，比如缺少字段或者字段类型不正确  
2\. API 无法形成完整闭环，缺少功能，「联调」还需要不断修改  
3\. 新加功能可能导致已有 API 出现 BUG，手动回归测试耗时且繁琐

其中第一个问题，在上一篇文章中，我阐述了通过 json-validator 解决该问题的思路与实践。

而本文将尝试阐述我对剩余两个问题的思考，旨在通过更高效自动化测试，保证高质量 API 的交付。

## 自动化测试流程

我一直有编写自动化测试的习惯，但之前一直没有达到最理想的效果，也在反思问题出现的原因，最近终于有了一些思路。其主要的原因是之前写测试都是靠感觉，感觉哪些需要测试了，就写一个测试，所有的思考都是碎片化的，没有一个系统化思考的过程，也没有通过合理的流程做保证。

确实，无论是开发还是测试，良好的流程更能确保事情更加有条不紊的推进，能够帮助我们把思维聚焦在更小范围的事务上，从而达到更理想的效果。

对于自动化测试也是一样，我们需要有一个合理的流程，把涉及的工作分为几个阶段，每个阶段关注不同的要点，避免同一时刻关注点太多带来的思维分散。

那么，就我而言，我觉得把自动化测试分为如下3个阶段是比较合理的：

1、测试用例的设计  
2、测试代码的架构  
3、测试代码的编写

在接下来的篇幅中，我将详细的介绍每个阶段需要主要关注的事情以及如何把每个阶段都做的更好~

## 测试用例的设计

无论是自动化测试还是手动的黑盒测试，测试用例都是非常重要的，测试用例的质量直接关系到整体测试的效果。因此，我们第一步需要设计测试用例，保证测试用例能够最大范围的覆盖 API 的使用场景。  

那么，如何才能设计好测试用例呢？

第一、合理的划分场景

一般而言，一个 API 或者一组 API 能满足用户的多个场景，在把握用户需求的基础上，我们需要将这些需求拆分成多个测试场景，通过合理的场景划分，一方面能够确保我们的 API 能够能够满足用户需求，另一方面聚焦每一个测试的范围，避免一个测试范围太广或者太窄，太广导致测试不到位，太窄需要写大量测试，浪费时间也不够高效。

第二、考虑边界条件

设计好主要场景的测试用例之后，API 的主要逻辑分支都应该被覆盖到了，接下来需要考虑针对边界条件的测试用例。

比较常见的边界条件有如下这些：

* 权限，没有权限的用户操作会如何，是否正确返回 403
* 非法数据提交是否正确检验，返回 400 错误
* 业务逻辑中的异常流程

当然，边界条件的判定是很复杂的，各个应用根据其业务的不同而差异很大，重要性也各不相同，开发者应该就具体业务具体确定。

第三、引入 Review 机制

一个人难免有考虑不全面的情况，这个时候可以让团队的其他成员来 Review 这些测试用例，以便更早的做出优化和调整。

当然，Review 机制适用于任何阶段，只是一般而言， Review 的时间点越早，价值越大。

  
## 测试代码的架构

你没有看错，测试代码也是需要架构的，和我们写其他代码一样，良好的架构能够使得维护变得简单，扩展变得容易。

那么，测试代码的架构需要考虑哪些方面呢？

第一、测试库的选取与扩展开发

首先我们需要选择合适的测试库来让测试的编写更简单，如果所有工具都从零手动编写，那工作量无疑是巨大的。这个时候，我们需要根据自己的需求，选择合适的库来做测试开发，就我所熟悉 PHP 领域而言，可能是 PHPUnit 配合 Guzzle，可能是 Codeception，也可能是其他的。

当然，即便使用现成的库和工具，有的需求这些库也并不能满足，还需要进行一些扩展开发工作。比如我的前一篇文章中，为了更严谨的验证 json 结构而开发的 json-validator。

第二、基础业务的封装

  
在代码的世界中，有底层代码与上层代码的区分，底层代码被上层代码所调用，做为一个软件系统的基石。

测试代码也是类似的，底层的代码用于封装基础的业务逻辑，上层代码是最外层的业务测试，其运行依赖于底层代码。

例如，我们现在需要测试「登录用户能够正常发表文章」的这一场景，其中包含「让用户处于登录状态」和「发表文章」两个操作。很显然「让用户处于登录状态」应该更底层，许多操作都依赖于它，它应该属于底层代码的范畴。

在这个例子中，我们可以把「让用户处于登录状态」这一操作封装为底层库，让其他测试代码更方便的调用。

说了这么多，总结一下其实就四个字：**分层思想**。


## 测试代码的编写

有了良好的测试用例和测试架构支撑，终于到了「编写测试代码」这一环节，这一环节有如下两个要点，帮助我们高效的编写测试代码：

1、测试用例与测试代码分离

测试的过程，其实就是检查对应的输入能够得到期望的输出的过程，为了让同一段测试代码能够适用于更多的测试用例，我们可以把测试用例和测试输出参数化，同一类别的测试用例复用同一段测试代码，从而做到测试用例与测试代码的分离。这样做可以在新添加测试用例的时候不用修改测试代码，扩展性更好。

在 PHPUnit 中，我们可以通过内置 dataProvider 功能实现此目的，我相信其他测试框架应该也有对应的支持。

2、保持测试代码的简单

测试的代码应该尽量简单，最好是申明式的，给什么数据出什么结果，让人一目了然。如果测试代码逻辑复杂，又如何保证测试代码是正确的呢？可能与测试编写的初衷相悖。

本文到此已经结束了，整体而言显得有些抽象，如有任何疑问，欢迎通过「代码写诗」微信公众号与我探讨。


[1]: https://zhuanlan.zhihu.com/p/25305186
