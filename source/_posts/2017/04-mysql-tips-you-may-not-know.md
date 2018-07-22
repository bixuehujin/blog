---
title: 关于 MySQL 你可能不知道的 SQL 使用技巧
date: 2017-08-01
tags: 
    - MySQL
---

近来处理了比较多的数据库维护工作，对 SQL 的语法也算有了更深层次的认识，也学到了很多以前没有用过的 SQL 语法技巧，这里统一整理一下，希望对读者也有所启发。

本文将主要介绍一些我认为有用的 SQL 语法和技巧，并通过适当案例说明，但案例本身做了简化处理，只希望通过案例让读者更好的理解。

## 使用 UNION | UNION ALL 语法

UNION 用于合并多个查询的结果集，我目前遇到的主要有如下两个场景用起来比较有效：

1\. 同表的复杂查询，很难通过一个 SELECT 语句搞定  
2\. 多表查询，但返回的数据一致，常见一些聚合数据统计需求

UNION 也可以加 limit、order by 子句，用于对 UNION 后的结果集进行排序和过滤。

通过 UNION 的这些特性，我们可以把原本需要编写代码才能处理的一些工作交给数据库，同时还减少 SQL 数，提高性能。

比如说，下面这个例子：

```sql
select 'product' as type, count(*) as count from `products`  
union  
select 'comment' as type, count(*) as count from `comments`  
order by count;
```

我们通过 UNION 语法同时查询了 products 表 和 comments 表的总记录数，并且按照 count 排序。

## 结合 UPDATE | DELETE 与 JOIN 

一直以来，我们得益于联合查询（SELECT + JOIN）给我们带来的便利，但无形之中确形成了思维定势（至少对于我而言是这样的），殊不知 UPDATE DELETE 也能与 JOIN 联合使用，从而简化 SQL 编写。

在使用 UPDATE | DELETE + JOIN 之前，我们可能做法要么是先查询出待删除记录的 ID 然后再根据 ID 进行删除，要么是使用 IN 子查询。前者需要写两个 SQL 语句，在程序中处理逻辑，后者有时并不能正常工作。  

就后者而言，应该有人遇到过这样的错误：

> ERROR 1093 (HY000): You can't specify target table 'xxx' for update in FROM clause  

这样的错误产生的原因是：MySQL 不支持同一个 SQL 语句尝试对同一个表进行查询和修改两个操作。

比如，删除没有评论的文章这条语句


```sql
delete from articles   
where id in (  
    select a.id from articles as a left join comments as c on a.id=c.article_id   
    where c.is is NULL  
)
```

articles 表既被查询，也被更新，将出现上面的错误。  

但是，如果 DELETE 结合 JOIN，则可以直接写出这样的 SQL 语句，简洁许多：

```sql
delete s from articles as a   
left join comments as c on a.id=c.article_id   
where c.is is NULL
```

当然，UPDATE 也是同理：

```sql
update articles as a   
left join comments as c on a.id=c.article_id   
set a.deleted=1   
where c.is is NULL
```

## CASE 语法

CASE 语法可以在 SQL 内做简单的分支判断，根据不同的条件返回不同的值。比如考虑这样的需求：

> 一个商品有多个订单，订单有已付款和未付款两个状态，现在给定一个商品列表，返回每个商品已付款和未付款订单的数量。  

这个时候我们可以通过 CASE 语句和 GROUP BY 通过一条 SQL 实现：

```sql
select   
    product_id,   
    count(  
        case is_paid  
        when 1 then 1  
        else null  
        end  
    ) as total_paid,  
    count(  
        case is_paid  
        when 0 then 1  
        else null  
        end  
    ) as total_not_paid  
from orders  
where product_id in (1, 2, 3, 4)  
group by product_id;
```

配合 ORM 库，这样的写法可以帮助我们实现 eager loading，避免 n + 1 查询。

因为这个场景比较简单，我们也可以使用 MySQL 提供的流程控制函数(Control Flow Functions) 使得该 SQL 更简洁：

```sql
select   
    product_id,   
    count(if(is_paid = 1, 1, null)) as total_paid,  
    count(if(is_paid = 0, 1, null)) as total_not_paid  
from orders  
where product_id in (1, 2, 3, 4)  
group by product_id;
```

## 使用 INSERT INTO ... SELECT 语法

通过 INSERT INTO ... SELECT 语法，我们可以把 SELECT 的结果集直接写入另一张表中，而不需要程序处理。通过这个语法，外加一些变通，我们可以很方便的实现更多的需求场景。

比如说，我们要给所有购买了某一商品的用户发放一张元价值10元的优惠券，我们可以这样写：

```sql
insert into tickets (user_id, price, expires_in)   
select   
    user_id, 10 as price, 
    '2017-09-09' as expires_in   
from orders   
where product_id=123 and is_paid=1;
```

又比如说，在选课的场景中，我们要给一批人分配一批课，假设要给1班的人分配体育课和美术课，我们可以通过该语法加 CROSS JOIN 实现：

```sql
insert into class_members (class_id, user_id, status)   
select   
    c.id as class_id,   
    u.id as user_id,   
    1 as status  
from classes as c cross join users as u  
where c.name in ('体育课', '美术课') and u.class_name='1班';
```

本文就这些了，你还知道有哪些技巧呢？
