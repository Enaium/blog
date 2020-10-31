---
layout: post
title: "[BlocklyNukkit入门]#7选择窗口"
data: 2020-10-31 23:07
categories: blocklynukkit
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