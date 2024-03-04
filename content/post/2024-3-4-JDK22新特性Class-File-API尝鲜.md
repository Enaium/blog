---
layout: post
title: "JDK22新特性Class-File API尝鲜"
date: 2024-03-04T12:14:18+08:00
---

## 前言

到目前为止`JDK22`已经`Final Release Candidate`了，不出意外的话，这个就是最终`General Availability`版本了，在本次更新有一个新的的特性也就是，`Class-File API`，不过还是在预览版中，不过我们可以尝鲜一下，也就是在未来的版本中可能会被删除或者修改，大家在之前可能使用过`ASM`等第三方库，但现在`JDK`是每6个月就会发布一个新的版本，第三方库可能会更新不及时，所以`JDK`内置了一个`Class-File API`，这样就可以更好的支持`Java`的新特性。

## 安装

我们先需要在[jdk.java.net](https://jdk.java.net/22/)JDK22，之后再`IntelliJ IDEA`中开启`22(Preview)`，之后就可以使用`Class-File API`了。

## 使用

### 读取类信息

我们首先是读取一个`class`文件，也就是读取它的类信息，既然是读取类，我们就写一个类之后再编译。

```java
public class Test {

    public String name = "Enaium";

    public void render() {
        System.out.println(name);
    }
}
```

之后我们在`IntelliJ IDEA`中编译一下，然后我们就可以读取这个`class`文件了。

```java
void main() throws IOException {
    ClassFile.of().parse(Path.of("out/production/untitled1/Test.class"));
    ClassFile.of().parse(Files.readAllBytes(Path.of("out/production/untitled1/Test.class")));
}
```

我们可以看到`ClassFile`有一个`of`方法，这个方法返回一个`ClassFile`对象，然后我们可以调用`parse`方法解析`class`文件，这里可以使用两种方法，一种是传入`Path`对象，一种是传入`byte`数组。

```java
void main() throws IOException {
    final ClassModel parse = ClassFile.of().parse(Path.of("out/production/untitled1/Test.class"));
    System.out.println(parse.majorVersion());
    System.out.println(parse.superclass().get().name());
    for (PoolEntry poolEntry : parse.constantPool()) {
        System.out.println(STR."  \{poolEntry.toString()}");
    }
}
```

我们可以看到`ClassFile`有一个`parse`方法，这个方法返回一个`ClassModel`对象，然后我们可以调用`majorVersion`方法获取`class`文件的版本，`superclass`方法获取父类，`constantPool`方法获取常量池。

其中`PoolEntry`比较特殊，它是一个接口，所以我直接调用`toString`方法，这个方法返回一个`String`对象，这个对象就是常量池的内容，我们进入到`JDK`源码中，获取它有哪些实现类，`ClassEntry`、`FieldRefEntry`、`MethodRefEntry`等等。

### 读取字段信息

```java
void main() throws IOException {
    final ClassModel parse = ClassFile.of().parse(Path.of("out/production/untitled1/Test.class"));
    for (FieldModel field : parse.fields()) {
        System.out.println(STR."\{field.flags().flags()} \{field.fieldName()}: \{field.fieldType()} | \{field.fieldTypeSymbol().packageName()}.\{field.fieldTypeSymbol().displayName()}");
    }
}
```

我们调用`fields`可以获取这个类中的所有字段，然后我们可以调用`flags`方法获取字段的修饰符，`fieldName`方法获取字段的名字，`fieldType`方法获取字段的类型，`fieldTypeSymbol`方法获取字段的类型的符号。

### 读取方法信息

```java
void main() throws IOException {
    final ClassModel parse = ClassFile.of().parse(Path.of("out/production/untitled1/Test.class"));
    for (MethodModel method : parse.methods()) {
        System.out.println(STR."\{method.flags().flags()} \{method.methodName()}\{method.methodType()}");
        for (CodeElement codeElement : method.code().get()) {
            System.out.println(STR."  \{codeElement}");
        }
    }
}
```

和读取字段不同的是，可以使用`code`方法获取方法的指令，类似于`ASM`的中的`Instruction`。

### 创建类信息

```java
void main() throws IOException {
    final String name = "Enaium";
    final byte[] build = ClassFile.of().build(ClassDesc.of(name), classBuilder -> {

    });
    Files.write(Path.of(STR."\{name}.class"), build);
}
```

这里使用`ClassFile`中的`build`方法构建一个`class`文件，这个方法传入一个类信息，一个`ClassBuilder`的消费者，我们这里只是创建一个空的`class`文件，之后我们将返回的`byte`数组写入到文件中，之后我们使用`IntelliJ IDEA`打开这个`class`文件，我们可以看到这个`class`文件是一个空的`class`文件。

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by FernFlower decompiler)
//

