# PostgreSQL 常用知识点、优化手段与设计方案题库

这份题库用于系统学习 PostgreSQL。题目覆盖：

- PostgreSQL 基础概念
- 常用 SQL
- 表结构与约束设计
- 事务、MVCC、锁
- 索引与执行计划
- SQL 优化
- 分区、归档、冷热数据
- 连接池、并发与高可用
- 备份恢复、安全与企业级设计

建议学习方式：

```text
先自己回答，再看答案。
看完答案后，最好在本地 PostgreSQL 里写 SQL 验证。
```

---

## 一、基础概念

### 1. PostgreSQL 是什么？适合哪些场景？

答案：

PostgreSQL 是一个开源关系型数据库，支持标准 SQL、事务、索引、视图、函数、触发器、JSON、全文检索、分区、复制和扩展插件。

适合：

- 企业后台系统
- 订单、支付、积分、账户系统
- 数据一致性要求高的业务
- 复杂查询和报表分析
- GIS 地理数据，配合 PostGIS
- JSON 半结构化数据场景
- 需要强事务和复杂 SQL 的系统

不太适合：

- 极高写入吞吐的日志系统，通常 ClickHouse / Kafka / Elasticsearch 更合适
- 简单 KV 缓存，通常 Redis 更合适
- 海量全文检索，通常 Elasticsearch / OpenSearch 更合适

---

### 2. PostgreSQL 和 MySQL 的主要区别是什么？

答案：

PostgreSQL 更强调 SQL 标准、复杂查询、事务一致性、扩展能力和数据类型丰富度。MySQL 使用更广，生态成熟，很多 Web 项目默认使用 MySQL。

常见区别：

```text
PostgreSQL：复杂 SQL、窗口函数、JSONB、GIS、事务能力更强
MySQL：部署普遍、使用门槛低、互联网业务中非常常见
```

企业选择建议：

- 复杂业务系统、强一致、复杂报表：PostgreSQL
- 普通 Web CRUD、团队更熟 MySQL：MySQL
- 云厂商托管数据库：两者都可以，看团队经验和业务需求

---

### 3. PostgreSQL 中 database、schema、table 的关系是什么？

答案：

层级关系：

```text
PostgreSQL 实例
  └── database
       └── schema
            └── table
```

一个 PostgreSQL 实例可以有多个 database。一个 database 里可以有多个 schema。一个 schema 里可以有多张表。

常见默认 schema 是：

```sql
public
```

比如：

```sql
select *
from public.users;
```

企业项目中可以用 schema 做模块隔离：

```text
auth.users
billing.orders
credit.credit_accounts
```

---

### 4. PostgreSQL 常见数据类型有哪些？

答案：

常用类型：

```text
bigint / integer       整数
numeric                精确小数，适合金额
text / varchar         字符串
boolean                布尔值
timestamp              时间，不带时区
timestamptz            时间，带时区
date                   日期
json / jsonb           JSON 数据
uuid                   UUID
array                  数组
inet                   IP 地址
```

常见建议：

- 主键通常用 `bigserial`、`bigint`、`uuid`
- 金额用 `numeric` 或整数分，不建议用 `float`
- JSON 查询多用 `jsonb`
- 时间字段优先用 `timestamptz`

---

### 5. `timestamp` 和 `timestamptz` 的区别是什么？

答案：

`timestamp` 不带时区，存什么就是什么。

`timestamptz` 带时区语义，PostgreSQL 内部会统一存储为 UTC，查询时按当前 session 时区展示。

企业项目建议：

```text
跨地区、跨时区系统：优先 timestamptz
只在单一地区内部使用：timestamp 也可以
```

常见建表：

```sql
created_at timestamptz not null default now()
```

---

### 6. `json` 和 `jsonb` 有什么区别？

答案：

`json` 原样存储文本，查询时再解析。

`jsonb` 会以二进制格式存储，写入时解析，查询效率更高，也支持更多索引能力。

推荐：

```text
大多数业务使用 jsonb
只有特别要求保留原始 JSON 格式时才用 json
```

示例：

```sql
create table events (
    id bigserial primary key,
    payload jsonb not null
);
```

查询：

```sql
select *
from events
where payload ->> 'type' = 'payment_success';
```

---

### 7. PostgreSQL 中 `serial`、`bigserial`、`identity` 有什么区别？

答案：

`serial` 和 `bigserial` 是 PostgreSQL 早期常用的自增类型，本质上是整数列 + sequence。

`identity` 是更符合 SQL 标准的自增方式。

示例：

```sql
create table users (
    id bigint generated always as identity primary key,
    name text not null
);
```

企业项目中：

```text
老项目常见 bigserial
新项目可以使用 identity
```

---

### 8. PostgreSQL 一条 SQL 的执行过程大概是什么？

答案：

大致过程：

```text
客户端发送 SQL
Parser 解析 SQL
Analyzer 分析表、字段、类型
Rewriter 重写查询
Planner 生成执行计划
Executor 执行计划
返回结果
```

优化主要发生在：

```text
Planner 选择索引、Join 顺序、扫描方式
Executor 按计划执行
```

所以排查慢 SQL 时重点看：

```sql
explain analyze
```

---

## 二、常用 SQL

### 9. 如何创建一张用户表？

答案：

```sql
create table users (
    id bigserial primary key,
    username text not null unique,
    email text not null unique,
    status text not null default 'active',
    created_at timestamptz not null default now(),
    updated_at timestamptz not null default now()
);
```

说明：

- `primary key` 保证主键唯一
- `not null` 保证字段必填
- `unique` 保证唯一性
- `default now()` 自动写入创建时间

---

### 10. 如何插入数据并返回主键？

答案：

使用 `returning`：

```sql
insert into users (username, email)
values ('alice', 'alice@example.com')
returning id;
```

`returning` 很常用，适合插入后立刻拿到：

- id
- 创建时间
- 默认值
- 完整记录

---

### 11. 如何批量插入数据？

答案：

```sql
insert into users (username, email)
values
    ('alice', 'alice@example.com'),
    ('bob', 'bob@example.com'),
    ('tom', 'tom@example.com');
```

大量导入时可以使用：

```sql
copy users(username, email)
from '/tmp/users.csv'
with csv header;
```

业务服务中批量插入通常比循环一条条插入性能更好。

---

### 12. 如何做 UPSERT？

