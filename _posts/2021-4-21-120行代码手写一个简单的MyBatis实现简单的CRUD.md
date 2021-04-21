---
layout: post
title: "120行代码手写一个简单的MyBatis实现简单的CRUD"
data: 2021-4-21 20:57
categories: java
---

不用XML只用注解

首先需要创建6个注解

SQL用于输入SQL语句

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface SQL {
    String[] value();
}
```

用来表示这个方法是Update

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Update {
}
```


用来表示这个方法是Select

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Select {
}
```

用来表示这个方法是Insert

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Insert {
}
```

用来表示这个方法是Delete

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Delete {
}
```

用来表示方法参数名

```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface Param {
    String value();
}
```

好了注解写完了

开始写主类

用map实现简单的配置，然后读取配置连接数据库，然后程序关闭的时候关闭连接。

```java
public class Satis {
    private final Statement statement;

    public Satis(Map<String, String> config) throws Exception {
        Class.forName(config.get("driver"));
        Connection connection = DriverManager.getConnection(config.get("url"), config.get("username"), config.get("password"));
        statement = connection.createStatement();
        Runtime.getRuntime().addShutdownHook(new Thread(() -> {
            try {
                statement.close();
                connection.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }));
    }
}
```

创建`getMapper`方法。

```java
public <T> T getMapper(Class<?> mapper) {

}
```

使用动态代理。

```java
public <T> T getMapper(Class<?> mapper) {
    Object instance = Proxy.newProxyInstance(mapper.getClassLoader(), new Class<?>[]{mapper}, new InvocationHandler() {
        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            return null;
        }
    };
    return (T) instance;
}
```

遍历方法获取方法是否有`SQL`这个注解。

```java
@Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    if (method.isAnnotationPresent(SQL.class)) {

    }
    return null;
}
```

创建4个方法，分别获取方法是否有这几个注解。

```java
private boolean isSelect(Method method) {
    return method.isAnnotationPresent(Select.class);
}
private boolean isDelete(Method method) {
    return method.isAnnotationPresent(Delete.class);
}
private boolean isInsert(Method method) {
    return method.isAnnotationPresent(Insert.class);
}
private boolean isUpdate(Method method) {
    return method.isAnnotationPresent(Update.class);
}
```

遍历`SQL`注解的值。

```java
@Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    if (method.isAnnotationPresent(SQL.class)) {
        SQL sql = method.getAnnotation(SQL.class);
        for (String value : sql.value()) {
            
        }
    }
    return null;
}
```

获取方法上是否有`Select`或`Delete`注解，是的话把`SQL`参数的值(获取参数的`Param`注解的值)替换为调用方法时的参数。

```java
@Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    if (method.isAnnotationPresent(SQL.class)) {
        SQL sql = method.getAnnotation(SQL.class);
        for (String value : sql.value()) {
            if (isSelect(method) || isDelete(method)) {
                int index = 0;
                for (Parameter parameter : method.getParameters()) {
                    if (parameter.isAnnotationPresent(Param.class)) {
                        Param param = parameter.getAnnotation(Param.class);
                        value = value.replace("#{" + param.value() + "}", args[index].toString());
                    }
                    index++;
                }
            }
        }
    }
    return null;
}
```

如果是`Insert`或`Update`注解，获取第一个参数，获取这个参数的所有方法并获取值。

```java
@Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    if (method.isAnnotationPresent(SQL.class)) {
        SQL sql = method.getAnnotation(SQL.class);
        for (String value : sql.value()) {
            if (isInsert(method) || isUpdate(method)) {
                Class<?> parameterType = method.getParameterTypes()[0];
                for (Field declaredField : parameterType.getDeclaredFields()) {
                    value = value.replace("#{" + declaredField.getName() + "}", "\"" + parameterType.getMethod(getGetMethodName(declaredField.getName())).invoke(args[0]).toString() + "\"");
                }
            }
        }
    }
    return null;
}
```

简单的获取方法名的方法。

```java
private String getSetMethodName(String name) {
    return "set" + name.substring(0, 1).toUpperCase(Locale.ROOT) + name.substring(1);
}
private String getGetMethodName(String name) {
    return "get" + name.substring(0, 1).toUpperCase(Locale.ROOT) + name.substring(1);
}
```

执行SQL语句，如果有返回值那就返回实例化。

```java
@Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    if (method.isAnnotationPresent(SQL.class)) {
        SQL sql = method.getAnnotation(SQL.class);
        for (String value : sql.value()) {
            String typeName = method.getGenericReturnType().getTypeName();
            ResultSet resultSet = statement.executeQuery(value);
            if (!typeName.equals("void")) {
                return toEntity(resultSet, typeName);
            }
        }
    }
    return null;
}
```

将返回值实例化。

```java
private <T> T toEntity(ResultSet resultSet, String className) throws SQLException, NoSuchMethodException, InvocationTargetException, IllegalAccessException, InstantiationException, ClassNotFoundException {
    boolean list = className.contains("<");//是不是列表
    if (list) {
        className = className.substring(className.indexOf("<") + 1, className.lastIndexOf(">"));//获取列表的泛型
    }
    Class<?> klass = Class.forName(className);
    HashMap<String, Class<?>> fieldNameList = new HashMap<>();
    for (int i = 0; i < resultSet.getMetaData().getColumnCount(); i++) {//获取字段和参数
        fieldNameList.put(resultSet.getMetaData().getColumnName(i + 1), Class.forName(resultSet.getMetaData().getColumnClassName(i + 1)));
    }
    List<Object> objectList = new ArrayList<>();
    while (resultSet.next()) {
        Object instance = klass.newInstance();//实例化
        for (Map.Entry<String, Class<?>> entry : fieldNameList.entrySet()) {//遍历字段
            //调用set方法赋值
            klass.getMethod(getSetMethodName(entry.getKey()), entry.getValue()).invoke(instance, resultSet.getObject(entry.getKey(), entry.getValue()));
        }
        objectList.add(instance);//添加到列表
    }
    resultSet.close();//关闭
    if (objectList.isEmpty()) {
        return null;//如果列表为空返回null
    }
    return list ? (T) objectList : (T) objectList.get(0);//判断是否为列表，如果是直接返回，如果不是取第一个
}
```

好了已经写完了。

现在来测试

实体

```java
@Data
@AllArgsConstructor
public class AccountEntity {
    private Long id;
    private String name;
    private Integer age;

