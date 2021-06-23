
# Mazur 的 SQL 风格指南

您好！我是 [Matt Mazur](https://mattmazur.com/)，是一名数据分析师，曾在几家初创公司工作过，帮助公司利用数据发展业务。本指南记录了我对格式化 SQL 的喜好，希望对其他人有一些用处。如果您或您的团队还没有 SQL 风格指南，那么它可以作为一个很好的起点，您可以根据自己的喜好来采用和更新它。

另外，我是坚持 [“有强烈的观点，但不盲目坚持”（Strong Opinions, Weakly Held）](https://medium.com/@ameet/strong-opinions-weakly-held-a-framework-for-thinking-6530d417e364) 这种态度的人。如果你对这篇文档有任何异议, [给我私信](https://mattmazur.com/contact/)， 我愿意一起讨论。

如果你喜欢这个话题，你应该也喜欢我的 [LookML Style Guide](https://github.com/mattm/lookml-style-guide) 或者我的[博客](https://mattmazur.com/category/analytics/)，我写了很多关于分析和数据分析的文章.

## 例子

这里是一些比较简单的查询，主要是为了展示下这个指南：


```sql
with hubspot_interest as (

    select
        email,
        timestamp_millis(property_beacon_interest) as expressed_interest_at
    from hubspot.contact
    where property_beacon_interest is not null

), 

support_interest as (

    select 
        conversation.email,
        conversation.created_at as expressed_interest_at
    from helpscout.conversation
    inner join helpscout.conversation_tag on conversation.id = conversation_tag.conversation_id
    where conversation_tag.tag = 'beacon-interest'

), 

combined_interest as (

    select * from hubspot_interest
    union all
    select * from support_interest

),

final as (

    select 
        email,
        min(expressed_interest_at) as expressed_interest_at
    from combined_interest
    group by email

)

select * from final
```

## 指南

### 使用小写字母 SQL

它就像大写 SQL 一样易读，而且你不必总是按住 `Shift` 键。

```sql
-- 好
select * from users

-- 不好
SELECT * FROM users

-- 不好
Select * From users
```

### 单行查询 vs 多行查询

仅在查询一个事物，且不复杂的情况下，使用单行查询：

```sql
-- 好
select * from users

-- 好
select id from users

-- 好
select count(*) from users
```

一旦你需要查询更多的列或比较复杂，分散在多行可以变得更容易阅读：

```sql
-- 好
select
    id,
    email,
    created_at
from users

-- 好
select *
from users
where email = 'example@domain.com'

-- 好
select
    user_id,
    count(*) as total_charges
from charges
group by user_id

-- 不好
select id, email, created_at
from users

-- 不好
select id,
    email
from users
```

### 左对齐 SQL 关键字

有些 IDE 能够自动格式化 SQL，以便 SQL 关键字之后的空格垂直对齐。手动做这个格式化非常麻烦（在我看来这样也更难阅读），所以我建议所有的关键字都左对齐。

```sql
-- 好
select id, email
from users
where email like '%@gmail.com'

-- 不好
select id, email
  from users
 where email like '%@gmail.com'
```

### 使用单引号

有些 SQL 分支（例如 BigQuery）支持使用双引号，但是对于大多数分支，双引号都使用在列名上，因此最好使用单引号。

```sql
-- 好
select *
from users
where email = 'example@domain.com'

-- 不好
select *
from users
where email = "example@domain.com"
```

### 使用 `!=` 而不是 `<>`

很简单，因为 `!=` 看起来像 “不等于”，更接近我们想要表达的意思。

```sql
-- 好
select count(*) as paying_users_count
from users
where plan_name != 'free'
```

### 逗号应该在行尾

```sql
-- 好
select
    id,
    email
from users

-- 不好
select
    id
    , email
from users
```

### where 条件的缩进

当只有一个条件时，条件与 `where` 保持在同一行：

```sql
select email
from users
where id = 1234
```

当有多个条件时，每一个条件都比 `where` 缩进一层。将逻辑运算符放在前一个条件的末尾：


```sql
select id, email
from users
where 
    created_at >= '2019-03-01' and 
    vertical = 'work'
```

### 避免括号内的空格

```sql
-- 好
select *
from users
where id in (1, 2)

-- 不好
select *
from users
where id in ( 1, 2 )
```

### `in` 中比较长的列表，应该分在多个不同的缩进行

```sql
-- 好
select *
from users
where email in (
    'user-1@example.com',
    'user-2@example.com',
    'user-3@example.com',
    'user-4@example.com'
)
```

### 表名应该是蛇形复数名词风格

```sql
-- 好
select * from users
select * from visit_logs

-- 不好
select * from user
select * from visitLog
```

### 列名应该是蛇形风格

```sql
-- 好
select
    id,
    email,
    timestamp_trunc(created_at, month) as signup_month
from users

-- 不好
select
    id,
    email,
    timestamp_trunc(created_at, month) as SignupMonth
from users
```

### 列名约定

* 布尔值类型应该有 `is_`、`has_` 或 `does_` 前缀。例如 `is_customer`、 `has_unsubscribed` 等。
* 只有日期的类型应该有 `_date` 后缀。例如 `report_date` 等。
* 日期时间类型应该有 `_at` 后缀。例如 `created_at`、`posted_at` 等。

### 列顺序约定

将主键放到最前面，然后是外键，最后是其他列。如果有任何系统列（如 `created_at`、`updated_at`、`is_deleted`），把它们放到最后。

```sql
-- 好
select
    id,
    name,
    created_at
from users

-- 不好
select
    created_at,
    name,
    id,
from users
```

### 内连接写出 `inner`

最好是显性写出 `inner join`，而不是省略 `inner`

```sql
-- 好
select
    users.email,
    sum(charges.amount) as total_revenue
from users
inner join charges on users.id = charges.user_id

-- 不好
select
    users.email,
    sum(charges.amount) as total_revenue
from users
join charges on users.id = charges.user_id
```

### 对于 `join` 条件，按引用的表顺序排序到 `on` 之后

通过这样做，可以更容易确定连接是否导致结果呈扇形分布：

```sql
-- 好
select
    ...
from users
left join charges on users.id = charges.user_id
-- primary_key = foreign_key --> one-to-many --> fanout
  
select
    ...
from charges
left join users on charges.user_id = users.id
-- foreign_key = primary_key --> many-to-one --> no fanout

-- 不好
select
    ...
from users
left join charges on charges.user_id = users.id
```

### 单个连接条件应与 `join` 在同一行上

```sql
-- 好
select
    users.email,
    sum(charges.amount) as total_revenue
from users
inner join charges on users.id = charges.user_id
group by email

-- 不好
select
    users.email,
    sum(charges.amount) as total_revenue
from users
inner join charges
on users.id = charges.user_id
group by email
```

当有多个连接条件时，请将每个条件放在它们自己的缩进行中：

```sql
-- 好
select
    users.email,
    sum(charges.amount) as total_revenue
from users
inner join charges on 
    users.id = charges.user_id and
    refunded = false
group by email
```

### 大多数情况下尽量避免表名的别名

将表名 `users` 缩写为 `u`，将 `charges` 缩写为 `c`，这可能很诱人，但这最终会降低 SQL 的可读性：

```sql
-- 好
select
    users.email,
    sum(charges.amount) as total_revenue
from users
inner join charges on users.id = charges.user_id

-- 不好
select
    u.email,
    sum(c.amount) as total_revenue
from users u
inner join charges c on u.id = c.user_id
```

大多数情况下，最好是使用完整的表名。

有两个例外：
- 如果需要在同一个查询中多次连接到一个表，并且需要区分这几个之间的不同，那么就需要别名。
- 另外，如果表名很长或有歧义，可以使用别名（但仍然需要使用有意义的名称）：

```sql
-- 好：有意义的表别名
select
  companies.com_name,
  beacons.created_at
from stg_mysql_helpscout__helpscout_companies companies
inner join stg_mysql_helpscout__helpscout_beacons_v2 beacons on companies.com_id = beacons.com_id

-- 还行：没有表别名
select
  stg_mysql_helpscout__helpscout_companies.com_name,
  stg_mysql_helpscout__helpscout_beacons_v2.created_at
from stg_mysql_helpscout__helpscout_companies
inner join stg_mysql_helpscout__helpscout_beacons_v2 on stg_mysql_helpscout__helpscout_companies.com_id = stg_mysql_helpscout__helpscout_beacons_v2.com_id

-- 不好：不清晰的表别名
select
  c.com_name,
  b.created_at
from stg_mysql_helpscout__helpscout_companies c
inner join stg_mysql_helpscout__helpscout_beacons_v2 b on c.com_id = b.com_id
```

### 当存在 `join` 时，显性写出表名，否则省略表名

当没有涉及到 `join` 时，就不会对列来自哪个表产生歧义，因此可以省略表名：

```sql
-- 好
select
    id,
    name
from companies

-- 不好
select
    companies.id,
    companies.name
from companies
```

当涉及到 `join` 时，最好是显式的，这样就可以清楚地知道列来源：

```sql
-- 好
select
    users.email,
    sum(charges.amount) as total_revenue
from users
inner join charges on users.id = charges.user_id

-- 不好
select
    email,
    sum(amount) as total_revenue
from users
inner join charges on users.id = charges.user_id

```

### 总是重命名聚合和函数包装的参数

```sql
-- 好
select count(*) as total_users
from users

-- 不好
select count(*)
from users

-- 好
select timestamp_millis(property_beacon_interest) as expressed_interest_at
from hubspot.contact
where property_beacon_interest is not null

-- 不好
select timestamp_millis(property_beacon_interest)
from hubspot.contact
where property_beacon_interest is not null
```

### 显式写出布尔条件

```sql
-- 好
select * from customers where is_cancelled = true
select * from customers where is_cancelled = false

-- 不好
select * from customers where is_cancelled
select * from customers where not is_cancelled
```

### 定义列名别名时写出 `as` 

```sql
-- 好
select
    id,
    email,
    timestamp_trunc(created_at, month) as signup_month
from users

-- 不好
select
    id,
    email,
    timestamp_trunc(created_at, month) signup_month
from users
```

### 使用列名或列号进行分组，但不要同时使用两种

我更喜欢按列名分组，但按数字分组也是[极好的](https://blog.getdbt.com/write-better-sql-a-defense-of-group-by-1/)。

```sql
-- 好
select user_id, count(*) as total_charges
from charges
group by user_id

-- 好
select user_id, count(*) as total_charges
from charges
group by 1

-- 不好
select
    timestamp_trunc(created_at, month) as signup_month,
    vertical,
    count(*) as users_count
from users
group by 1, vertical
```

### 按名称分组时，利用在旁边添加的别名

```sql
-- 好
select
  timestamp_trunc(com_created_at, year) as signup_year,
  count(*) as total_companies
from companies
group by signup_year

-- 不好
select
  timestamp_trunc(com_created_at, year) as signup_year,
  count(*) as total_companies
from companies
group by timestamp_trunc(com_created_at, year)
```

### 分组的列放在前面

```sql
-- 好
select
  timestamp_trunc(com_created_at, year) as signup_year,
  count(*) as total_companies
from companies
group by signup_year

-- 不好
select
  count(*) as total_companies,
  timestamp_trunc(com_created_at, year) as signup_year
from mysql_helpscout.helpscout_companies
group by signup_year
```

### 调整 case/when 语句

每个 `when` 都应该独自一行（ `case` 行单独成行），并且应该缩进比 `case` 深一层，`then` 可以和 `when` 保持在同一行，也可以换行。

```sql
-- 好
select
    case
        when event_name = 'viewed_homepage' then 'Homepage'
        when event_name = 'viewed_editor' then 'Editor'
        else 'Other'
    end as page_name
from events

-- 也不错
select
    case
        when event_name = 'viewed_homepage'
            then 'Homepage'
        when event_name = 'viewed_editor'
            then 'Editor'
        else 'Other'            
    end as page_name
from events

-- 不好 
select
    case when event_name = 'viewed_homepage' then 'Homepage'
        when event_name = 'viewed_editor' then 'Editor'
        else 'Other'        
    end as page_name
from events
```

### 使用 CTE （公用表表达式），而不是子查询

CTE 避免了子查询，使查询更容易阅读和理解。

使用 CTE 时，将查询放在新行中。

在任何情况下使用 CTE 时，都要在最后一个 CTE 中使用 `select *`。通过这种方式，可以快速检查查询中使用的其他 CTE，以便调试结果。

结尾的 CTE 括号应该使用与 `with` 和 CTE 名称相同的缩进。

```sql
-- 好
with ordered_details as (

    select
        user_id,
        name,
        row_number() over (partition by user_id order by date_updated desc) as details_rank
    from billingdaddy.billing_stored_details

),

final as (

    select user_id, name
    from ordered_details
    where details_rank = 1

)

select * from final

-- 不好
select user_id, name
from (
    select
        user_id,
        name,
        row_number() over (partition by user_id order by date_updated desc) as details_rank
    from billingdaddy.billing_stored_details
) ranked
where details_rank = 1
```

### 使用有意义的 CTE 名称

```sql
-- 好
with ordered_details as (

-- 不好
with d1 as (
```

### 窗口函数

你可以把它单独放在一行上，或者根据它的长度把它分成多行：

```sql
-- 好
select
    user_id,
    name,
    row_number() over (partition by user_id order by date_updated desc) as details_rank
from billingdaddy.billing_stored_details

-- 好
select
    user_id,
    name,
    row_number() over (
        partition by user_id
        order by date_updated desc
    ) as details_rank
from billingdaddy.billing_stored_details
```

## Credits

这个风格指南的灵感部分来自于：

* [Fishtown Analytics' dbt Style Guide](https://github.com/fishtown-analytics/corp/blob/master/dbt_coding_conventions.md#sql-style-guide)
* [KickStarter's SQL Style Guide](https://gist.github.com/fredbenenson/7bb92718e19138c20591)
* [GitLab's SQL Style Guide](https://about.gitlab.com/handbook/business-ops/data-team/sql-style-guide/)

向 Peter Butler、Dan Wyman、Simon Ouderkirk、Alex Cano、Adam Stone、Brian Kim 和 Claire Carroll 致敬，感谢他们对本指南提供的反馈。