答案：

UPSERT 指：

```text
存在就更新，不存在就插入
```

PostgreSQL 使用 `on conflict`：

```sql
insert into users (username, email)
values ('alice', 'new@example.com')
on conflict (username)
do update set
    email = excluded.email,
    updated_at = now();
```

`excluded.email` 表示本次准备插入的新值。

---

### 13. 如何分页查询？

答案：

普通分页：

```sql
select *
from users
order by id desc
limit 20 offset 40;
```

问题：

`offset` 很大时性能会变差，因为数据库仍然要跳过前面的数据。

更推荐游标分页：

```sql
select *
from users
where id < 10000
order by id desc
limit 20;
```

适合信息流、订单列表、消息列表。

---

### 14. `count(*)`、`count(1)`、`count(column)` 有什么区别？

答案：

`count(*)` 统计所有行。

`count(1)` 也统计所有行，和 `count(*)` 通常无明显性能差异。

`count(column)` 只统计该字段不为 null 的行。

示例：

```sql
select count(*) from users;
select count(email) from users;
```

推荐：

```sql
count(*)
```

语义最清晰。

---

### 15. 如何查询重复数据？

答案：

比如查询重复邮箱：

```sql
select email, count(*)
from users
group by email
having count(*) > 1;
```

如果要删除重复数据，需要先明确保留哪条，常用窗口函数：

```sql
delete from users
where id in (
    select id
    from (
        select id,
               row_number() over (partition by email order by id) as rn
        from users
    ) t
    where rn > 1
);
```

---

### 16. 如何批量更新不同记录的不同值？

答案：

PostgreSQL 推荐 `update ... from values`：

```sql
update skills as s
set sort_order = v.sort_order
from (
    values
        (1, 10),
        (2, 20),
        (3, 30)
) as v(id, sort_order)
where s.id = v.id;
```

适合前端拖拽排序后，后端一次性更新多条记录的排序值。

---

### 17. 如何软删除数据？

答案：

软删除常用字段：

```sql
deleted_at timestamptz
```

删除时：

```sql
update users
set deleted_at = now()
where id = 1;
```

查询时：

```sql
select *
from users
where deleted_at is null;
```

建议给常用查询加部分索引：

```sql
create index idx_users_active
on users(id)
where deleted_at is null;
```

---

### 18. 如何使用窗口函数查询每组最新一条记录？

答案：

比如每个用户最新订单：

```sql
select *
from (
    select o.*,
           row_number() over (
               partition by user_id
               order by created_at desc
           ) as rn
    from orders o
) t
where rn = 1;
```

窗口函数适合：

- 排名
- 分组 Top N
- 同比环比
- 去重保留最新

---

## 三、表设计与约束

### 19. 主键应该怎么设计？

答案：

常见主键方案：

```text
bigint 自增
uuid
雪花 ID
业务唯一 ID
```

推荐：

- 单体或普通业务：`bigint` 自增简单高效
- 分布式、多库多表：雪花 ID 或 UUID
- 对外暴露：可以用业务编号，避免直接暴露自增 ID

注意：

主键不应该频繁变化，也不应该依赖容易变更的业务字段。

---

### 20. 外键一定要用吗？

答案：

不一定。

外键优点：

- 数据库层保证引用完整性
- 防止脏数据
- 建模清晰

外键缺点：

- 高并发写入下可能增加锁和维护成本
- 跨服务、跨库时无法使用
- 删除和迁移数据更复杂

建议：

```text
中小型单体系统：可以使用外键
高并发微服务系统：可以在业务层保证关联关系
核心强一致数据：优先考虑外键或更严格约束
```

---

### 21. `unique` 约束和唯一索引有什么关系？

答案：

PostgreSQL 中 `unique` 约束底层会创建唯一索引。

比如：

```sql
create table users (
    email text unique
);
```

会自动创建唯一索引，保证 email 不重复。

如果需要带条件唯一，比如只限制未删除数据唯一，需要部分唯一索引：

```sql
create unique index uk_users_email_active
on users(email)
where deleted_at is null;
```

---

### 22. 什么是部分索引？适合什么场景？

答案：

部分索引只索引满足条件的数据。

示例：

```sql
create index idx_orders_pending
on orders(created_at)
where status = 'pending';
```

适合：

- 只查询未删除数据
- 只查询 pending 任务
- 只查询 active 用户
- 表很大，但热点数据只占一小部分

优点：

```text
索引更小
查询更快
写入维护成本更低
```

---

### 23. 什么是检查约束？

答案：

检查约束 `check` 用来限制字段值必须符合条件。

示例：

```sql
create table credit_accounts (
    id bigserial primary key,
    balance bigint not null default 0,
    frozen_balance bigint not null default 0,
    check (balance >= 0),
    check (frozen_balance >= 0)
);
```

适合保证基础数据规则：

- 金额不能小于 0
- 状态只能在固定范围内
- 开始时间必须小于结束时间

---

### 24. 状态字段应该用 text、integer 还是 enum？

答案：

三种都可以。

`text`：

- 可读性好
- 扩展方便
- 推荐大多数业务使用

`integer`：

- 存储小
- 可读性差
- 需要代码映射

`enum`：

- 数据库层约束强
- 修改枚举值稍麻烦

企业业务常用：

```sql
status text not null
```

再配合 check：

```sql
check (status in ('pending', 'processing', 'success', 'failed'))
```

---

### 25. 金额字段应该怎么存？

答案：

推荐两种：

方案一：用整数分。

```sql
price_amount bigint not null
```

例如：

```text
1999 表示 19.99 元
```

方案二：用 `numeric`。

```sql
amount numeric(18, 2) not null
```

不建议用：

```sql
float
double precision
```

因为浮点数有精度问题。

---

### 26. 积分账户表为什么不能只存一个 balance？

答案：

只存 `balance` 会有问题：

- 无法追溯每次积分变化
- 无法对账
- 重复扣费难排查
- 异步任务失败后无法判断是否该退回

推荐设计：

```text
credit_accounts  存当前余额
credit_ledgers   存不可变流水
credit_orders    存购买订单
credit_reservations 存冻结记录
```

余额用于快速查询，流水用于审计和对账。

---

### 27. 表字段是否应该都加 `created_at` 和 `updated_at`？

答案：

大多数业务表建议加：

```sql
created_at timestamptz not null default now(),
updated_at timestamptz not null default now()
```

