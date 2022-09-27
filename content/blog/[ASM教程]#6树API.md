---
title: "[ASM教程]#6树API"
date: 2021-04-27T15:02:00+0800
categories: asm
---

使用树API来生成一个类

```java
public static void main(String[] args) {
    ClassWriter classWriter = new ClassWriter(ClassWriter.COMPUTE_MAXS);
    ClassNode classNode = new ClassNode();
    classNode.visit(V1_8, ACC_PUBLIC, "cn/enaium/learn/asm/learn6/Learn6Test", null, "java/lang/Object", null);
    MethodNode methodNode = new MethodNode(ACC_PUBLIC + ACC_STATIC, "render", "()V", null, null);//一个方法
    methodNode.instructions.add(new FieldInsnNode(GETSTATIC, "java/lang/System", "out", "Ljava/io/PrintStream;"));
    methodNode.instructions.add(new LdcInsnNode("Hello ASM!"));
    methodNode.instructions.add(new MethodInsnNode(INVOKEVIRTUAL, "java/io/PrintStream", "println", "(Ljava/lang/String;)V", false));
    methodNode.visitMaxs(2, 1);
    methodNode.instructions.add(new InsnNode(RETURN));
    classNode.methods.add(methodNode);//添加方法
    classNode.accept(classWriter);
    String name = Learn2.class.getResource("/cn/enaium/learn/asm/learn6/").getPath() + "Learn6Test.class";

    try {
        FileOutputStream out = new FileOutputStream(name);
        out.write(classWriter.toByteArray());
        out.close();
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

字段或方法都在`fields`和`methods`里是一个列表，并且操作也是一个列表（双向链表），所以可以很容易的操作一个类。

接着再分析类。

```java
try {
    ClassReader classReader = new ClassReader(new FileInputStream(name));
    ClassNode readClassNode = new ClassNode();
    classReader.accept(readClassNode,0);
    System.out.println(readClassNode.name);//类名
    for (MethodNode method : readClassNode.methods) {
        System.out.println(method.name);//方法名
        ListIterator<AbstractInsnNode> iterator = method.instructions.iterator();
        while (iterator.hasNext()) {
            AbstractInsnNode next = iterator.next();
            System.out.println(next.getClass());//操作
        }
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

也可以把方法的操作给遍历出来。
