---
title: "自写一个EventBus"
data: 2021-2-23 22:09
categories: java
---

EventBus，什么是EventBus。

EventBus是事件发布-订阅总线，简单来说监听一个事件，一个方法订阅这个事件，如果事件调用，那么订阅了这个事件的方法也会跟着调用，这就是EventBus。

创建一个注解，用于订阅事件，名字可以随便起，当然也可以叫Subscribe，我这里叫Event。

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Event {
}
```

创建Listener监听器。

```java
public class Listener {
}
```

创建MethodBean类，来储存订阅方法，Object是订阅类的对象，Method就是被订阅的方法。

```java
public class MethodBean {
    private final Object object;
    private final Method method;

    public MethodBean(Object object, Method method) {
        this.object = object;
        this.method = method;
    }

    public Object getObject() {
        return object;
    }

    public Method getMethod() {
        return method;
    }
}
```

创建一个EventManager，来管理订阅的事件。

```java
public class EventManager {
}
```

创建一个HashMap合集K是监听器，V是被调用的方法，因为一个监听器可能有多个方法，并且要保证线程安全，需要使用CopyOnWriteArrayList。

```java
public class EventManager {
    private final HashMap<Class<? extends Listener>, CopyOnWriteArrayList<MethodBean>> events = new HashMap<>();
}
```

创建register和unregister方法来注册和取消注册订阅的对象。

```java
public class EventManager {
    public void register(Object o) {

    }

    public void unregister(Object o) {
        
    }
}
```

注册。

```java
public void register(Object o) {
     Class<?> type = o.getClass();//获取类。

    for (Method method : type.getDeclaredMethods()) {//遍历出所有方法。
        if (method.getParameterTypes().length == 1 && method.isAnnotationPresent(Event.class)) {//保证方法只有一个参数，并且有Event这个注解。
            method.setAccessible(true);
            @SuppressWarnings("unchecked")
            Class<? extends Listener> listener = (Class<? extends Listener>) method.getParameterTypes()[0];

            MethodBean methodBean = new MethodBean(o, method);

            //把这些都put到events里面。
            if (events.containsKey(listener)) {
                if (!events.get(listener).contains(methodBean)) {
                    events.get(listener).add(methodBean);
                }
            } else {
                events.put(listener, new CopyOnWriteArrayList<>(Collections.singletonList(methodBean)));
            }
        }
    }
}
```

取消注册很简单，只要将events的K和V移除就行。

```java
public void unregister(Object o) {
    events.values().forEach(methodBeans -> methodBeans.removeIf(methodMethodBean -> methodMethodBean.getObject().equals(o)));
    events.entrySet().removeIf(event -> event.getValue().isEmpty());
}
```

创建一个getEvent方法来获取一个监听器的所有订阅。

```java
public CopyOnWriteArrayList<MethodBean> getEvent(Class<? extends Listener> type) {
    return events.get(type);
}
```

创建一个单例。

```java
public enum Main {

    INSTANCE;

    EventManager eventManager = new EventManager();
    
}
```

回到刚才创建的Listener类。

创建一个call方法来进行事件触发操作，当事件触发，获取监听器的所有订阅方法来调用，参数就是当前的监听器。

```java
public class Listener {
    public void call() {
        CopyOnWriteArrayList<MethodBean> methodBeans = Main.INSTANCE.eventManager.getEvent(this.getClass());

        if(methodBeans == null) {
            return;
        }

        methodBeans.forEach(event -> {
            try {
                event.getMethod().invoke(event.getObject(), this);
            } catch (IllegalAccessException | InvocationTargetException e) {
                e.printStackTrace();
            }
        });
    }
}
```

创建一个监听器。

```java
public class UpdateEvent extends Listener {
}
```

一个简单的EventBus已经写好了，现在来测试一下。

```java
public enum Main {

    INSTANCE;

    EventManager eventManager = new EventManager();

    public static void main(String[] args) {
        Main.INSTANCE.eventManager.register(new Test());//register
        new UpdateEvent().call();
    }

    static class Test {
        @Event
        public void on(UpdateEvent e) {
            System.out.println("Event trigger");
        }
    }
}
```

[源码](https://github.com/Enaium-Learn/EventBus)