原因：

- 排查问题需要知道数据何时创建和更新
- 后台管理、同步、审计常依赖时间字段
- 很多业务需要按时间排序和筛选

流水表通常只需要 `created_at`，因为流水不应该修改。

---

## 四、事务、MVCC 与锁

### 28. 什么是事务？

答案：

事务是一组数据库操作，要么全部成功，要么全部失败。

事务满足 ACID：

```text
Atomicity    原子性
Consistency 一致性
Isolation   隔离性
Durability  持久性
```

示例：

```sql
begin;

update accounts set balance = balance - 100 where id = 1;
update accounts set balance = balance + 100 where id = 2;

commit;
```

如果中途失败：

```sql
rollback;
```

---

### 29. PostgreSQL 的默认事务隔离级别是什么？

答案：

默认是：

```text
Read Committed
```

含义：

每条 SQL 只能看到执行前已经提交的数据。

PostgreSQL 支持：

```text
Read Committed
Repeatable Read
Serializable
```

常用业务系统默认 `Read Committed` 就足够。

---

### 30. 什么是 MVCC？

答案：

MVCC 是多版本并发控制。

简单理解：

```text
写操作不会直接覆盖旧数据，而是生成新版本
读操作根据事务快照读取自己应该看到的版本
```

好处：

- 读写互不阻塞
- 提升并发性能
- 支持事务隔离

代价：

- 会产生旧版本数据
- 需要 vacuum 清理

---

### 31. 为什么 PostgreSQL 需要 VACUUM？

答案：

因为 PostgreSQL 使用 MVCC，更新和删除数据时旧版本不会立刻物理删除，而是变成 dead tuple。

`vacuum` 负责：

- 清理 dead tuple
- 释放可复用空间
- 防止事务 ID 回卷
- 维护可见性信息

通常依赖 autovacuum 自动执行。

手动执行：

```sql
vacuum analyze users;
```

---

### 32. `VACUUM` 和 `VACUUM FULL` 有什么区别？

答案：

`VACUUM`：

- 不重写整张表
- 一般不阻塞正常读写
- 清理后空间可被表内部复用

`VACUUM FULL`：

- 重写整张表
- 会锁表
- 可以把磁盘空间真正还给操作系统

生产环境慎用：

```sql
vacuum full large_table;
```

大表执行可能导致长时间阻塞。

---

### 33. 什么是行锁？

答案：

行锁是锁住某一行数据，防止并发事务同时修改。

示例：

```sql
begin;

select *
from credit_accounts
where id = 1
for update;

-- 当前事务提交前，其他事务不能修改这一行

commit;
```

常用于：

- 扣库存
- 扣积分
- 账户余额变更
- 抢任务

---

### 34. 如何安全扣减库存或积分？

答案：

推荐使用条件更新：

```sql
update credit_accounts
set balance = balance - 100
where id = 1
  and balance >= 100;
```

然后检查影响行数。

如果影响行数是 0，说明：

```text
账户不存在，或者余额不足
```

这个写法天然支持并发，不会把余额扣成负数。

---

### 35. 什么是死锁？

答案：

死锁是两个或多个事务互相等待对方释放锁，导致都无法继续。

例子：

```text
事务 A 锁住用户 1，再等用户 2
事务 B 锁住用户 2，再等用户 1
```

解决方式：

- 固定加锁顺序
- 缩短事务时间
- 避免事务中做外部 HTTP 调用
- 及时提交或回滚
- 捕获死锁错误后重试

---

### 36. 事务里可以调用第三方接口吗？

答案：

不建议。

原因：

- 第三方接口慢会导致数据库锁长时间持有
- 容易造成锁等待和死锁
- 事务失败后外部接口无法自动回滚

推荐：

```text
事务内只做本地数据库变更
需要通知外部系统时，写 outbox 表
事务提交后由后台任务异步发送
```

---

### 37. 什么是 `select ... for update skip locked`？

答案：

它适合多个 worker 抢任务。

示例：

```sql
select *
from jobs
where status = 'pending'
order by id
limit 10
for update skip locked;
```

含义：

- `for update` 锁住选中的任务
- `skip locked` 跳过已经被其他事务锁住的任务

适合：

- 后台任务队列
- outbox 扫描
- 批量处理 pending 数据

---

### 38. 什么是 advisory lock？

答案：

advisory lock 是 PostgreSQL 提供的应用级锁。

示例：

```sql
select pg_try_advisory_lock(12345);
```

适合：

- 防止同一个定时任务多实例同时执行
- 对某个业务资源加锁
- 做轻量分布式锁

释放：

```sql
select pg_advisory_unlock(12345);
```

注意：

它不是行锁，需要应用自己遵守锁约定。

---

## 五、索引与执行计划

### 39. PostgreSQL 常见索引类型有哪些？

答案：

常见索引：

```text
B-tree   默认索引，适合等值、范围、排序
GIN      适合 jsonb、数组、全文检索
GiST     适合几何、范围、全文检索
BRIN     适合超大表、时间序列、物理顺序相关数据
Hash     适合等值查询，但 B-tree 已经足够常用
```

最常用：

```text
B-tree
GIN
BRIN
```

---

### 40. 什么情况下需要建索引？

答案：

常见需要索引的字段：

- where 高频过滤字段
- join 关联字段
- order by 排序字段
- group by 聚合字段
- 唯一性约束字段
- 外键字段

示例：

```sql
create index idx_orders_user_id
on orders(user_id);
```

不要给所有字段都建索引，因为索引会增加写入成本和磁盘占用。

---

### 41. 联合索引的顺序怎么设计？

答案：

联合索引顺序取决于查询条件。

例如常见查询：

```sql
select *
from orders
where user_id = 123
  and status = 'paid'
order by created_at desc;
```

可以建：

```sql
create index idx_orders_user_status_created
on orders(user_id, status, created_at desc);
```

一般原则：

```text
等值条件靠前
范围条件靠后
排序字段结合 order by 设计
选择性高的字段通常更靠前，但要结合实际查询
```

---

### 42. 什么是覆盖索引？

答案：

覆盖索引指查询需要的数据都在索引里，尽量不回表。

PostgreSQL 可以使用 `include`：

```sql
create index idx_orders_user_created_include
on orders(user_id, created_at desc)
include (status, amount);
```

查询：

