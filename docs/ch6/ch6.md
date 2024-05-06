## 范式与反范式

### 数据库范式

> 请回顾《数据管理基础》关于范式的内容。

### 数据库反范式 —— 考虑引入可控制的冗余

- 规范化的结果是一个在结构上一致，且拥有最少冗余的逻辑数据库设计

- 但未必性能最优

带来的结果

- 实现更加复杂（手动维护更新）

- 降低灵活性

- 加快了读取，降低了更新

反范式的动作：增加冗余，主要的动作就是复制

反范式的核心目标：减少连接次数

## 数据库反范式模式

1. 合并`1:1`关系

2. 复制`1:*`关系的非Key、FK及值

3. 复制`*:*`关系的属性

4. 引入重复组

5. 创建提取临时表

### 模式1：合并1:1关系

为什么不直接合并呢？

> 学院给每个年龄大于等于40岁的老师发一辆SU7， Car表和Teacher表属于**部分参与**情况——不是所有的老师都有车。

只有在需要降低连接次数的时候可以合并，会存在大量$NULL$（引入空值很麻烦）

> 万一学校有些车不发给老师呢（公共用车🤭）这个时候数据库可以合并吗 
>
> 不一定。设置一个教师的Id为000，表示这是学院的公共车。~~（当然这大概率是很糟糕的设计😂）~~
>
> 还可以添加一个字段。（虽然也不太好）

### 模式2：复制`1:*`关系的非Key、FK及值

### 模式3：复制`*:*`关系的属性

### 模式4：引入重复组

不满足4范式

- 地址、电话
- 静态、数量较小

## 数据表存树状结构

在数据库设计（单表）中，树通常三种模型

- Adjacency model-邻接模型

- Materialized path model-物化路径模型

- Nested set model-嵌套集合模型

### 邻接模型

<img src="D:\Myfl\DatabaseDevelopment\docsify\DataBaseDev2024\docs\ch6\ch6.assets\image-20240506141544960.png" alt="image-20240506141544960" style="zoom:67%;" />

<center>表的每一行描述一个部队，parent_id指向树中的上级部队</center>

**问题：**

1. 没有描述子节点的前后顺序（无法描述族谱）。

2. `parent_id`存在多处，不满足数据库设计归一化原则中的<u>一地</u>。

3. 存储的不一定是树，还可能是环等。只能保证最多有一个父节点，需要由应用程序保证。不满足归一化原则。

   > 数据库设计的归一化原则：一事、一地、一次(one simple fact、in one place、on one time)

### 物化路径模型

就像我们写论文的标号一样，`1.1  1.1.2 2.1`

<img src="D:\Myfl\DatabaseDevelopment\docsify\DataBaseDev2024\docs\ch6\ch6.assets\image-20240506143341373.png" alt="image-20240506143341373" style="zoom:67%;" />

表中有两个索引，在materialized_path上的唯一性索引以及在commander上的索引，正确的设计应该增加id字段。

示例：

<img src="D:\Myfl\DatabaseDevelopment\docsify\DataBaseDev2024\docs\ch6\ch6.assets\image-20240506143444490.png" alt="image-20240506143444490" style="zoom:67%;" />

**问题：**仍然没解决归一化的问题，`3.1.3.1`中仍然包含了`3.1`。

但是不会把树变成图了，能表示子节点顺序而且对插入友好。

### 嵌套集合模型

<img src="D:\Myfl\DatabaseDevelopment\docsify\DataBaseDev2024\docs\ch6\ch6.assets\image-20240506144109332.png" alt="image-20240506144109332" style="zoom: 67%;" />

<center>每个结点都有一个左编号和右编号，有点像DFS序</center>

- 只要节点编号在$X$的左编号和右编号之间，都是$X$的子节点。
- 对删除节点友好
- **问题**：在某些节点下增加一个节点，要修改的节点太多了😂。一个不太好的解决办法，给数据之间留一点冗余，如1->10。

## 树状结构查询

- 法国将军Dominique	Vandamme指挥哪些部队，以缩排方式或简单列表的方式显示他们。注意，所有的commander字段都构建了索引（简称Vandamme查询）

- Scottish	Highlanders的每个团各属于哪个部队（自底向上的查询）。在部队的名称（description字段）上没有索引，唯一的方法是在description字段中查找“Highland”字符串，在没有任何全文索引的情况下，这个问题简称highland问题

### 自顶向下查询：Vandamme查询

#### 邻接模式

Oracle四行解决：

<img src="D:\Myfl\DatabaseDevelopment\docsify\DataBaseDev2024\docs\ch6\ch6.assets\image-20240506151135967.png" alt="image-20240506151135967" style="zoom:50%;" />

MySQL：递归

太痛苦了，如果我不想学，也没有大冤种帮我写怎么办？从数据库设计下手

数据表：$(id, pid, dis)$，空间换时间，只要根据pid查找就行了

#### 物化路径模型

前缀查询

#### 嵌套集合模型

范围查询