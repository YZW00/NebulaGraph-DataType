需求背景

List / Set 是重要的基础数据类型，在nebula最新版中，缺少对List/Set数据类型的支持，期望增加对List/Set 数据类型的支持，以满足各种业务场景的需求；该需求也参与了开源之夏2024的活动；

实现目标

期望实现对 List /SET的数据支持，包括：

支持属性定义为 List /Set 数据类型；

支持存储时以 List /Set 数据类型进行存储；

支持查询时能对 List /Set 数据类型进行过滤；

支持查询时能返回 List /Set 数据类型的属性；

支持查询时能对 List /Set 数据类型进行运算； List 优先级最高，Set 优先级次之；

详细需求

支持属性定义为 List /Set 数据类型； 涉及语法包括：
create tag
alter tag
desc tag
create edge
alter edge
desc edge
show create tag/edge

元素类型限制

NebulaGraph 当前支持的数据类型，List应该都能支持包括：

（高）数值型：INT8、INT16、INT32、INT64、FLOAT、DOUBLE（含科学计数法）
（中）布尔型：BOOL
（高）字符串型：string、fix_string
（中）日期时间：DATE、TIME、DATETIME、TIMESTAMP和DURATION
（低）地理空间：Point、LineString、Polygon

本次开源之夏优先实现整形（int）、浮点型（float）、字符串（string）的定义，List\Set本次仅支持一维和二维，不支持更高维嵌套；即LIST[LIST,LIST,LIST，……]；或者 SET[SET,SET,SET,……]或者 LIST[SET,SET,SET,……]或者SET[LIST,LIST,LIST,……]。不支持以List /Set 属性做为索引；

List /Set 数据类型支持 NULL 和 Default 整形默认值为0，浮点型默认值为0.0，字符串默认值""

支持存储时以 List / Set 数据类型进行存储；
支持以 List 数据类型进行存储； 涉及语法包括：

insert vertex
update vertex
upsert vertex
insert edge
update edge
upsert edge
在TAG的INSERT语句中写入List

插入一维list;创建player属性ids必须是一维的list<int>
INSERT VERTEX player(name, age, ids) VALUES "player100":("jimmy", 27, [1, 2, 3]);

插入二维list；创建player属性ids必须是二维的list<list<int>>
INSERT VERTEX player(name, age, ids) VALUES "player100":("jimmy", 27, [[1, 2, 3], [4, 5], [6]]);
在TAG的插入语句中写入Set
这里比较特殊的情况，由于List在命令行中可以直接用[]来表示，而Set需要区分开，这里我们设计由<>来表示Set容器

插入一维set;创建player属性ids必须是一维的Set<int>
INSERT VERTEX player(name, age, ids) VALUES "player100":("jimmy", 27, <1, 2, 3>);

插入二维set；创建player属性ids必须是二维的Set<Set<int>>
INSERT VERTEX player(name, age, ids) VALUES "player100":("jimmy", 27, <<1, 2, 3>, <4, 5>, <6>>);
set的插入数据中如果有重复数据，应该返回报错

INSERT VERTEX player(name, age, ids) VALUES "player100":("jimmy", 27, <1, 2, 2>);

该命令返回报错，并提示重复的元素
在TAG的UPDATE语句中修改List\Set
关于 update 中，支持对List/Set 属性的字段进行增删改；


支持查询时能对List/Set数据类型进行过滤；
支持 where 条件中以对属性类型为 List / Set 的进行过滤； 应尽可能考虑有良好的性能 举例：

MATCH (v:player) WHERE ANY(x IN v.player.hobby WHERE x == "basketball") RETURN v
支持查询时能返回List/Set数据类型的属性；