```sql
select status, amount
from orders
where user_id = 123
order by created_at desc
limit 20;
```

索引中已经包含 `status` 和 `amount`，可能减少访问表数据的成本。

---

### 43. `like '%abc%'` 为什么普通索引通常没用？

答案：

B-tree 索引适合从左开始匹配。

可以使用索引：

```sql
where name like 'abc%'
```

通常不能有效使用普通 B-tree：

```sql
where name like '%abc%'
```

解决方式：

使用 `pg_trgm` 扩展和 GIN 索引：

```sql
create extension if not exists pg_trgm;

create index idx_users_name_trgm
on users using gin (name gin_trgm_ops);
```

---

### 44. JSONB 字段如何建索引？

答案：

整体 JSONB GIN 索引：

```sql
create index idx_events_payload
on events using gin (payload);
```

适合包含查询：

```sql
select *
from events
where payload @> '{"type": "payment"}';
```

如果经常按某个 key 查询，可以建表达式索引：

```sql
create index idx_events_type
on events ((payload ->> 'type'));
```

查询：

```sql
select *
from events
where payload ->> 'type' = 'payment';
```

---

### 45. 什么是表达式索引？

答案：

表达式索引不是直接索引字段，而是索引计算结果。

示例：邮箱忽略大小写唯一。

```sql
create unique index uk_users_lower_email
on users (lower(email));
```

查询：

```sql
select *
from users
where lower(email) = lower('Alice@Example.com');
```

适合：

- lower(email)
- date(created_at)
- jsonb key 提取
- 函数计算结果

---

### 46. 为什么有索引但查询没有走索引？

答案：

常见原因：

- 表太小，全表扫描更快
- 查询条件选择性太低
- 条件写法导致索引失效
- 类型不一致，发生隐式转换
- 统计信息过旧
- `like '%xxx%'` 普通索引无效
- 返回数据太多，顺序扫描更划算

处理方式：

```sql
analyze table_name;
explain analyze select ...
```

先看执行计划，再决定是否调整索引或 SQL。

---

### 47. `EXPLAIN` 和 `EXPLAIN ANALYZE` 有什么区别？

答案：

`EXPLAIN` 只展示预估执行计划，不真正执行 SQL。

```sql
explain
select * from orders where user_id = 1;
```

`EXPLAIN ANALYZE` 会真正执行 SQL，并返回实际耗时和实际行数。

```sql
explain analyze
select * from orders where user_id = 1;
```

排查慢查询通常用：

```sql
explain (analyze, buffers)
```

---

### 48. 执行计划里常见扫描方式有哪些？

答案：

常见：

```text
Seq Scan          顺序扫描，全表扫描
Index Scan        使用索引扫描，再回表
Index Only Scan   只扫索引，尽量不回表
Bitmap Index Scan 先扫索引生成 bitmap
Bitmap Heap Scan  再按 bitmap 扫表
```

看到 `Seq Scan` 不一定就是坏事，小表全表扫描可能更快。

大表高频查询出现 `Seq Scan`，需要重点关注。

---

### 49. 执行计划里的 `cost` 是什么？

答案：

`cost` 是 PostgreSQL 优化器估算的执行成本，不是时间单位。

示例：

```text
cost=0.42..8.44
```

含义：

- 第一个值：启动成本
- 第二个值：总成本

优化器会比较不同计划的 cost，选择认为最便宜的方案。

真正耗时看：

```text
actual time
```

---

### 50. 执行计划里的 `rows` 估算不准怎么办？

答案：

`rows` 估算不准通常是统计信息不准。

处理：

```sql
analyze table_name;
```

如果字段分布很不均匀，可以提高统计目标：

```sql
alter table orders
alter column status
set statistics 1000;

analyze orders;
```

估算不准会导致：

- 选错索引
- Join 顺序错误
- Nested Loop 被误用
- 查询突然变慢

---

### 51. Join 有哪些常见执行方式？

答案：

PostgreSQL 常见 Join：

```text
Nested Loop Join  嵌套循环，适合小表驱动大表索引查询
Hash Join         哈希连接，适合大表等值连接
Merge Join        排序后合并，适合有序数据连接
```

优化 Join：

- Join 字段加索引
- 减少参与 Join 的数据量
- 先过滤再 Join
- 确保统计信息准确

---

### 52. 什么是 N+1 查询问题？

答案：

N+1 查询是应用层常见性能问题。

例子：

```text
先查询 100 个订单
再循环每个订单查询用户
最终 1 + 100 次 SQL
```

解决：

使用 Join 或批量查询：

```sql
select o.*, u.username
from orders o
join users u on u.id = o.user_id
where o.created_at >= now() - interval '1 day';
```

或：

```sql
select *
from users
where id = any(array[1,2,3]);
```

---

### 53. 如何找出慢 SQL？

答案：

方式一：开启慢日志。

```sql
alter system set log_min_duration_statement = '500ms';
select pg_reload_conf();
```

方式二：使用 `pg_stat_statements`。

```sql
create extension if not exists pg_stat_statements;
```

查询耗时最高 SQL：

```sql
select query,
       calls,
       total_exec_time,
       mean_exec_time,
       rows
from pg_stat_statements
order by total_exec_time desc
limit 20;
```

---

### 54. 什么是索引膨胀？

答案：

索引膨胀是索引中存在大量无效或空洞页，导致索引变大、查询变慢。

原因：

- 大量 update/delete
- autovacuum 不及时
- 长事务阻止清理

处理：

```sql
reindex index idx_name;
```

或：

```sql
reindex table table_name;
```

生产环境需要评估锁影响，必要时使用并发方式：

```sql
reindex index concurrently idx_name;
```

---

### 55. `CREATE INDEX CONCURRENTLY` 有什么作用？

答案：

普通建索引可能阻塞写入。

并发建索引：

```sql
create index concurrently idx_orders_created_at
on orders(created_at);
```

优点：

- 不长时间阻塞读写
- 适合生产环境大表建索引

注意：

- 不能放在普通事务块里执行
- 执行时间通常比普通建索引更长
- 失败后可能留下 invalid index，需要清理

---

## 六、SQL 优化与性能设计

### 56. 慢 SQL 优化的一般流程是什么？

答案：

推荐流程：