public class Enaium {
}
```

### 添加字段信息

```java
classBuilder.withField("name", ClassDesc.ofDescriptor("Ljava/lang/String;"), ClassFile.ACC_PUBLIC);
```

这里使用`withField`方法添加一个字段，这个方法传入字段的名字，字段的类型，字段的修饰符。

### 添加方法信息

```java
classBuilder.withMethod("<init>", MethodTypeDesc.ofDescriptor("()V"), ClassFile.ACC_PUBLIC, methodBuilder -> {
});
```

这里使用`withMethod`方法添加一个方法，这个方法传入方法的名字，方法的类型，方法的修饰符，一个`MethodBuilder`的消费者，我们这里只是创建一个空的方法。

### 添加代码信息

这里我们为刚才创建好的字段添加一个值。

```java
methodBuilder.withCode(codeBuilder -> {
    codeBuilder.aload(codeBuilder.receiverSlot());
    codeBuilder.invokespecial(ClassDesc.ofDescriptor("Ljava/lang/Object;"), "<init>", MethodTypeDesc.ofDescriptor("()V"));
    codeBuilder.aload(codeBuilder.receiverSlot());
    codeBuilder.ldc("Enaium");
    codeBuilder.putfield(ClassDesc.of(name), "name", ClassDesc.ofDescriptor("Ljava/lang/String;"));
    codeBuilder.return_();
});
```

这里使用`withCode`方法添加代码，这个方法传入一个`CodeBuilder`的消费者，这里我们使用`aload`方法加载`this`，`invokespecial`方法调用父类的构造方法，`putfield`方法设置字段的值，`return_`方法返回。

现在我们可以创建一个方法用来获取刚才创建好的字段。

```java
methodBuilder.withCode(codeBuilder -> {
    codeBuilder.aload(codeBuilder.receiverSlot());
    codeBuilder.getfield(ClassDesc.of(name), "name", ClassDesc.ofDescriptor("Ljava/lang/String;"));
    codeBuilder.areturn();
});
```

这里使用`withCode`方法添加代码，这个方法传入一个`CodeBuilder`的消费者，这里我们使用`aload`方法加载`this`，`getfield`方法获取字段，`areturn`方法返回，这里返回的是一个对象，所以和刚才的`return_`不一样。

之后我也可以添加一个方法来设置刚才创建好的字段。

```java
classBuilder.withMethod("setName", MethodTypeDesc.ofDescriptor("(Ljava/lang/String;)V"), ClassFile.ACC_PUBLIC, methodBuilder -> {
    methodBuilder.withCode(codeBuilder -> {
        codeBuilder.aload(codeBuilder.receiverSlot());
        codeBuilder.aload(codeBuilder.parameterSlot(0));
        codeBuilder.putfield(ClassDesc.of(name), "name", ClassDesc.ofDescriptor("Ljava/lang/String;"));
        codeBuilder.return_();
    });
});
```

### 测试

```java
final Object o = URLClassLoader.newInstance(Collections.singleton(Path.of(".").toUri().toURL()).toArray(URL[]::new)).loadClass(name).getConstructor().newInstance();
final Method getName = o.getClass().getMethod("getName");
System.out.println(getName.invoke(o));
final Method setName = o.getClass().getMethod("setName", String.class);
setName.invoke(o, "This is enaium's class file");
System.out.println(getName.invoke(o));
```
这里我们使用`URLClassLoader`加载刚才创建好的`class`文件，之后我吗就可以使用反射调用里面的方法了。

## 总结

本篇文章简单的使用了`Class-File API`，之后我会继续深入的了解这个新特性，也会写一些关于`Class-File API`的文章。