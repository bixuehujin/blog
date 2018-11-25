---
title: 数据仓库思考与实践（一） - 为什么说我们应该学习数据仓库
date: 2017-04-17
tags: 
    - 数据仓库
---

以前简单了解过诸如 Hadoop，Spark 等与大数据相关的技术，但总觉得少了什么，不知道应该从哪里入手，该如何应用于实践，直到前一段时间，偶然发现了「The Data Warehouse Toolkit, 3rd Edition」这本书，阅读后甚为畅快，很多萦绕在脑海中的问题迎刃而解，还因此得到了一些灵感得以解决目前业务中遇到的难题。  

在这样的背景下，打算结合自己的经历，陆陆续续写一些关于数据仓库的系列文章，记录我在数据仓库方面的实践与思考。本文作为系列文章的开篇，主要从宏观层面说一说我对数据仓库的理解，如有纰漏，欢迎不吝赐教。以下是本文的主要内容：

1. 什么是数据仓库
2. 数据仓库与大数据的关系
3. 事务型系统与数据仓库系统  
4. 构建数据仓库的流程
5. 为什么说我们应该学一点数据仓库知识  

## 什么是数据仓库

数据仓库并不是什么新东西，它已经存在了几十年了，最早可以追溯到 60 年代，然后由数据仓库之父 Bill Inmon 与 80 年代提出被广泛接受的数据仓库定义：

> 数据仓库是一个用于支持管理决策的面向主题的（Subject Oriented）、相对稳定的（Non-Volatile）、集成的（Integrate）、反映历史变化（Time Variant）的数据集合。

这句话核心表达的是数据仓库的目的是为了支持管理决策。

在没有数据仓库的支撑下，通常管理决策都更多的凭借管理者的感觉或者经验，有很大的可能导致决策失误而造成重大损失。

有了数据仓库后，我们就可以通过数据进行更合理的分析，结合历史记录，多维分析，帮助我们做更准确的判断，从而帮助企业抓住机会，降低损失，达到收益最大化。近来比较火数据驱动的概念也离不开这个，通过充分挖掘数据的价值，让数据帮我们决策。

## 数据仓库与大数据的关系

数据仓库诞生的时候还没有大数据的概念。所以本质上来说，数据仓库与大数据没有必然联系，数据仓库的存在并不依赖大数据。

不过随着互联网的发展，人们产生的数据越来越多，TB PG级的数据越来越常见，传统的技术无法处理如此大量的数据，所以数据仓库也需要引入大数据技术，作为数据仓库的一环，解决大数量下的存储与分析问题，让数据仓库在大数据量下仍然高效。

就个人而言，我觉得数据仓库更多的是一套理念和架构方法，与数据量无关，而大数据技术则是为了这套理念和架构而服务。

## 事务型系统与数据仓库系统

事务型系统也就是 OLTP 应用系统，它是我们接触最多的系统，这类系统的设计初衷就是为了满足快速的事务处理，比如购物，买票，秒杀等。系统处理的每一个请求，一般仅涉及一个或者多个资源，能在很快的时间内完成请求处理并返回结果。

但一旦涉及到复杂的聚合统计查询，这类系统就比较难以处理了，比如要查询某一些类型的用户过去三个月购买最多的商品，因为同一时间需要查询大量数据，OLTP 系统并不擅长处理这类需求。

但数据仓库就不一样了，数据仓库的数据是按主题进行组织的，存储上做了专门的优化，每个主题的数据都是为了这个主题的分析而服务，它生来就是了进行复杂的聚合和统计，所以数据仓库系统能在更短的时间内响应各种复杂的统计查询。

> 这里所谓的主题可以是销售额，可以是用户行为。通过对销售额的分析我们可以知道那些商品在什么时候对那些人销售量最高，通过对用户行为的分析我们可以知道那些人对那些功能最为常用，粘度最高，诸如此类。

## 构建数据仓库的流程

数据仓库的构建是有一些最佳流程和实践的，正确的流程能让数据仓库的构建事半功倍，降低成本，构建一个数据仓库系统主要涉及如下几个步骤。

**收集商业需求（Business Requirements）**

前面我们说到，数据仓库的目的是为了支持管理决策，所以我们在构建数据仓库的时候首先需要考虑的做的是收集商业需求，深入理解企业的各个业务流程（Business Process），抓住核心需求，为数据仓库的建立提供指导方向，这样才能反过来为业务提供更好的决策数据支撑，让数据仓库价值的最大化。

**进行数据建模**

深入理解业务之后就可以着手建立数据模型了，因为一个企业可能有多个产品线，而一个产品一般会涉及多个业务流程。所以建立数据模型首先需要选择一个业务流程，然后针对这个流程选取合适数据、合理的纬度进行建模。建模的合理与否，直接影响到将来进行数据分析的效果以及系统的扩展性。

因为分析需求总是多种多样、猝不及防的，只有建立合适的数据模型，才能更加从容的面对未来多变的分析需求，建模的重要性可见一斑。

**数据加工与处理**

完成数据建模之后就可以进行数据的加工和处理了，数据仓库里我们称之为ETL（Extraction, Transformation, and Loading）流程，主要的工作是从各个业务系统或者日志系统提取数据，并做必要的转换，保证数据的正确性，然后加载到数据仓库系统中，以便之后的查询和分析。

这个过程根据业务以及数据量的不同，复杂度天差地别。对于小系统来说，可能几个 SQL 语句，一个简单的脚本就能搞定，然后对于大型系统（大数据量），我们可能需要用到 Hadoop、Spark 之类的大数据框架才能做好。

**数据查询与分析**

前面的流程完成之后就可以进行数据查询于分析了，这是最终展现出来的阶段，也是产生价值阶段。

数据分析可以通过数据仓库系统提供的查询语言完成，也可以基于这些查询语言编写 商业智能(Business Intelligence) 应用，提供可视化的报表，从而为管理决策提供支撑。

## 为什么说我们应该学一点数据仓库知识

了解到数据仓库是因为业务中面临数据于统计分析的难题，于是尝试寻找更好的解决方案，九牛二虎之力之后终于找到一些线索，然后顺藤摸瓜了解到数据仓库并开始学习，并深深被其思想所折服，不能自拔。

数据仓库通过对数据进行建模和重新组织，使得数据能够更容易的被统计和分析，为数据统计分析提供了一个新的思路与理念。即便我们不用数据仓库做前文提到的更宏观的目的，数据仓库也能为我们提供一个不一样的思路，可以用在系统设计过程中，让系统更简单高效。

所以，学习数据仓库，未必一定要构建一个数据仓库系统，可能还可以为业务系统的设计提供不一样的思路和灵感。所以，也许你也可以来看一看、学一学～

本文就这些了，泛泛而谈，下一篇文章会写一写数据仓库数据建模相关的东西。