```text
1. 确认慢 SQL 和参数
2. 使用 explain analyze 查看执行计划
3. 看扫描方式、实际行数、耗时、buffers
4. 判断是否缺索引或索引不合适
5. 判断 Join 顺序和过滤条件是否合理
6. 优化 SQL 或索引
7. 再次 explain analyze 验证
8. 上生产后观察 pg_stat_statements
```

不要凭感觉加索引。

---

### 57. 为什么 `select *` 不推荐？

答案：

问题：

- 读取不需要的字段
- 增加网络传输
- 增加内存消耗
- 可能无法使用覆盖索引
- 表字段变更会影响应用

推荐：

```sql
select id, username, created_at
from users
where id = 1;
```

只查询业务需要的字段。

---

### 58. 大表分页为什么不推荐大 offset？

答案：

例如：

```sql
select *
from orders
order by id
limit 20 offset 1000000;
```

数据库需要跳过前 1000000 行，再返回 20 行。

优化：

使用游标分页：

```sql
select *
from orders
where id > 1000000
order by id
limit 20;
```

适合：

- 订单列表
- 消息列表
- Feed 流
- 日志列表

---

### 59. `or` 查询为什么可能慢？如何优化？

答案：

复杂 `or` 可能导致索引利用差。

示例：

```sql
select *
from orders
where user_id = 1
   or status = 'pending';
```

优化方式：

- 分别建合适索引
- 拆成 `union all`
- 评估业务是否能拆查询

```sql
select * from orders where user_id = 1
union all
select * from orders where status = 'pending' and user_id <> 1;
```

具体要以执行计划为准。

---

### 60. 在字段上使用函数为什么可能导致索引失效？

答案：

比如：

```sql
select *
from orders
where date(created_at) = '2026-05-19';
```

如果只有 `created_at` 普通索引，函数计算可能导致索引无法直接使用。

推荐写法：

```sql
select *
from orders
where created_at >= '2026-05-19 00:00:00'
  and created_at <  '2026-05-20 00:00:00';
```

或者创建表达式索引：

```sql
create index idx_orders_created_date
on orders ((date(created_at)));
```

---

### 61. 为什么要避免长事务？

答案：

长事务会带来：

- 阻止 vacuum 清理旧版本
- 导致表膨胀
- 锁持有时间过长
- 增加死锁风险
- 影响复制延迟

常见错误：

```text
begin 后执行很久的业务逻辑
事务里调用第三方接口
事务里等待用户输入
忘记 commit / rollback
```

原则：

```text
事务越短越好
事务内只做必要数据库操作
```

---

### 62. 批量更新大表时应该注意什么？

答案：

不要一次更新几千万行。

建议：

- 分批更新
- 每批提交
- 控制批次大小
- 避开业务高峰
- 观察锁和复制延迟
- 必要时先建索引

示例：

```sql
update users
set status = 'inactive'
where id in (
    select id
    from users
    where last_login_at < now() - interval '1 year'
    limit 10000
);
```

循环执行，直到没有数据。

---

### 63. 如何优化统计类报表查询？

答案：

方式：

- 给过滤字段和时间字段建索引
- 使用物化视图
- 使用汇总表
- 按天/月预聚合
- 对大表做分区
- 把 OLAP 查询迁移到 ClickHouse 等分析数据库

物化视图示例：

```sql
create materialized view daily_order_stats as
select date(created_at) as day,
       count(*) as order_count,
       sum(amount) as total_amount
from orders
group by date(created_at);
```

刷新：

```sql
refresh materialized view daily_order_stats;
```

---

### 64. 什么是冷热数据分离？

答案：

冷热数据分离是把高频访问的新数据和低频访问的旧数据分开存储或分开管理。

例如订单表：

```text
近 3 个月订单：热数据
3 个月以前订单：冷数据
```

方案：

- 按时间分区
- 历史数据归档到 history 表
- 冷数据放对象存储或分析库
- 热表保持小而快

好处：

- 提升查询速度
- 降低索引体积
- 降低 vacuum 压力
- 方便归档和清理

---

### 65. 什么时候应该反范式设计？

答案：

反范式指为了查询性能，适当冗余数据。

适合：

- 查询远多于写入
- Join 成本很高
- 页面需要频繁展示固定字段
- 历史快照不能受源表变化影响

示例：

订单表冗余用户下单时昵称：

```sql
user_name_snapshot text
```

即使用户后来改名，历史订单仍显示下单时昵称。

注意：

反范式会增加数据一致性维护成本。

---

### 66. 如何设计高并发任务领取？

答案：

使用 `for update skip locked`：

```sql
begin;

select id
from jobs
where status = 'pending'
order by id
limit 10
for update skip locked;

update jobs
set status = 'processing',
    locked_at = now()
where id in (...);

commit;
```

多个 worker 同时执行时，不会领取到同一批任务。

适合：

- outbox 投递
- 异步任务
- 邮件发送
- 视频任务调度

---

### 67. Outbox 表应该怎么设计？

答案：

Outbox 用于保证“业务数据变更”和“消息发送”一致。

表结构：

```sql
create table outbox_events (
    id bigserial primary key,
    event_type text not null,
    aggregate_type text not null,
    aggregate_id text not null,
    payload jsonb not null,
    status text not null default 'pending',
    retry_count integer not null default 0,
    next_retry_at timestamptz not null default now(),
    created_at timestamptz not null default now(),
    sent_at timestamptz
);
```

业务事务中：

```text
更新业务表
插入 outbox_events
一起 commit
```

后台 worker 再扫描 pending 事件并发送到 MQ / Redis Stream。

---

## 七、分区、归档与大表设计

### 68. PostgreSQL 分区表适合什么场景？

答案：

适合：

- 超大表
- 时间序列数据
- 日志、事件、订单、流水
- 需要按时间清理历史数据
- 查询经常带时间范围

不适合：

- 小表
- 查询条件不包含分区键
- 分区过多但管理不当

分区不是万能优化，必须结合查询模式。

---

### 69. 如何创建按月分区表？

答案：

```sql
create table events (
    id bigint not null,
    created_at timestamptz not null,
    payload jsonb not null
) partition by range (created_at);
```

创建分区：

```sql
create table events_2026_05
partition of events
for values from ('2026-05-01') to ('2026-06-01');
```

查询：

```sql
select *
from events
where created_at >= '2026-05-01'
  and created_at < '2026-06-01';
```

可以触发分区裁剪，只扫描对应月份分区。

