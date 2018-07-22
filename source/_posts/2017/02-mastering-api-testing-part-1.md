---
title: 做好自动化 API 测试（1）- 从一个好用的 Json Validator 开始
date: 2017-03-05
tags: 
    - 自动化测试
    - API 测试
---

目前，前后端分离的开发模式越来越受到大家的青睐，前端与后端的职责也更加清晰，后端通过 API 提供数据，前端通过 API 获取数据，展示页面，前端有更大的发挥空间、后端也可以更加专注于数据处理。  

在这样的时代大背景下，我尝试从后端的角度出发，通过一系列文章，探讨后端开发人员应该如何交付高质量的 API 接口，减少沟通成本，降低「联调」可能出现的问题，保证代码质量以及项目按期完成。

就目前我的面试经验而言，绝大部分项目都没有引入自动化 API 测试，测试基本都是依靠开发人员的自测以及测试人员黑盒测试。本文不探讨黑盒测试，仅仅发表一点我对自动化 API 测试的想法。

首先，在我来看，缺少自动化 API 测试可能存在以下问题：

1\. API 实现与设计不符，比如缺少字段或者字段类型不正确  
2\. API 无法形成完整闭环，缺少功能，「联调」还需要不断修改  
3\. 新加功能可能导致已有 API 出现 BUG，手动回归测试耗时且繁琐

API 测试是一个比较大的主题，本文主要就「**如何让 API 的实现与设计相符合**」这一点出发，说一说我的想法及开发 Json Validator 的心路历程。

## 缘起

早在两年多以前，就在思考如何做 API Schema 的测试，保证 API 返回的结果与预期的一致。当时发现了 [Json-Schema][1] [1] 这个工具，也研究了一番，但上手成本还是略高，最终并没有实质的产出什么东西。

然后去年在开发 [Blink Framework][2] [2] 的 API 测试组件的时候，发现 Laravel 中 [验证 Json][3] [3] 的用法，很简洁优雅，同时 Codeception 也有类似的组件。

于是我就在想，有没有可能结合这两种方式的优点，自己开发一个 Json Validator，保持严谨的同时尽可能简单易用。

## 契机

正好2017新的一年，公司产品需要对外提供新的 Open API，方便与合作厂商就行业务对接。我认为这是一个实验和推行新 API 测试方式的时机，于是便利用假期的时间，开发出了 Json Validator 的原型，读者可以在 [rethinkphp/json-validator][4] [4] 查看它的最新代码。

Json Validator 通过简单的语法定义 Json 结构，然后验证给定的数据是否满足预定义的 Json 结构，比 Json-Schema 更加简单易用。

## 设计思想

除了保证设计的简洁优雅外，Json Validator 引入类型系统的概念，通过开发者自定义类型，实现类型的复用与自由组合，让 Json 的验证更方便高效，只要一个项目的基础类型定义好了，剩下基本搭积木组装即可。

## 基本用法之类型

Json Validator 默认提供7种内置类型，他们分别是: integer, double, boolean, string, number, array 和 object。开发者也可以定义自己的复合数据类型，如下的例子定义了一个 User 复合类型：
    
```php
$validator->defineType('User', [
    'name' => 'string',
    'gender' => 'string',
    'age' => '?integer',
]);
``` 

这个 User 类型有三个属性，name、gender 和 age，其中 name 和 gender 都是 string 类型，age 为 integer 类型，但允许为 null。这里我们在类型的前面加一个 ? 表示该字段可以为 null。

除了定义复合数据类型，我们也可以定义列表类型，比如定义一个 UserCollection 的类型，它是一个数组，数组的每个元素都是 User：
    
```php    
$validator->defineType('UserCollection', ['User']);
```    

要定义一个列表类型，类型的定义必须是只有一个元素的数组，数组的第一个元素即该列表类型所允许容纳的类型。

对于复杂应用场景，我们也可以使用 PHP 的 callable 来定义更灵活的类型，如下我们定义一个 timestamp 的类型，它验证给定的值必须是合法的时间字符串。
    
    
```php    
$validator->defineType('timestamp', function ($value) {
    if ((!is_string($value) &amp;&amp; !is_numeric($value)) || strtotime($value) === false) {
        return false;
    }

    $date = date_parse($value);

    return checkdate($date['month'], $date['day'], $date['year']);
});
```    

## 基本用法之验证

定义好了数据类型之后，我们就可以验证给定的数据是否符合定义，如果不符合定义，Validator 会给出错误信息。
    
    
```php    
use rethinkphp\jsv\Validator;

$validator = new Validator();
// $validator->defineType(...)  Add your custom type if necessary

$matched = $validator->matches($data, 'User');
if ($matched) {
    // Validation passed
} else {
    $errors = $validator->getErrors();
}
    
```

在一些场景下，我们希望我们定义的类型与给定的数据完全匹配，不需要多余的字段，这个时候可以使用 Json Validator 的严格模式：
    
```php
$data = [
    'name' => 'Bob',
    'gender' => 'Male',
    'age' => 19,
    'phone' =>; null, // This property is unnecessary
];
$matched = $validator->matches($data, 'User', true); // strict mode is turned on
var_dump($matched); // false is returned
```    

这个例子就会验证失败，因为 phone 字段在 User 中并没有定义。

## 未来畅想

本文介绍了我开发 Json Validator 的背景和过程以及它的简单用法，但我也在想它的使用场景可能不仅局限于此，下面是我对其未来的一些想法：

1\. 类型系统与文档生成工具相结合，从代码直接生成 API 文档的数据模型部分  
2\. 自动生成 Json-Schema 定义文件，跨语言使用更方便  
3\. 通过生成 Json-Schema 定义文件，实现自描述 RESTFul API

未完待续...

## 附

1\. [Json-Schema] [http://json-schema.org/][1]  
2\. [Blink Framework] [https://github.com/bixuehujin/blink][2]  
3\. [Laravel API Testing] [https://laravel.com/docs/5.2/testing#testing-json-apis][3]  
4\. [Json Validator] [https://github.com/rethinkphp/json-validator][4]


[1]: http://json-schema.org/
[2]: https://github.com/bixuehujin/blink
[3]: https://laravel.com/docs/5.2/testing%23testing-json-apis
[4]: https://github.com/rethinkphp/json-validator
