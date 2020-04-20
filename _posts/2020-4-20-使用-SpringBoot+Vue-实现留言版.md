---
layout: post
title: "使用 SpringBoot+Vue 实现留言版"
date: 2020-4-20 17:29
categories: springboot
---

# [完成源码](https://github.com/Enaium/SpringBoot-Vue-MessageBoard)

## 一.新建Vue项目和SpringBoot项目

### 新建Vue项目
> 1. 新建文件夹`SpringBoot-Vue-MessageBoard`
> 2. 创建Vue项目使用`vue ui`命令(需要vue 3.0
>>选择刚才的目录![2020-4-20-1](/assets/springboot/2020-4-20-1.png)
>>名字为`Vue`创建后V还是小写 创建后可以改为大写![2020-4-20-2](/assets/springboot/2020-4-20-2.png)
>>取消git初始化![2020-4-20-3](/assets/springboot/2020-4-20-3.png)
>>手动配置![2020-4-20-4](/assets/springboot/2020-4-20-4.png)
>>取消![2020-4-20-5](/assets/springboot/2020-4-20-5.png)
>>打开![2020-4-20-6](/assets/springboot/2020-4-20-6.png)
>>创建项目，不保存预设![2020-4-20-7](/assets/springboot/2020-4-20-7.png)

### 新建SpringBoot项目

> 1. 用IDEA打开`SpringBoot-Vue-MessageBoard`这个目录
>>![2020-4-20-8](/assets/springboot/2020-4-20-8.png)
>>![2020-4-20-9](/assets/springboot/2020-4-20-9.png)
> 2. 创建SpringBoot项目
>>右键![2020-4-20-10](/assets/springboot/2020-4-20-10.png)
>>选择`Spring Initializr`![2020-4-20-11](/assets/springboot/2020-4-20-11.png)
>>![2020-4-20-12](/assets/springboot/2020-4-20-12.png)
>>选择这四个![2020-4-20-13](/assets/springboot/2020-4-20-13.png)
>>名字改为`SpringBoot`![2020-4-20-14](/assets/springboot/2020-4-20-14.png)

## 二. 后端

### 配置`application.properties`

```properties
#Mysql
spring.datasource.url=jdbc:mysql://localhost:3306/enaium?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai
spring.datasource.username=root
spring.datasource.password=root
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.jpa.show-sql= true
spring.jpa.properties.hibernate.format_sql = true
#Server
server.port=8181
```

### 写实体类

```java
package cn.enaium.message.entity;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

import javax.persistence.Entity;
import javax.persistence.Id;

/**
 * Project: message
 * -----------------------------------------------------------
 * Copyright © 2020 | Enaium | All rights reserved.
 */
@Data
@Entity
@NoArgsConstructor
@AllArgsConstructor
public class Message {
    @Id
    private Long id;
    private String author;
    private String message;
    private String time;
}
```

### 实体类Jpa

```java
package cn.enaium.message.repository;

import cn.enaium.message.entity.Message;
import org.springframework.data.jpa.repository.JpaRepository;

/**
 * Project: message
 * -----------------------------------------------------------
 * Copyright © 2020 | Enaium | All rights reserved.
 */
public interface MessageRepository extends JpaRepository<Message, Integer> {
}
```

### Controller

```java
package cn.enaium.message.controller;

import cn.enaium.message.entity.Message;
import cn.enaium.message.repository.MessageRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;

/**
 * Project: message
 * -----------------------------------------------------------
 * Copyright © 2020 | Enaium | All rights reserved.
 */
@RestController
public class Controller {

    @Autowired
    private MessageRepository messageRepository;

    @RequestMapping("/getMessages")
    private List<Message> getMessages() {
        return messageRepository.findAll();//遍历所有留言
    }

    @GetMapping("/postMessage")
    private String postMessage(@RequestParam String author, @RequestParam String message) {
        if(author.replaceAll(" ","").equals("") || message.replaceAll(" ","").equals("")) {
            return "filed";
        }//判断名字和留言是否为空
        messageRepository.save(new Message((long) (messageRepository.findAll().size() + 1),author,message,new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date())));//保存留言到数据库
        return "success";
    }
}
```

### 解决跨源请求问题

![2020-4-20-15](/assets/springboot/2020-4-20-15.png)

```java
package cn.enaium.message.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

/**
 * Project: message
 * -----------------------------------------------------------
 * Copyright © 2020 | Enaium | All rights reserved.
 */
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    @Override//重写这个方法
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedOrigins("*")
                .allowedMethods("GET", "HEAD", "POST", "PUT", "DELETE", "OPTIONS")
                .allowCredentials(true)
                .maxAge(3600)
                .allowedHeaders("*");
    }
}
```

## 三. 前端

### 安装插件`axios`和`Element UI`

![2020-4-20-16](/assets/springboot/2020-4-20-16.png)

![2020-4-20-17](/assets/springboot/2020-4-20-17.png)


### 写`Home`页面

```html
<template>
    <div>
        <h1>留言版</h1>
        <el-input
                type="text"
                placeholder="请输入你的名字"
                v-model="messageBoard.author"
                maxlength="16"
                show-word-limit
        >
        </el-input>
        <div style="margin: 20px 0;"></div>
        <el-input
                type="textarea"
                placeholder="请输入留言"
                v-model="messageBoard.message"
                show-word-limit
        >
        </el-input>
        <div style="margin: 20px 0;"></div>
        <el-button type="primary" plain @click="postMessage">留言</el-button>
        <el-divider></el-divider>
        <el-table
                :data="messages"
                height="250"
                border
                style="width: 100%">
            <el-table-column
                    prop="id"
                    label="序号"
                    width="100">
            </el-table-column>
            <el-table-column
                    prop="author"
                    label="名字">
            </el-table-column>
            <el-table-column
                    prop="message"
                    label="留言">
            </el-table-column>
            <el-table-column
                    prop="time"
                    label="时间"
                    width="160">
            </el-table-column>
        </el-table>
        <el-link type="primary" href="https://github.com/Enaium">By Enaium</el-link>
    </div>
</template>

<script>
    export default {
        name: "Home",
        data() {
            return {
                messageBoard: {
                    author: '',
                    message: ''
                },
                messages: []
            }
        },
        methods: {
            postMessage() {
                if (this.messageBoard.author === '') {
                    this.$message.error('请输入你的名字');
                    return
                }

                if (this.messageBoard.message === '') {
                    this.$message.error('请输入留言');
                    return
                }

                axios.get("http://localhost:8181/postMessage?author=" + this.messageBoard.author + "&message=" + this.messageBoard.message).then((t) => {
                    if (t.data === 'success') {
                        this.$message({
                            message: '留言成功',
                            type: 'success'
                        });
                    } else {
                        this.$message.error('留言失败');
                    }
                })
            }
        },
        created() {
            axios.get("http://localhost:8181/getMessages").then((t) => {
                this.messages = t.data
            })
        }
    }
</script>
```

### 路由页面

```js
import Vue from 'vue'
import VueRouter from 'vue-router'
import Home from '../views/Home.vue'

Vue.use(VueRouter)

const routes = [
    {
        path: '/',
        name: 'Home',
        component: Home
    }
]

const router = new VueRouter({
    mode: 'history',
    routes
})

export default router
```

## 四. 运行

> 1. 运行SpringBoot
> 2. cd 到`Vue`使用`npm run serve`运行

![2020-4-20-18](/assets/springboot/2020-4-20-18.png)