---

### 70. 分区键应该怎么选？

答案：

分区键要看查询模式。

常见选择：

```text
时间字段 created_at
租户 ID tenant_id
业务 ID hash
```

原则：

- 查询条件经常包含这个字段
- 数据分布较均匀
- 方便清理和归档
- 不要选择频繁变化字段

日志、订单、流水通常按时间分区。

---

### 71. 分区表还需要索引吗？

答案：

需要。

分区只是把数据拆到不同子表，不等于自动优化所有查询。

示例：

```sql
create index idx_events_2026_05_type
on events_2026_05 ((payload ->> 'type'));
```

或者在父表上创建分区索引：

```sql
create index idx_events_created_at
on events(created_at);
```

PostgreSQL 会在各分区创建对应索引。

---

### 72. 如何快速删除历史数据？

答案：

如果使用时间分区，直接 drop 老分区：

```sql
drop table events_2025_01;
```

比下面这种快很多：

```sql
delete from events
where created_at < '2025-02-01';
```

原因：

`drop partition` 是元数据操作，`delete` 会逐行删除并产生大量 dead tuple。

---

### 73. 分库分表和 PostgreSQL 分区有什么区别？

答案：

PostgreSQL 分区：

```text
一个数据库内部拆表
应用层通常无感知
主要解决大表管理和部分查询性能
```

分库分表：

```text
数据分散到多个数据库或多个实例
应用层或中间件需要路由
主要解决单机容量和吞吐瓶颈
```

优先级建议：

```text
先优化 SQL 和索引
再考虑分区
最后才考虑分库分表
```

---

## 八、连接池、复制、高可用与备份

### 74. 为什么需要数据库连接池？

答案：

数据库连接创建成本较高，连接数过多也会拖垮数据库。

连接池作用：

- 复用连接
- 控制最大连接数
- 降低连接创建开销
- 防止服务把数据库打爆

常见工具：

```text
应用内连接池
PgBouncer
```

Python 常见：

```text
SQLAlchemy pool
asyncpg pool
psycopg pool
```

---

### 75. PostgreSQL 连接数是不是越大越好？

答案：

不是。

连接数过高会导致：

- 内存占用增加
- 上下文切换增加
- 锁竞争增加
- 查询变慢

高并发系统通常使用 PgBouncer 控制连接。

原则：

```text
应用并发高，不代表数据库连接数也要高
数据库连接应该被复用和限制
```

---

### 76. 什么是 PgBouncer？

答案：

PgBouncer 是 PostgreSQL 常用连接池代理。

请求路径：

```text
应用服务 -> PgBouncer -> PostgreSQL
```

作用：

- 减少真实数据库连接数
- 提升连接复用
- 防止连接风暴
- 适合大量短连接场景

常见池模式：

```text
session
transaction
statement
```

最常用是 transaction pooling。

---

### 77. PostgreSQL 主从复制是什么？

答案：

主从复制是把主库数据同步到一个或多个从库。

常见用途：

- 高可用
- 读写分离
- 备份
- 灾备

基本结构：

```text
应用写入 -> 主库
应用读取 -> 主库或从库
主库 WAL -> 从库重放
```

PostgreSQL 通常使用流复制。

---

### 78. 读写分离要注意什么？

答案：

读写分离常见问题是复制延迟。

比如：

```text
用户刚写入主库
马上去从库查询
从库还没同步到
用户看不到刚写入的数据
```

解决：

- 强一致读走主库
- 用户刚写完的一段时间读主库
- 检测复制延迟
- 关键链路不走从库

不是所有读都适合走从库。

---

### 79. WAL 是什么？

答案：

WAL 是 Write-Ahead Logging，预写日志。

PostgreSQL 修改数据前，会先写 WAL。

作用：

- 崩溃恢复
- 主从复制
- PITR 时间点恢复
- 保证事务持久性

可以理解为：

```text
先记操作日志，再真正修改数据文件
```

---

### 80. 如何做 PostgreSQL 备份？

答案：

常见方式：

逻辑备份：

```bash
pg_dump -Fc -f backup.dump mydb
```

恢复：

```bash
pg_restore -d mydb backup.dump
```

物理备份：

```bash
pg_basebackup
```

企业建议：

- 小库用 `pg_dump`
- 大库用物理备份 + WAL 归档
- 定期演练恢复
- 备份要异地保存

---

### 81. 备份成功就安全吗？

答案：

不一定。

真正安全需要：

- 定期备份
- 备份文件可用
- 定期恢复演练
- 明确 RPO / RTO
- 备份加密
- 异地备份

RPO：

```text
最多能丢多少数据
```

RTO：

```text
最多多久恢复服务
```

---

### 82. PostgreSQL 如何做权限控制？

答案：

使用 role 和 grant。

创建用户：

```sql
create role app_user login password 'strong_password';
```

授权：

```sql
grant connect on database appdb to app_user;
grant usage on schema public to app_user;
grant select, insert, update, delete on all tables in schema public to app_user;
```

原则：

```text
应用账号不要使用超级用户
不同环境不同账号
只授予必要权限
```

---

### 83. 如何避免 SQL 注入？

答案：

不要拼接用户输入。

错误示例：

```python
sql = f"select * from users where name = '{name}'"
```

正确方式：

```python
cursor.execute(
    "select * from users where name = %s",
    [name],
)
```

或使用 ORM 参数绑定。

原则：

```text
值用参数绑定
表名和字段名不能直接用用户输入
```

---

### 84. 什么是行级安全 RLS？

答案：

RLS 是 Row Level Security，行级安全策略。

启用：

```sql
alter table documents enable row level security;
```

创建策略：

```sql
create policy user_documents_policy
on documents
using (owner_id = current_setting('app.user_id')::bigint);
```

适合：

- 多租户系统
- 用户只能访问自己的数据
- 数据权限强隔离

使用 RLS 要谨慎设计和充分测试。

---

### 85. 如何加密敏感数据？

答案：

常见方式：

- 传输层使用 TLS
- 磁盘层使用云盘加密
- 应用层加密敏感字段
- 密码使用哈希，不可逆加密

密码不要明文存储，使用：

```text
bcrypt
argon2
pbkdf2
```

身份证、手机号等可以：

- 加密存储
- 单独脱敏展示
- 建 hash 字段用于等值查询

---

## 九、企业级设计题

