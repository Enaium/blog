---
title: "Jimmer VS MyBatisPlus查询自关联表"
date: 2023-05-30T09:48:17+08:00
---

本文是对`Jimmer`文档中[对象抓取器-自关联递归抓取](https://babyfish-ct.github.io/jimmer/zh/docs/jimmer-sql/query/fetcher#%E8%87%AA%E5%85%B3%E8%81%94%E9%80%92%E5%BD%92%E6%8A%93%E5%8F%96)部分的介绍,之后会对比`MyBatisPlus`的查询自关联表的能力。

> 对象抓取器是 jimmer-sql 一个非常强大的特征，具备可媲美 GraphQL 的能力。
> 即使用户不采用任何 GraphQL 相关的技术栈，也能在 SQL 查询层面得到和 GraphQL 相似的对象图查询能力。

## 准备数据库和实体类

```sql
create table tree_node(
    node_id bigint not null,
    name varchar(20) not null,
    parent_id bigint
);
alter table tree_node
    add constraint pk_tree_node
        primary key(node_id);
alter table tree_node
    add constraint uq_tree_node
        unique(parent_id, name);
alter table tree_node
    add constraint fk_tree_node__parent
        foreign key(parent_id)
            references tree_node(node_id);

insert into tree_node(
    node_id, name, parent_id
) values
    (1, 'Home', null),
        (2, 'Food', 1),
            (3, 'Drinks', 2),
                (4, 'Coca Cola', 3),
                (5, 'Fanta', 3),
            (6, 'Bread', 2),
                (7, 'Baguette', 6),
                (8, 'Ciabatta', 6),
        (9, 'Clothing', 1),
            (10, 'Woman', 9),
                (11, 'Casual wear', 10),
                    (12, 'Dress', 11),
                    (13, 'Miniskirt', 11),
                    (14, 'Jeans', 11),
                (15, 'Formal wear', 10),
                    (16, 'Suit', 15),
                    (17, 'Shirt', 15),
            (18, 'Man', 9),
                (19, 'Casual wear', 18),
                    (20, 'Jacket', 19),
                    (21, 'Jeans', 19),
                (22, 'Formal wear', 18),
                    (23, 'Suit', 22),
                    (24, 'Shirt', 22)
;
```

```java
@Entity
public interface TreeNode {

    @Id
    @Column(name = "NODE_ID")
    long id();

    String name();

    @Null
    @ManyToOne
    TreeNode parent();

    @OneToMany(mappedBy = "parent")
    List<TreeNode> childNodes();
}
```

## 指定查询的深度

我们可以看到，这是一个自关联的表，每个节点都有一个父节点，也可以有多个子节点。

使用 Jimmer 的`Fetcher`功能，我们可以很容易的查询出这个表的所有节点,并且可以很容易的控制查询的深度,还有条件查询。

```java
TreeNodeTable node = TreeNodeTable.$;

List<TreeNode> treeNodes = sqlClient
    .createQuery(node)//创建一个查询
    .where(node.parent().isNull())//查询条件,这里查询出所有的根节点,也就是parent_id为null的节点
    .select(//查询的字段
        node.fetch(
            TreeNodeFetcher.$
                .name()//查询节点的名称
                .childNodes(
                    TreeNodeFetcher.$.name(),//查询子节点的名称
                    it -> it.depth(2)//查询子节点的深度,这里查询2层
                )
        )
    )
    .execute();
```

如果你使用`Kotlin`,那么你可以这样写

```kotlin
val treeNodes = sqlClient
    .createQuery(TreeNode::class) {
        where(table.parent.isNull())//查询条件,这里查询出所有的根节点,也就是parent_id为null的节点
        select(
            table.fetchBy {
                allScalarFields()//查询节点的所有字段
                childNodes({
                    depth(2)//查询子节点的深度,这里查询2层
                }) {
                    allScalarFields()//查询子节点的所有字段
                }
            }
        )
    }
    .execute()
```

### 生成的 SQL 语句

#### 第 0 层

```sql
select
    tb_1_.NODE_ID,
    tb_1_.NAME
from TREE_NODE as tb_1_
where
    tb_1_.PARENT_ID is null
```

#### 第 1 层

```sql
select

    tb_1_.PARENT_ID,

    tb_1_.NODE_ID,
    tb_1_.NAME

from TREE_NODE as tb_1_
where
    tb_1_.PARENT_ID in (?)
```

#### 第 2 层

```sql
select
    tb_1_.PARENT_ID,
    tb_1_.NODE_ID,
    tb_1_.NAME
from TREE_NODE as tb_1_
where
    tb_1_.PARENT_ID in (?, ?)
```

### 查询的结果

```json
{
  "id": 1,
  "name": "Home",
  "childNodes": [
    {
      "id": 9,
      "name": "Clothing",
      "childNodes": [
        { "id": 18, "name": "Man" },
        { "id": 10, "name": "Woman" }
      ]
    },
    {
      "id": 2,
      "name": "Food",
      "childNodes": [
        { "id": 6, "name": "Bread" },
        { "id": 3, "name": "Drinks" }
      ]
    }
  ]
}
```

## 查询无限层级的树

如果你想查询无限层级的树,那么你可以这样写

```java
TreeNodeTable node = TreeNodeTable.$;

List<TreeNode> treeNodes = sqlClient
    .createQuery(node)
    .where(node.parent().isNull())
    .select(
        node.fetch(
            TreeNodeFetcher.$
                .name()
                .childNodes(
                    TreeNodeFetcher.$.name(),
                    it -> it.recursive()//查询无限层级的树,这里不需要指定深度,也就是把depth()方法去掉换成recursive()方法
                )
        )
    )
    .execute();
```

```kotlin
val treeNodes = sqlClient
    .createQuery(TreeNode::class) {
        where(table.parent.isNull())
        select(
            table.fetchBy {
                allScalarFields()
                childNodes({
                    recursive()
                }) {
                    allScalarFields()
                }
            }
        )
    }
    .execute()
```

### 生成的 SQL 语句

#### 第 0 层

```sql
select
    tb_1_.NODE_ID,
    tb_1_.NAME
from TREE_NODE as tb_1_
where
    tb_1_.PARENT_ID is null
```

#### 第 1 层

```sql
select

    tb_1_.PARENT_ID,

    tb_1_.NODE_ID,
    tb_1_.NAME

from TREE_NODE as tb_1_
where
    tb_1_.PARENT_ID in (?)
```

#### 第 2 层

```sql
select
    tb_1_.PARENT_ID,
    tb_1_.NODE_ID,
    tb_1_.NAME
from TREE_NODE as tb_1_
where
    tb_1_.PARENT_ID in (?, ?)
```

#### 第 3 层

```sql
select
    tb_1_.PARENT_ID,
    tb_1_.NODE_ID,
    tb_1_.NAME
from TREE_NODE as tb_1_
where
    tb_1_.PARENT_ID in (?, ?, ?, ?)
```

#### 第 4 层

```sql
select
    tb_1_.PARENT_ID,
    tb_1_.NODE_ID,
    tb_1_.NAME
from TREE_NODE as tb_1_
where
    tb_1_.PARENT_ID in (?, ?, ?, ?, ?, ?, ?, ?)
```

#### 第 5 层

```sql
select
    tb_1_.PARENT_ID,
    tb_1_.NODE_ID,
    tb_1_.NAME
from TREE_NODE as tb_1_
where
    tb_1_.PARENT_ID in (?, ?, ?, ?, ?, ?, ?, ?, ?)
```

### 查询结果

```json
{
  "id": 1,
  "name": "Home",
  "childNodes": [
    {
      "id": 9,
      "name": "Clothing",
      "childNodes": [
        {
          "id": 18,
          "name": "Man",
          "childNodes": [
            {
              "id": 19,
              "name": "Casual wear",
              "childNodes": [
                { "id": 20, "name": "Jacket", "childNodes": [] },
                { "id": 21, "name": "Jeans", "childNodes": [] }
              ]
            },
            {
              "id": 22,
              "name": "Formal wear",
              "childNodes": [
                { "id": 24, "name": "Shirt", "childNodes": [] },
                { "id": 23, "name": "Suit", "childNodes": [] }
              ]
            }
          ]
        },
        {
          "id": 10,
          "name": "Woman",
          "childNodes": [
            {
              "id": 11,
              "name": "Casual wear",
              "childNodes": [
                { "id": 12, "name": "Dress", "childNodes": [] },
                { "id": 14, "name": "Jeans", "childNodes": [] },
                { "id": 13, "name": "Miniskirt", "childNodes": [] }
              ]
            },
            {
              "id": 15,
              "name": "Formal wear",
              "childNodes": [
                { "id": 17, "name": "Shirt", "childNodes": [] },
                { "id": 16, "name": "Suit", "childNodes": [] }
              ]
            }
          ]
        }
      ]
    },
    {
      "id": 2,
      "name": "Food",
      "childNodes": [
        {
          "id": 6,
          "name": "Bread",
          "childNodes": [
            { "id": 7, "name": "Baguette", "childNodes": [] },
            { "id": 8, "name": "Ciabatta", "childNodes": [] }
          ]
        },
        {
          "id": 3,
          "name": "Drinks",
          "childNodes": [
            { "id": 4, "name": "Coca Cola", "childNodes": [] },
            { "id": 5, "name": "Fanta", "childNodes": [] }
          ]
        }
      ]
    }
  ]
}
```

## 每个查询的节点是否递归

如果你想每个查询的节点是否递归,那么你可以这样写

```java
TreeNodeTable node = TreeNodeTable.$;

List<TreeNode> treeNodes = sqlClient
    .createQuery(node)
    .where(node.parent().isNull())
    .select(
        node.fetch(
            TreeNodeFetcher.$
                .name()
                .childNodes(
                    TreeNodeFetcher.$.name(),
                    it -> it.recursive(args ->
                        !args.getEntity().name().equals("Clothing")//每个查询的节点是否递归,这里可以根据实体的属性来判断是否递归
                    )
                )
        )
    )
    .execute();
```

```kotlin
val treeNodes = sqlClient
    .createQuery(TreeNode::class) {
        where(table.parent.isNull())
        select(

            table.fetchBy {
                allScalarFields()
                childNodes({
                    recursive {
                        entity.name != "Clothing"//每个查询的节点是否递归,这里可以根据实体的属性来判断是否递归
                    }
                }) {
                    allScalarFields()
                }
            }
        )
    }
    .execute()
```

这样就可以实现每个查询的节点是否递归了

## 使用 MybatisPlus 来查询树形结构

### 定义实体

```java
@Data
@TableName("tree_node")
public class TreeNode {
    @TableId
    private Long nodeId;
    private String name;
    @TableField(exist = false)
    private List<TreeNode> childNodes;
}
```

### 定义 Mapper

```java
@Mapper
public interface TreeNodeMapper extends BaseMapper<TreeNode> {
}
```

### 查询树形结构的 Service

```java
@Service
@AllArgsConstructor
public class TreeNodeService {
    private final TreeNodeMapper treeNodeMapper;

    public List<TreeNode> getTree() {
        // 查询根节点列表
        List<TreeNode> rootNodes = selectRoots();

        // 遍历根节点，递归查询每个节点的子孙节点
        for (TreeNode rootNode : rootNodes) {
            this.getChildren(rootNode);
        }

        return rootNodes;
    }

    private void getChildren(TreeNode node) {
        // 查询子节点
        List<TreeNode> children = selectByParentId(node.getNodeId());
        // 遍历子节点，递归查询子节点的子孙节点
        for (TreeNode child : children) {
            this.getChildren(child);
        }
        node.setChildNodes(children);
    }
    public List<TreeNode> selectRoots() {
        QueryWrapper<TreeNode> wrapper = new QueryWrapper<>();
        wrapper.isNull("parent_id");// 查询根节点，parent_id为null
        return treeNodeMapper.selectList(wrapper);
    }

    public List<TreeNode> selectByParentId(Long parentId) {
        QueryWrapper<TreeNode> wrapper = new QueryWrapper<>();
        wrapper.eq("parent_id", parentId);// 查询子节点，parent_id为当前节点的id
        return treeNodeMapper.selectList(wrapper);
    }
}
```

#### 查询结果

```json
{
  "nodeId": 1,
  "name": "Home",
  "childNodes": [
    {
      "nodeId": 9,
      "name": "Clothing",
      "childNodes": [
        {
          "nodeId": 18,
          "name": "Man",
          "childNodes": [
            {
              "nodeId": 19,
              "name": "Casual wear",
              "childNodes": [
                { "nodeId": 20, "name": "Jacket", "childNodes": [] },
                { "nodeId": 21, "name": "Jeans", "childNodes": [] }
              ]
            },
            {
              "nodeId": 22,
              "name": "Formal wear",
              "childNodes": [
                { "nodeId": 24, "name": "Shirt", "childNodes": [] },
                { "nodeId": 23, "name": "Suit", "childNodes": [] }
              ]
            }
          ]
        },
        {
          "nodeId": 10,
          "name": "Woman",
          "childNodes": [
            {
              "nodeId": 11,
              "name": "Casual wear",
              "childNodes": [
                { "nodeId": 12, "name": "Dress", "childNodes": [] },
                { "nodeId": 14, "name": "Jeans", "childNodes": [] },
                { "nodeId": 13, "name": "Miniskirt", "childNodes": [] }
              ]
            },
            {
              "nodeId": 15,
              "name": "Formal wear",
              "childNodes": [
                { "nodeId": 17, "name": "Shirt", "childNodes": [] },
                { "nodeId": 16, "name": "Suit", "childNodes": [] }
              ]
            }
          ]
        }
      ]
    },
    {
      "nodeId": 2,
      "name": "Food",
      "childNodes": [
        {
          "nodeId": 6,
          "name": "Bread",
          "childNodes": [
            { "nodeId": 7, "name": "Baguette", "childNodes": [] },
            { "nodeId": 8, "name": "Ciabatta", "childNodes": [] }
          ]
        },
        {
          "nodeId": 3,
          "name": "Drinks",
          "childNodes": [
            { "nodeId": 4, "name": "Coca Cola", "childNodes": [] },
            { "nodeId": 5, "name": "Fanta", "childNodes": [] }
          ]
        }
      ]
    }
  ]
}
```

### 查询树形结构的 Service 并指定查询深度

```java
@Service
@AllArgsConstructor
public class TreeNodeService {
    private final TreeNodeMapper treeNodeMapper;

    public List<TreeNode> getTree(int depth) {
        // 查询根节点列表
        List<TreeNode> rootNodes = selectRoots();

        // 遍历根节点，递归查询每个节点的子孙节点
        for (TreeNode rootNode : rootNodes) {
            this.getChildren(rootNode, depth, 0);
        }

        return rootNodes;
    }

    private void getChildren(TreeNode node, int maxDepth, int currentDepth) {
        if (currentDepth >= maxDepth) {
            // 当前深度达到最大深度，终止递归并返回结果
            node.setChildNodes(Collections.emptyList());
            return;
        }

        // 查询子节点
        List<TreeNode> children = selectByParentId(node.getNodeId());
        // 遍历子节点，递归查询子节点的子孙节点
        for (TreeNode child : children) {
            this.getChildren(child, maxDepth, currentDepth + 1);
        }
        node.setChildNodes(children);
    }

    public List<TreeNode> selectRoots() {
        QueryWrapper<TreeNode> wrapper = new QueryWrapper<>();
        wrapper.isNull("parent_id");
        return treeNodeMapper.selectList(wrapper);
    }

    public List<TreeNode> selectByParentId(Long parentId) {
        QueryWrapper<TreeNode> wrapper = new QueryWrapper<>();
        wrapper.eq("parent_id", parentId);
        return treeNodeMapper.selectList(wrapper);
    }
}
```

#### 查询结果

```json
{
  "nodeId": 1,
  "name": "Home",
  "childNodes": [
    {
      "nodeId": 9,
      "name": "Clothing",
      "childNodes": [
        { "nodeId": 18, "name": "Man", "childNodes": [] },
        { "nodeId": 10, "name": "Woman", "childNodes": [] }
      ]
    },
    {
      "nodeId": 2,
      "name": "Food",
      "childNodes": [
        { "nodeId": 6, "name": "Bread", "childNodes": [] },
        { "nodeId": 3, "name": "Drinks", "childNodes": [] }
      ]
    }
  ]
}
```

### 查询树形结构的 Service 并指定查询深度和查询条件

不好意思，这个功能我还没想好怎么用 MybatisPlus 实现，所以这里就不写了。

## 总结

这么一对比，使用`MybatisPlus`的代码量确实多了不少并且很复杂,又是递归又是递归计数等等,而`Jimmer`使用了`Fetcher`就会更容易的查出所有多层节点,并且代码量也非常少