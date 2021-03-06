---
title: "INSERT 语句"
nav-parent_id: sql
nav-pos: 5
---
<!--
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

  http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
-->

* This will be replaced by the TOC
{:toc}

INSERT 语句用来向表中添加行。

## 执行 INSERT 语句

可以使用 `TableEnvironment` 中的 `sqlUpdate()` 方法执行 INSERT 语句，也可以在 [SQL CLI]({{ site.baseurl }}/zh/dev/table/sqlClient.html) 中执行 INSERT 语句。`sqlUpdate()` 方法执行 INSERT 语句时时懒执行的，只有当`TableEnvironment.execute(jobName)`被调用时才会被执行。

以下的例子展示了如何在 `TableEnvironment` 和  SQL CLI 中执行一个 INSERT 语句。

<div class="codetabs" markdown="1">
<div data-lang="java" markdown="1">
{% highlight java %}
EnvironmentSettings settings = EnvironmentSettings.newInstance()...
TableEnvironment tEnv = TableEnvironment.create(settings);

// 注册一个 "Orders" 源表，和 "RubberOrders" 结果表
tEnv.sqlUpdate("CREATE TABLE Orders (`user` BIGINT, product VARCHAR, amount INT) WITH (...)");
tEnv.sqlUpdate("CREATE TABLE RubberOrders(product VARCHAR, amount INT) WITH (...)");

// 运行一个 INSERT 语句，将源表的数据输出到结果表中
tEnv.sqlUpdate(
  "INSERT INTO RubberOrders SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'");
{% endhighlight %}
</div>

<div data-lang="scala" markdown="1">
{% highlight scala %}
val settings = EnvironmentSettings.newInstance()...
val tEnv = TableEnvironment.create(settings)

// 注册一个 "Orders" 源表，和 "RubberOrders" 结果表
tEnv.sqlUpdate("CREATE TABLE Orders (`user` BIGINT, product STRING, amount INT) WITH (...)")
tEnv.sqlUpdate("CREATE TABLE RubberOrders(product STRING, amount INT) WITH (...)")

// 运行一个 INSERT 语句，将源表的数据输出到结果表中
tEnv.sqlUpdate(
  "INSERT INTO RubberOrders SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'")
{% endhighlight %}
</div>

<div data-lang="python" markdown="1">
{% highlight python %}
settings = EnvironmentSettings.newInstance()...
table_env = TableEnvironment.create(settings)

# 注册一个 "Orders" 源表，和 "RubberOrders" 结果表
table_env.sqlUpdate("CREATE TABLE Orders (`user` BIGINT, product STRING, amount INT) WITH (...)")
table_env.sqlUpdate("CREATE TABLE RubberOrders(product STRING, amount INT) WITH (...)")

# 运行一个 INSERT 语句，将源表的数据输出到结果表中
table_env \
    .sqlUpdate("INSERT INTO RubberOrders SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'")
{% endhighlight %}
</div>

<div data-lang="SQL CLI" markdown="1">
{% highlight sql %}
Flink SQL> CREATE TABLE Orders (`user` BIGINT, product STRING, amount INT) WITH (...);
[INFO] Table has been created.

Flink SQL> CREATE TABLE RubberOrders(product STRING, amount INT) WITH (...);

Flink SQL> SHOW TABLES;
Orders
RubberOrders

Flink SQL> INSERT INTO RubberOrders SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%';
[INFO] Submitting SQL update statement to the cluster...
[INFO] Table update statement has been successfully submitted to the cluster:
{% endhighlight %}
</div>
</div>

{% top %}

## 将 SELECT 查询数据插入表中

通过 INSERT 语句，可以将查询的结果插入到表中，

### 语法

{% highlight sql %}

INSERT { INTO | OVERWRITE } [catalog_name.][db_name.]table_name [PARTITION part_spec] select_statement

part_spec:
  (part_col_name1=val1 [, part_col_name2=val2, ...])

{% endhighlight %}

**OVERWRITE**

`INSERT OVERWRITE` 将会覆盖表中或分区中的任何已存在的数据。否则，新数据会追加到表中或分区中。

**PARTITION**

`PARTITION` 语句应该包含需要插入的静态分区列与值。

### 示例

{% highlight sql %}
-- 创建一个分区表
CREATE TABLE country_page_view (user STRING, cnt INT, date STRING, country STRING)
PARTITIONED BY (date, country)
WITH (...)

-- 追加行到该静态分区中 (date='2019-8-30', country='China')
INSERT INTO country_page_view PARTITION (date='2019-8-30', country='China')
  SELECT user, cnt FROM page_view_source;

-- 追加行到分区 (date, country) 中，其中 date 是静态分区 '2019-8-30'；country 是动态分区，其值由每一行动态决定
INSERT INTO country_page_view PARTITION (date='2019-8-30')
  SELECT user, cnt, country FROM page_view_source;

-- 覆盖行到静态分区 (date='2019-8-30', country='China')
INSERT OVERWRITE country_page_view PARTITION (date='2019-8-30', country='China')
  SELECT user, cnt FROM page_view_source;

-- 覆盖行到分区 (date, country) 中，其中 date 是静态分区 '2019-8-30'；country 是动态分区，其值由每一行动态决定
INSERT OVERWRITE country_page_view PARTITION (date='2019-8-30')
  SELECT user, cnt, country FROM page_view_source;
{% endhighlight %}

## 将值插入表中

通过 INSERT 语句，也可以直接将值插入到表中，

### 语法

{% highlight sql %}
INSERT { INTO | OVERWRITE } [catalog_name.][db_name.]table_name VALUES values_row [, values_row ...]

values_row:
    : (val1 [, val2, ...])
{% endhighlight %}

**OVERWRITE**

`INSERT OVERWRITE` 将会覆盖表中的任何已存在的数据。否则，新数据会追加到表中。

### 示例

{% highlight sql %}

CREATE TABLE students (name STRING, age INT, gpa DECIMAL(3, 2)) WITH (...);

INSERT INTO students
  VALUES ('fred flintstone', 35, 1.28), ('barney rubble', 32, 2.32);

{% endhighlight %}

{% top %}