### 86. 如何设计积分系统表结构？

答案：

推荐：

```text
credit_accounts      积分账户余额
credit_ledgers       积分流水
credit_orders        积分购买订单
credit_reservations  积分冻结记录
```

核心原则：

- 余额表快速查余额
- 流水表不可变，用于审计
- 购买、消耗、退款、赠送都写流水
- 长任务先冻结，成功确认扣除，失败释放
- 幂等 key 防止重复扣费

---

### 87. 积分消耗如何做幂等？

答案：

用业务唯一键生成幂等 key：

```text
consume:video_generation:task_1001
```

流水表加唯一约束：

```sql
create unique index uk_credit_ledgers_idempotency
on credit_ledgers(idempotency_key);
```

事务内：

```text
1. 锁定账户
2. 查询是否已有该 idempotency_key
3. 如果已存在，直接返回成功
4. 如果不存在，扣减余额
5. 插入流水
6. commit
```

解决：

- 重复点击
- 接口超时重试
- worker 重试
- 消息重复投递

---

### 88. 如何设计订单支付回调？

答案：

流程：

```text
1. 接收支付平台回调
2. 校验签名
3. 根据 order_no 查询订单 for update
4. 如果订单已 paid，直接返回成功
5. 如果未支付，更新为 paid
6. 发放权益或积分
7. 写流水
8. commit
```

关键：

- 回调必须幂等
- 支付金额必须校验
- 订单状态流转必须受控
- 不要相信前端支付成功页面

---

### 89. 如何设计生视频任务表？

答案：

常见字段：

```sql
create table video_tasks (
    id bigserial primary key,
    user_id bigint not null,
    status text not null,
    provider text,
    provider_task_id text,
    prompt text,
    video_url text,
    error_message text,
    retry_count integer not null default 0,
    created_at timestamptz not null default now(),
    updated_at timestamptz not null default now(),
    completed_at timestamptz
);
```

状态：

```text
pending
processing
submitted
succeeded
failed
cancelled
```

索引：

```sql
create index idx_video_tasks_user_created
on video_tasks(user_id, created_at desc);

create index idx_video_tasks_pending
on video_tasks(created_at)
where status = 'pending';
```

---

### 90. 如何设计后台任务队列表？

答案：

```sql
create table jobs (
    id bigserial primary key,
    job_type text not null,
    payload jsonb not null,
    status text not null default 'pending',
    retry_count integer not null default 0,
    max_retries integer not null default 3,
    next_run_at timestamptz not null default now(),
    locked_at timestamptz,
    locked_by text,
    error_message text,
    created_at timestamptz not null default now(),
    updated_at timestamptz not null default now()
);
```

领取任务：

```sql
select *
from jobs
where status = 'pending'
  and next_run_at <= now()
order by id
limit 10
for update skip locked;
```

---

### 91. 如何设计用户通知系统？

答案：

表：

```text
notifications
notification_reads
```

`notifications` 存通知内容：

```sql
create table notifications (
    id bigserial primary key,
    user_id bigint not null,
    title text not null,
    content text not null,
    type text not null,
    created_at timestamptz not null default now()
);
```

如果每个用户一条通知，直接在通知表加 `read_at`。

如果一条通知发给很多人，使用关联表：

```sql
notification_reads(user_id, notification_id, read_at)
```

---

### 92. 如何设计多租户系统？

答案：

常见方案：

方案一：所有表加 `tenant_id`

```sql
tenant_id bigint not null
```

优点：简单，成本低。

方案二：每个租户一个 schema。

优点：隔离更强。

方案三：每个租户一个 database。

优点：隔离最强，成本最高。

大多数 SaaS 项目第一阶段推荐：

```text
共享数据库 + 表加 tenant_id + 严格索引 + 权限校验
```

---

### 93. 多租户表索引怎么建？

答案：

多租户表查询通常都带 `tenant_id`。

索引应该包含 `tenant_id`：

```sql
create index idx_orders_tenant_user_created
on orders(tenant_id, user_id, created_at desc);
```

不要只建：

```sql
create index idx_orders_user_id on orders(user_id);
```

否则不同租户数据混在一起，过滤效率可能差。

---

### 94. 如何设计审计日志？

答案：

审计日志记录谁在什么时候做了什么。

```sql
create table audit_logs (
    id bigserial primary key,
    actor_type text not null,
    actor_id bigint,
    action text not null,
    resource_type text not null,
    resource_id text,
    ip inet,
    user_agent text,
    before_data jsonb,
    after_data jsonb,
    created_at timestamptz not null default now()
);
```

特点：

- 通常只插入，不修改
- 按时间分区
- 长期归档
- 适合合规和问题追踪

---

### 95. 如何设计软删除下的唯一约束？

答案：

需求：

```text
未删除用户 email 唯一
已删除用户 email 可以重复
```

使用部分唯一索引：

```sql
create unique index uk_users_email_active
on users(email)
where deleted_at is null;
```

这样只限制未删除数据唯一。

---

### 96. 如何设计标签系统？

答案：

三张表：

```text
tags
articles
article_tags
```

```sql
create table tags (
    id bigserial primary key,
    name text not null unique
);

create table article_tags (
    article_id bigint not null,
    tag_id bigint not null,
    primary key (article_id, tag_id)
);
```

查询文章标签：

```sql
select t.*
from tags t
join article_tags at on at.tag_id = t.id
where at.article_id = 1;
```

---

### 97. 如何设计树形结构？

答案：

常见方案：

邻接表：

```sql
id
parent_id
```

适合简单树。

路径枚举：

```sql
path = '1.3.8'
```

适合快速查询子树。

闭包表：

```text
ancestor_id
descendant_id
depth
```

适合复杂树关系查询。

如果层级不深，邻接表通常够用。

---

### 98. 如何设计消息已读未读？

答案：

如果消息是发给单个用户：

```sql
messages (
    id,
    user_id,
    content,
    read_at
)
```

如果消息是群发：

```text
messages
message_receipts
```

```sql
create table message_receipts (
    message_id bigint not null,
    user_id bigint not null,
    read_at timestamptz,
    primary key (message_id, user_id)
);
```

群发消息不要为每个用户复制完整消息内容。

---

### 99. 如何设计资源排序？

答案：

简单方案：

```sql
sort_order integer not null
```

前端拖拽后，后端批量更新：

