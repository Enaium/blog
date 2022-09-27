---
title: "解决IDEA Push rejected问题"
date: 2020-1-6 11:25
categories: others
---

![a](/assets/others/2020-1-6-2.png)

右击目录选择`Git Bash Here`

![a](/assets/others/2020-1-6-4.png)

依次输入以下命令


```
git pull
git pull origin master
git pull origin master --allow-unrelated-histories
```

![a](/assets/others/2020-1-6-1.png)

再次推送

![a](/assets/others/2020-1-6-3.png)