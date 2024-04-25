---
layout: post
title: "[BlocklyNukkit入门]#7选择窗口"
date: 2020-10-31T23:07:00+08:00
categroy: blocklynukkit
---

```javascript
manager.newCommand("test", "test", function (sender, args) {
    var test = window.getModalWindowBuilder("Test", "Select");//创建选择窗口
    test.setButton1("A");//设置按钮1
    test.setButton2("B");//设置按钮2

    test.showToPlayer(sender, "Call");//显示给玩家
});


function Call(e) {
    e.getPlayer().sendMessage(window.getEventResponseModal(e));//给玩家发送选择的按钮的消息
}
```
