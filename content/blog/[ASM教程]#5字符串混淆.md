---
title: "[ASM教程]#5字符串混淆"
date: 2021-03-19T20:05:00+0800
categories: asm
---

测试类，需要混淆这里所有的字符串，上一期我们学习了如果插入一个常量，那么拦截插入常量的时候判断是不是字符串，如果是就混淆它。

```java
public class Learn5Test {
    private final String name = "Enaium";

    private Learn5Test() {
        render(name);
    }

    private void render() {
        System.out.println("obfuscatory by " + name);
    }

    private void render(String text) {
        System.out.println(text);
    }
}
```

重写方法访问。

```java
@Override
public MethodVisitor visitMethod(int access, String name, String desc, String signature, String[] exceptions) {
    return new MethodVisitor(api, super.visitMethod(access, name, desc, signature, exceptions)) {

    };
}
```

重写常量访问。

```java
@Override
public void visitLdcInsn(Object cst) {
                
}
```

```java
@Override
public MethodVisitor visitMethod(int access, String name, String desc, String signature, String[] exceptions) {
    return new MethodVisitor(api, super.visitMethod(access, name, desc, signature, exceptions)) {
        @Override
        public void visitLdcInsn(Object cst) {
            if (cst instanceof String) {//判断是不是字符串类型，如果不是不重写。
                byte[] bytes = ((String) cst).getBytes();//先将字符串转换为字节组。
                mv.visitTypeInsn(NEW, "java/lang/String");//创建字符串对象。
                mv.visitInsn(DUP);
                mv.visitIntInsn(SIPUSH, bytes.length);//将字符串转换的字节组插入。
                mv.visitIntInsn(NEWARRAY, T_BYTE);
                for (int i = 0; i < bytes.length; i++) {
                    mv.visitInsn(DUP);
                    mv.visitIntInsn(SIPUSH, i);
                    mv.visitIntInsn(SIPUSH, bytes[i]);
                    mv.visitInsn(AASTORE);
                }
                mv.visitLdcInsn("UTF-8");
                mv.visitMethodInsn(INVOKESPECIAL, "java/lang/String", "<init>", "([BLjava/lang/String;)V", false);//调用String的构造方法，把字节组转换为字符串，使用UTF-8保证中文不乱码，因为字符串有一个解密的过程，所以反字符串混淆也很简单。
            } else {
                super.visitLdcInsn(cst);
            }
        }
    };
}
```
