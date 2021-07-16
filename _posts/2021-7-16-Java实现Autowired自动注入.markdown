---
layout: post
title: "Java实现Autowired自动注入"
data: 2021-7-16 21:22
categories: java
---

继续使用上个文章的类容器

创建一个注解

```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Autowired {
}
```

遍历所有字段包括私有的

```java
private void autowired() {
        for (Map.Entry<Class<?>, Object> classObjectEntry : classes.entrySet()) {
            for (Field declaredField : classObjectEntry.getKey().getDeclaredFields()) {
                declaredField.setAccessible(true);
                if (classes.get(declaredField.getType()) != null) {//容器内是否有这个类的对象
                    try {
                        //赋值
                        declaredField.set(classObjectEntry.getValue(), classes.get(declaredField.getType()));
                    } catch (IllegalAccessException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }

```

在加入到容器后就调用autowired

```java
    public ClassContainer() {
        List<Class<?>> scanClasses = new ArrayList<>(List.of(Test1.class, Test2.class));//注意这里Test2也被加入到了容器里
        scanClasses.forEach(it -> {
            try {
                classes.put(it, it.getConstructor().newInstance());
            } catch (InstantiationException | IllegalAccessException | NoSuchMethodException | InvocationTargetException e) {
                e.printStackTrace();
            }
        });
        autowired();
    }
```

```java
    public <T> T create(Class<T> klass, Object instance) {
        classes.put(klass, instance);
        autowired();
        return (T) classes.get(klass);
    }
```

创建Test3

```java
public class Test3 {
    public void render() {
        System.out.println("Test3");
    }
}
```

Test1使用Autowired

```java
public class Test1 {
    @Autowired
    private Test2 test2;

    @Autowired
    private Test3 test3;

    public void render() {
        test2.render();
        test3.render();
    }
}
```

测试一下

```java
public class Main {

    private static final ClassContainer classContainer = new ClassContainer();

    public static void main(String[] args) {
        classContainer.create(Test1.class).render();
    }
}
```

Test2正常 Test3空指针 因为不在容器里