    public AccountEntity() {//用了lombok就不会自动生成无参构造方法了

    }
}
```

接口

```java
public interface AccountMapper {
    @Select
    @SQL("select * from account")
    List<AccountEntity> getAll();

    @Select
    @SQL("select * from account where id = #{id}")
    AccountEntity getById(@Param("id") Serializable id);

    @Delete
    @SQL("delete from account where id = #{id}")
    void deleteById(@Param("id") Serializable id);

    @Insert
    @SQL("insert into account(id, name, age) values (#{id}, #{name}, #{age})")
    void insert(AccountEntity accountEntity);

    @Update
    @SQL("update account set name=#{name}, age=#{age} where id=#{id}")
    void update(AccountEntity accountEntity);
}
```

主类

```java
public class Main {
    public static void main(String[] args) throws Exception {
        Satis satis = new Satis(ImmutableMap.of(
                "url", "jdbc:mariadb://localhost:3306/enaium?useUnicode=true&characterEncoding=UTF-8",
                "driver", "org.mariadb.jdbc.Driver",
                "username", "root",
                "password", "root"));//使用guava
        AccountMapper mapper = satis.getMapper(AccountMapper.class);//获取Mapper
        System.out.println(mapper.getById(1));//获取ID为1的Account
        mapper.getAll().forEach(System.out::println);//获取所有Account
        mapper.insert(new AccountEntity(0L, "Enaium", 1));//插入一个新的Account
        mapper.getAll().forEach((it) -> {//把所有Account的Age都改为0
            it.setAge(0);
            mapper.update(it);
        });
    }
}
```

[源码](https://github.com/Enaium-Learn/SimpleBatis/)