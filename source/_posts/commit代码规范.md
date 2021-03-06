---
title: commit代码规范
date: 2018-12-17 13:32:39
description: commit代码规范
tags: [Git]
categories: Git
---

基本样式：

      [type] [module] : --bug=1000627 Message
      //空一行
      detail
      //空一行

---

type, module 和 message 是必选。detail 为可选。

- type 用于说明 commit 的类别，使用下面标识：
    1. bug-fix：修复bug
    2. feature：新功能实现
    3. optimize：重构，优化，工程配置等
    4. merge:  用于替换自动生成合并分支log

- module 用于说明commit内容所属的业务模块（如果不属于任何模块，使用“通用”）
- detail 用于在该commit内容不单一的时候，加以说明
- 针对bug-fix 和 feature需要添加关键字： --bug=bugid --story=storyid

---

### 样例

===
[fixbug] [咨询]  : 修复会话页面在iOS10上显示异常的问题

===
[fixbug] [咨询] [tapd id: 1009080,1090900] : 修复以下bug：

1.修复会话页面在iOS10上显示异常的问题
2.修复tabbarItem小红点显示逻辑不正确bug
3.修复navigationbar高度不正确的问题

===
[feature] [登陆]  : 导入微信SDK，实现微信登陆

===
[optimize] [工程配置] : 增加会话创建失败后的重连，降低失败概率

===
[merge] :  Merge branch 'releaseV0.2.0' into ‘develop’