```sql
update skills as s
set sort_order = v.sort_order
from (
    values
        (1, 1),
        (5, 2),
        (3, 3)
) as v(id, sort_order)
where s.id = v.id;
```

注意：

- 校验这些 ID 是否属于当前用户或当前项目
- 放在事务中更新
- 必要时加唯一约束：

```sql
unique(project_id, sort_order)
```

---

### 100. 如何设计数据库迁移流程？

答案：

推荐使用迁移工具：

```text
Flyway
Liquibase
Alembic
Prisma Migrate
Goose
Atlas
```

原则：

- DDL 进入版本管理
- 禁止手工改生产库
- 迁移脚本可重复审查
- 大表变更要评估锁
- 生产建索引用 concurrently
- 字段删除分多步走

安全删除字段流程：

```text
1. 代码停止写这个字段
2. 代码停止读这个字段
3. 观察一段时间
4. 再删除字段
```

---

## 十、综合排查题

### 101. 一个接口突然变慢，如何判断是不是数据库慢？

答案：

排查顺序：

```text
1. 看接口总耗时
2. 看数据库查询耗时
3. 找出慢 SQL
4. explain analyze 看执行计划
5. 查看是否锁等待
6. 查看数据库 CPU、IO、连接数
7. 查看最近是否发版或数据量突增
```

常用 SQL：

```sql
select pid, state, wait_event_type, wait_event, query
from pg_stat_activity
where state <> 'idle';
```

---

### 102. 如何查看当前数据库正在执行的 SQL？

答案：

```sql
select pid,
       usename,
       state,
       wait_event_type,
       wait_event,
       now() - query_start as duration,
       query
from pg_stat_activity
where state <> 'idle'
order by duration desc;
```

用于排查：

- 慢查询
- 锁等待
- 长事务
- 卡住的 SQL

---

### 103. 如何排查锁等待？

答案：

先看等待中的 SQL：

```sql
select pid,
       wait_event_type,
       wait_event,
       now() - query_start as duration,
       query
from pg_stat_activity
where wait_event_type = 'Lock';
```

再找阻塞关系：

```sql
select blocked.pid as blocked_pid,
       blocked.query as blocked_query,
       blocking.pid as blocking_pid,
       blocking.query as blocking_query
from pg_stat_activity blocked
join pg_locks blocked_locks
  on blocked_locks.pid = blocked.pid
join pg_locks blocking_locks
  on blocking_locks.locktype = blocked_locks.locktype
 and blocking_locks.database is not distinct from blocked_locks.database
 and blocking_locks.relation is not distinct from blocked_locks.relation
 and blocking_locks.page is not distinct from blocked_locks.page
 and blocking_locks.tuple is not distinct from blocked_locks.tuple
 and blocking_locks.pid <> blocked_locks.pid
join pg_stat_activity blocking
  on blocking.pid = blocking_locks.pid
where not blocked_locks.granted;
```

处理前要确认业务影响，不能随便 kill。

---

### 104. 如何终止一个异常 SQL？

答案：

温和取消：

```sql
select pg_cancel_backend(pid);
```

强制终止连接：

```sql
select pg_terminate_backend(pid);
```

区别：

```text
pg_cancel_backend：只取消当前 SQL
pg_terminate_backend：断开整个连接
```

生产环境操作前必须确认 pid、用户和 SQL。

---

### 105. 数据库 CPU 很高可能是什么原因？

答案：

常见原因：

- 慢 SQL 大量全表扫描
- 缺索引
- 排序、聚合消耗大
- Join 计划不合理
- 连接数过多
- 高频简单 SQL 没有缓存
- 执行计划突然变化

排查：

```text
pg_stat_activity
pg_stat_statements
explain analyze
系统 CPU / IO 指标
```

---

### 106. 数据库磁盘增长很快可能是什么原因？

答案：

常见原因：

- 业务数据真实增长
- 大量 update/delete 导致表膨胀
- autovacuum 不及时
- 长事务阻止清理
- 索引膨胀
- WAL 增长
- 日志文件增长

排查：

```sql
select relname,
       pg_size_pretty(pg_total_relation_size(relid)) as total_size
from pg_catalog.pg_statio_user_tables
order by pg_total_relation_size(relid) desc
limit 20;
```

---

### 107. 如何判断一个表是否需要优化索引？

答案：

看三个方面：

```text
1. 慢查询是否经常访问这张表
2. explain analyze 是否出现大表 Seq Scan
3. 现有索引是否长期未使用
```

查看索引使用情况：

```sql
select relname,
       indexrelname,
       idx_scan
from pg_stat_user_indexes
order by idx_scan asc;
```

`idx_scan` 很低不一定代表索引没用，需要结合业务周期判断。

---

### 108. 如何设计一个高可靠的“扣积分 + 创建任务”流程？

答案：

推荐：

```text
1. 开启事务
2. 根据 user_id 锁定积分账户
3. 检查余额
4. 检查幂等 key
5. 冻结积分
6. 创建视频任务
7. 写积分冻结流水
8. 写 outbox 事件
9. commit
10. 后台 worker 读取 outbox，真正提交给视频供应商
```

这样可以保证：

- 积分不会重复扣
- 任务创建和积分冻结一致
- 提交第三方失败可以重试
- 任务失败后可以释放冻结积分

---

### 109. 生产环境加字段怎么降低风险？

答案：

普通新增可空字段通常风险较低：

```sql
alter table users add column nickname text;
```

大表注意：

- 避免带复杂默认值导致重写表
- 避免高峰期执行
- 先加 nullable 字段
- 分批回填
- 再加 not null 约束

安全流程：

```text
1. add nullable column
2. 发布代码写新字段
3. 后台分批回填历史数据
4. 校验无 null
5. 添加 not null 约束
```

---

### 110. PostgreSQL 项目上线前应该检查什么？

答案：

清单：

```text
表是否有主键
核心字段是否有 not null
唯一业务字段是否有 unique
常用查询是否有索引
慢查询是否 explain 过
事务是否过长
是否有幂等设计
是否有备份策略
是否有连接池
是否开启慢 SQL 日志
是否有权限隔离
是否有迁移脚本
是否有监控和告警
```

企业级项目至少要具备：

- 备份恢复
- 慢 SQL 发现
- 连接数监控
- 锁等待监控
- 磁盘监控
- 关键业务流水和审计

