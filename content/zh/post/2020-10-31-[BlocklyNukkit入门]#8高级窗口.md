---
layout: post
title: "[BlocklyNukkit入门]#8高级窗口"
date: 2020-10-31T23:10:00+08:00
categroy: blocklynukkit
---

```javascript
manager.newCommand("test", "test", function (sender, args) {
    var test = window.getCustomWindowBuilder("Title");//创建一个高级窗口
    test.buildLabel("Label");//创建一个标签,参数1标题
    test.buildInput("Input", "Input");//创建一个输入框,参数1标题,参数2提示
    test.buildToggle("Toggle");//创建一个开关,参数1标题
    test.buildDropdown("Dropdown", "A;B;C;D;E;F;G");//创建一个下拉框,参数1标题,参数2元素列表,用分号隔开
    test.buildSlider("Slider", 0.0, 100.0, 1.0);//创建一个滑块条,参数1标题,参数2最小的值,参数3最大的值,参数4刻度
    test.buildStepSlider("StepSlider", "A;B;C;D");//创建一个 步骤滑块条,参数1标题,参数2元素列表,用分号隔开

    test.showToPlayer(sender, "Call");
})

function Call(e) {
    logger.info(window./* 获取元素,参数2元素位置,从0开始,参数3元素类型 */getEventCustomVar(e, 1, "input"));
    logger.info(window.getEventCustomVar(e, 2, "dropdown"));
    logger.info(window.getEventCustomVar(e, 3, "slider"));
    logger.info(window.getEventCustomVar(e, 4, "stepSlider"));
    logger.info(window.getEventCustomVar(e, 5, "toggle"));
}
```
