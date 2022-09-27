---
title: "[ASM教程]#1分析类"
data: 2021-3-2 14:07
categories: asm
---

ASM是一种通用Java字节码操作和分析框架，它可以用于修改现有的class文件或动态生成class文件。

直接进入主题，分析这个类。

```java
public class Learn1Test {
    public boolean aBoolean = false;

    public void render() {
        System.out.println("Hello ASM");
    }
}
```

先创建一个MyClassVisitor类，继承ClassVisitor。

```java
public class MyClassVisitor extends ClassVisitor {
    public MyClassVisitor() {
        super(ASM5);
    }

    @Override
    public void visit(int version, int access, String name, String signature, String superName, String[] interfaces) {
        System.out.println(name + " extends " + superName + " {");
        super.visit(version, access, name, signature, superName, interfaces);
    }

    @Override
    public FieldVisitor visitField(int access, String name, String desc, String signature, Object value) {
        System.out.println(desc + " " + name);
        return super.visitField(access, name, desc, signature, value);
    }

    @Override
    public MethodVisitor visitMethod(int access, String name, String desc, String signature, String[] exceptions) {
        System.out.println(name + " " + desc);
        return super.visitMethod(access, name, desc, signature, exceptions);
    }

    @Override
    public void visitEnd() {
        System.out.println("}");
        super.visitEnd();
    }
}
```

使用ClassReader读取这个类，然后调用accept来分析。

```java
public class Learn1 {
    public static void main(String[] args) throws IOException {
        MyClassVisitor myClassVisitor = new MyClassVisitor();
        ClassReader classReader = new ClassReader(Learn1Test.class.getName());
        classReader.accept(myClassVisitor, 0);
    }
}
```

最后在控制台输出，Z就是boolean类型，<init>是无参数构造方法，render也是一个方法，()V代表一个方法无参数无返回值，由于所有类都继承了Object，所以也输出了。

```
cn/enaium/learn/asm/learn1/Learn1Test extends java/lang/Object {
Z aBoolean
<init> ()V
render ()V
}
```