---
title: "Java革命性ORM框架Jimmer简单介绍"
date: 2023-05-27T10:35:44+08:00
---

本文使用`Jimmer`的[官方用例](https://github.com/babyfish-ct/jimmer/tree/main/example/java/jimmer-sql)来介绍`Jimmer`的使用方法,`Jimmer`同时支持`Java`和`Kotlin`,本文使用`Java`来介绍,实际上`Kotlin`比`Java`使用起来更方便,这里为了方便大家理解,使用`Java`来介绍,本篇文章只是对`Jimmer`的一个简单介绍,更多的内容请参考[官方文档](https://babyfish-ct.github.io/jimmer/)

这里开始就不从实体类开始介绍了,这里简单的把用到的三张表之间的关系介绍一下:

- `BookStore`书店 可以拥有多个`Book`
- `Book`书 可以属于多个`BookStore`,可以有多个`Author`
- `Author`作者 可以拥有多个`Book`,多对多书与作者的关系.

## 查询

`Jimmer`可以配合`SpringData`(不是`SpringDataJPA`),但这里先介绍脱离`SpringData`的使用方法,但还是在`SpringBoot`环境下,这里使用`H2`内存数据库,`Jimmer`支持`H2`,`MySQL`,`PostgreSQL`,`Oracle`等数据库,这里使用`H2`内存数据库.

这里的查询都使用`Controller`来演示.

### 查询所有书店

`createQuery`就是创建一个查询,`select`就是选择要查询的字段,这里直接传入了`BookStoreTable`表示查询所有字段.

这里用到的`sql`就是使用`Jimmer`的`Sql`对象,这个对象是`Jimmer`的核心对象,所有的查询都是通过这个对象来实现的,使用`Spring`的注入方式注入`JSqlClient`对象.

```java
final BookStoreTable bookStore = BookStoreTable.$;//这里的$是一个静态方法,返回一个BookStoreTable对象
sql.createQuery(bookStore).select(bookStore).execute();
```

查询结果如下:

```json
[
  {
    "createdTime": "2023-05-27 11:00:37",
    "modifiedTime": "2023-05-27 11:00:37",
    "id": 1,
    "name": "O'REILLY",
    "website": null
  },
  {
    "createdTime": "2023-05-27 11:00:37",
    "modifiedTime": "2023-05-27 11:00:37",
    "id": 2,
    "name": "MANNING",
    "website": null
  }
]
```

### 指定查询字段

如何需要需要查询指定字段就可以这样,这里的`name`是`BookStoreTable`的一个字段,但这里的`Controller`返回的是`BookStore`对象,所以只好像上面的那样查询所有字段.

```java
sql.createQuery(bookStore).select(bookStore.name()).execute();
```

像上面的例子中如果我们非要查询指定字段又不想定义新的`DTO`对象,那么这种在`Jimmer`中也可以非常简单的实现,那就是使用`Jimmer`中的`Fetchr`

使用`BookStore`的`Fetchr`来指定查询的字段

```java
sql.createQuery(bookStore).select(bookStore.fetch(BookStoreFetcher.$.name())).execute();
```

查询结果如下:

```json
[
  {
    "id": 2,
    "name": "MANNING"
  },
  {
    "id": 1,
    "name": "O'REILLY"
  }
]
```

惊奇的发现,`Controller`的返回类型是`BookStore`,但是查询结果中只有`id`和`name`字段.

这里我把完整的`Controller`代码贴出来,`List`的类型就是`BookStore`的实体类,这就是`Jimmer`的强大之处,不需要定义`DTO`对象,就可以实现查询指定字段的功能.

```java
@GetMapping("/simpleList")
public List<BookStore> findSimpleStores() {
    final BookStoreTable bookStore = BookStoreTable.$;//这里的$是一个静态方法,返回一个BookStoreTable对象
    return sql.createQuery(bookStore).select(bookStore.fetch(BookStoreFetcher.$.name())).execute();
}
```

和实体类的`Table`一样,`Fetcher`也可以声明一个静态常量.

```java
private static final Fetcher<BookStore> SIMPLE_FETCHER = BookStoreFetcher.$.name();
```

这样就可以这样使用了.

```java
sql.createQuery(bookStore).select(bookStore.fetch(SIMPLE_FETCHER)).execute();
```

接下来详细介绍`Fetcher`的使用

查询所有标量字段,也就是非关联字段.

```java
private static final Fetcher<BookStore> DEFAULT_FETCHER = BookStoreFetcher.$.allScalarFields();//这里的allScalarFields()就是查询所有标量字段
```

在查询所有标量字段的基础上不查询`BookStore`的`name`字段

```java
private static final Fetcher<BookStore> DEFAULT_FETCHER = BookStoreFetcher.$.allScalarFields().name(false);//这里的name(false)就是不查询name字段
```

### 指定查询关联字段

像这样查询所有书店的所有书籍,并且查询书籍的所有作者,这样就可以使用`Fetcher`来实现,如果在使用传统`ORM`框架时,这里就需要定义一个`DTO`对象来接收查询结果,但是在`Jimmer`中,不需要定义`DTO`对象,就可以实现查询指定字段的功能,可能有读者会问了,没有`DTO`前端怎么接收数据呢,这里先剧透一下,`Jimmer`会根据后端写的`Fetcher`来生成前端的`DTO`,这里就不多说了,后面会详细介绍.

```java
private static final Fetcher<BookStore> WITH_ALL_BOOKS_FETCHER =
        BookStoreFetcher.$
                .allScalarFields()//查询所有标量字段
                .books(//查询关联字段
                        BookFetcher.$//书籍的Fetcher
                                .allScalarFields()//查询所有标量字段
                                .authors(//查询关联字段
                                        AuthorFetcher.$//作者的Fetcher
                                                .allScalarFields()//查询所有标量字段
                                )
                );
```

稍剧透一点,这里如果使用`Kotlin`来编写会更加简洁,因为`Kotlin`中的`DSL`特性

```kotlin
private val WITH_ALL_BOOKS_FETCHER = newFetcher(BookStore::class).by {
            allScalarFields()//查询所有标量字段
            books {//查询关联字段
                allScalarFields()//查询所有标量字段
                authors {//查询关联字段
                    allScalarFields()//查询所有标量字段
                }
            }
        }
```

这么一看`Kotlin`确实比`Java`简洁很多,但本篇文章还是介绍的是`Java`的使用方法.

### 指定查询条件和计算结果字段

如果需要查询书店中所有书籍的平均价格,那么就要查询书店中所有书籍的价格,然后计算平均值,这里先把查询的代码写出来,然后在介绍如何把计算结果字段添加到`Fetcher`中.

```java
sql.createQuery(bookStore)//这里的bookStore是一个BookStoreTable对象
    .where(bookStore.id().in(ids))//要查询的书店的id集合,也可以直接指定id,比如.eq(1L)
    .groupBy(bookStore.id())//按照书店的id分组
    .select(
            bookStore.id(),//查询书店的id
            bookStore.asTableEx().books(JoinType.LEFT).price().avg().coalesce(BigDecimal.ZERO)//查询书店中所有书籍的平均价格
    )
    .execute();//这样执行查询后,返回的结果就是书店的id和书店中所有书籍的平均价格,在Jimmer中会返回一个List<Tuple2<...>>类型的结果,其中Tuple元组的数量和查询的字段数量一致,这里就是2个字段,所以就是Tuple2
```

这里最后的`select`是查出了书店的 id 和书店中所有书籍的平均价格,`asTableEx()`是为了突破`Jimmer`的限制,`Jimmer`中的`Table`只能查询标量字段,而不能查询关联字段,这里的`asTableEx()`就是为了查询关联字段,`asTableEx()`的参数是`JoinType`,这里的`JoinType`是`LEFT`,表示左连接,如果不指定`JoinType`,默认是`INNER`,表示内连接.

这里的`avg()`是计算平均值的意思,`coalesce(BigDecimal.ZERO)`是为了防止计算结果为`null`,如果计算结果为`null`,那么就返回`BigDecimal.ZERO`.

这里介绍如何把计算结果字段添加到`Fetcher`中,这样就又引出了一个`Jimmer`的功能`计算属性`

### 计算属性

在`Jimmer`中如果要添加计算属性,那么就要实现`TransientResolver`接口,这里先把代码贴出来,然后再详细介绍.

```java
@Component
public class BookStoreAvgPriceResolver implements TransientResolver<Long, BigDecimal> {
    @Override
    public Map<Long, BigDecimal> resolve(Collection<Long> ids) {
        return null;
    }
}
```

这里的`ids`就是书店的 id 集合,这里的`resolve`方法就是计算书店中所有书籍的平均价格,这里的`Long`是书店的 id,`BigDecimal`是书店中所有书籍的平均价格,这里的`resolve`方法返回的`Map`的`key`就是书店的 id,`value`就是书店中所有书籍的平均价格.

接着配合上面写的查询代码,完成计算的代码

```java
BookStoreTable bookStore = BookStoreTable.$;
return sql.createQuery(bookStore)
        .where(bookStore.id().in(ids))
        .groupBy(bookStore.id())
        .select(
                bookStore.id(),
                bookStore.asTableEx().books(JoinType.LEFT).price().avg().coalesce(BigDecimal.ZERO)
        )
        .execute()//这里的execute()返回的结果是List<Tuple2<Long, BigDecimal>>类型的
        .stream()//这里把List转换成Stream
        .collect(
                Collectors.toMap(Tuple2::get_1, Tuple2::get_2)//这里把List<Tuple2<Long, BigDecimal>>转换成Map<Long, BigDecimal>
        );
```

这样一个`TransientResolver`的实现就完成了,接着就是把`TransientResolver`添加到实体类中

`Jimmer`中定义实体类是在接口中定义的

```java
@Transient(BookStoreAvgPriceResolver.class)//这里的BookStoreAvgPriceResolver.class就是上面写的计算属性的实现
BigDecimal avgPrice();//这里的avgPrice()就是计算属性,这里的BigDecimal就是计算属性的类型
```

这样就可以直接在`Fetcher`中查询计算属性了

```java
private static final Fetcher<BookStore> WITH_ALL_BOOKS_FETCHER =
            BookStoreFetcher.$
                    .allScalarFields()
                    .avgPrice()//这里就是查询计算属性
                    //...省略
```

接着看戏生成的`SQL`代码和查询结果,这里照样省略其他查询只关注标量字段和计算属性

```sql
select
    tb_1_.ID,
    coalesce(
        avg(tb_2_.PRICE), ? /* 0 */
    )
from BOOK_STORE tb_1_
left join BOOK tb_2_
    on tb_1_.ID = tb_2_.STORE_ID
where
    tb_1_.ID in (
        ? /* 1 */
    )
group by
    tb_1_.ID
```

```json
{
  "createdTime": "2023-05-27 12:04:39",
  "modifiedTime": "2023-05-27 12:04:39",
  "id": 1,
  "name": "O'REILLY",
  "website": null,
  "avgPrice": 58.5
}
```

## 定义实体类

在`Jimmer`中定义实体类是在接口中定义的,这里先把代码贴出来,然后再详细介绍.

### BookStore

```java
@Entity//这里的@Entity就是实体类
public interface BookStore extends BaseEntity {

    @Id//这里的@Id就是主键
    @GeneratedValue(strategy = GenerationType.IDENTITY)//这里的strategy = GenerationType.IDENTITY就是自增长
    long id();//这里的id()就是实体类的id

    @Key
    String name();//业务主键

    @Null//这里的@Null就是可以为null,建议使用Jetbrains的@Nullable
    String website();

    @OneToMany(mappedBy = "store", orderedProps = {
            @OrderedProp("name"),
            @OrderedProp(value = "edition", desc = true)
    })//这里的@OneToMany就是一对多,这里的mappedBy = "store"就是Book中的store字段,这里的orderedProps就是排序字段
    List<Book> books();

    @Transient(BookStoreAvgPriceResolver.class)//这里的BookStoreAvgPriceResolver.class就是上面写的计算属性的实现
    BigDecimal avgPrice();//这里的avgPrice()就是计算属性,这里的BigDecimal就是计算属性的类型
}
```

### Book

```java
@Entity
public interface Book extends TenantAware {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    long id();

    @Key//这里的@Key就是业务主键
    String name();

    @Key//和上面的name()一样,这里的@Key就是业务主键,表示name和edition的组合是唯一的
    int edition();

    BigDecimal price();

    @Nullable
    @ManyToOne
    BookStore store();

    @ManyToMany(orderedProps = {
            @OrderedProp("firstName"),
            @OrderedProp("lastName")
    })//这里的@ManyToMany就是多对多,这里的orderedProps就是排序字段
    @JoinTable(
            name = "BOOK_AUTHOR_MAPPING",//这里的name就是中间表的表名
            joinColumnName = "BOOK_ID",//这里的joinColumnName就是中间表的外键
            inverseJoinColumnName = "AUTHOR_ID"//这里的inverseJoinColumnName就是中间表的外键
    )
    List<Author> authors();
}
```

### Author

```java
@Entity
public interface Author extends BaseEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    long id();

    @Key
    String firstName();

    @Key
    String lastName();

    Gender gender();//这里的Gender就是枚举类型

    @ManyToMany(mappedBy = "authors", orderedProps = {
            @OrderedProp("name"),
            @OrderedProp(value = "edition", desc = true)
    })//这里的@ManyToMany就是多对多,这里的mappedBy = "authors"就是Book中的authors字段,这里的orderedProps就是排序字段
    List<Book> books();
}
```

```java
public enum Gender {

    @EnumItem(name = "M")//这里的name表示在数据库中存储的值
    MALE,

    @EnumItem(name = "F")
    FEMALE
}
```

如果使用过`Spring Data JPA`的话,这里的代码应该很熟悉,`Jimmer`中的实体类的关联关系和`Spring Data JPA`中的关联关系是一样的.

## 生成前端代码

还记得前面的剧透吗,现在开始正式介绍如何生成前端代码,这里先把生成的代码贴出来,然后再详细介绍.

### DTO

这里生成了好多根据`Controller`的返回类型的`Fetcher`生成的`DTO`,这里就不贴出来了,只贴一个`BookStoreDto`的代码.

```ts
export type BookStoreDto = {
  //只有查询书店的name
  "BookStoreService/SIMPLE_FETCHER": {
    readonly id: number
    readonly name: string
  }
  //查询书店的所有字段
  "BookStoreService/DEFAULT_FETCHER": {
    readonly id: number
    readonly createdTime: string
    readonly modifiedTime: string
    readonly name: string
    readonly website?: string
  }
  //查询书店的所有字段和书店中所有书籍的所有字段还有书籍的所有作者的所有字段
  "BookStoreService/WITH_ALL_BOOKS_FETCHER": {
    readonly id: number
    readonly createdTime: string
    readonly modifiedTime: string
    readonly name: string
    readonly website?: string
    readonly avgPrice: number //这里的avgPrice就是计算属性
    readonly books: ReadonlyArray<{
      readonly id: number
      readonly createdTime: string
      readonly modifiedTime: string
      readonly name: string
      readonly edition: number
      readonly price: number
      readonly authors: ReadonlyArray<{
        readonly id: number
        readonly createdTime: string
        readonly modifiedTime: string
        readonly firstName: string
        readonly lastName: string
        readonly gender: Gender
      }>
    }>
  }
}
```

### Controller

这里只看`BookStoreController`的主要请求

这里`Jimmer`把所有的`Controller`的请求都放在了一个`Controller`中,这里的`Controller`就是`BookStoreController`,这里的`BookStoreController`就是`BookStore`实体类的`Controller`,这里的`BookStoreController`的代码如下

```ts
async findComplexStoreWithAllBooks(options: BookStoreServiceOptions['findComplexStoreWithAllBooks']): Promise<
    BookStoreDto['BookStoreService/WITH_ALL_BOOKS_FETCHER'] | undefined
> {
    let _uri = '/bookStore/';
    _uri += encodeURIComponent(options.id);
    _uri += '/withAllBooks';
    return (await this.executor({uri: _uri, method: 'GET'})) as BookStoreDto['BookStoreService/WITH_ALL_BOOKS_FETCHER'] | undefined
}

async findSimpleStores(): Promise<
    ReadonlyArray<BookStoreDto['BookStoreService/SIMPLE_FETCHER']>
> {
    let _uri = '/bookStore/simpleList';
    return (await this.executor({uri: _uri, method: 'GET'})) as ReadonlyArray<BookStoreDto['BookStoreService/SIMPLE_FETCHER']>
}
async findStores(): Promise<
    ReadonlyArray<BookStoreDto['BookStoreService/DEFAULT_FETCHER']>
> {
    let _uri = '/bookStore/list';
    return (await this.executor({uri: _uri, method: 'GET'})) as ReadonlyArray<BookStoreDto['BookStoreService/DEFAULT_FETCHER']>
}
```

### 配置代码生成

需要再配置中指定生成代码的访问地址,因为`Jimmer`生成的前端代码是一个压缩包,访问这个地址就可以下载生成的源码了

```yaml
jimmer:
  client:
    ts:
      path: /ts.zip #这里的path就是访问地址
```

接着配置`Controller`的返回类型

```java
@GetMapping("/simpleList")
public List<@FetchBy("SIMPLE_FETCHER") BookStore> findSimpleStores() {
    final BookStoreTable bookStore = BookStoreTable.$;
    return sql.createQuery(bookStore).select(bookStore.fetch(SIMPLE_FETCHER)).execute();
}
```

这里使用了`FetchBy`注解,其中的值就是当前类的`Fetcher`常量,如果`Fetcher`不在当前的类下,可以指定注解中的`ownerType`来指定`Fetcher`所在的类.

好了,`Jimmer`的基本使用就介绍完了,如果想了解更多的使用方法,可以查看`Jimmer`的[文档](https://babyfish-ct.github.io/jimmer/),也可以观看`Jimmer`作者录制的视频教程[Jimmer0.6x: 前后端免对接+spring starter，让REST媲美GraphQL](https://www.bilibili.com/video/BV1wD4y1L7xr),[Jimmer-0.7之计算属性](https://www.bilibili.com/video/BV1NM4y1C78H)