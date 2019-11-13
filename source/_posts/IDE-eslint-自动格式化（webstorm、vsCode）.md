---
layout: _post
title: IDE-eslint-自动格式化（webstorm、vsCode）
date: 2019-10-30 16:19:19
description: IDE-eslint-自动格式化（webstorm、vsCode）
tags: [IDE]
categories: IDE
---
# 1. VScode配置

1. 安装`eslint`和`vetur`插件

   ![](https://junblog.oss-cn-beijing.aliyuncs.com/vscode-eslint%E5%AE%89%E8%A3%85.png)

2. `command + ，`打开配置项，点击编辑器右上角的`{}`符号

   ![](https://junblog.oss-cn-beijing.aliyuncs.com/vscode-%E9%85%8D%E7%BD%AE%E9%A1%B9.png)

3. 追加下列代码

   ![](https://junblog.oss-cn-beijing.aliyuncs.com/vscode-%E9%85%8D%E7%BD%AE.png)

   ```
   {
       "workbench.startupEditor": "newUntitledFile",
       "explorer.confirmDelete": false,
       "window.zoomLevel": 2,
       "eslint.autoFixOnSave": true, //  启用保存时自动修复,默认只支持.js文件
       "editor.formatOnSave": true,
       "eslint.validate": [
           "javascript", //  用eslint的规则检测js文件
           {
               "language": "vue", // 检测vue文件
               "autoFix": true //  为vue文件开启保存自动修复的功能
           }
       ],
       "emmet.syntaxProfiles": {
           "vue-html": "html",
           "vue": "html"
       },
       "vetur.format.defaultFormatter.js": "vscode-typescript",
       "javascript.format.insertSpaceBeforeFunctionParenthesis": true,
       "vetur.format.options.tabSize": 4,
       "vetur.format.options.useTabs": false,
       "vetur.format.defaultFormatterOptions": {
           "prettier": {
               "printWidth": 200, // No line exceeds 100 characters
               "semi": true,
               "singleQuote": true // Prefer double quotes over single quotes
           },
           "prettyhtml": {
               // "printWidth": 100, // No line exceeds 100 characters
               "singleQuote": false // Prefer double quotes over single quotes
           }
       }
   }
   ```

4. 打开package.json文件，新增代码

   ```
   "devDependencies": {
       "@vue/eslint-config-standard": "^4.0.0",	// Vue中JS的严格语法标准
       "babel-eslint": "^10.0.1",	// babel-eslint可以对所有有效的babel代码进行lint处理
       "eslint": "^5.8.0",	// 是在 ECMAScript/JavaScript 代码中识别和报告模式匹配的工具，它的目标是保证代码的一致性和避免错误
       "eslint-plugin-vue": "^5.0.0",	// vue中template的语法标准
   }
   ```

5. 删除node_modules,重新下载：`npm i`
6. ok！



# 2. WebStorem配置

1. 同上述4，5两步，加入依赖，并下载

2. `command + ,`打开配置项
   1. Languages & Frameworks
   2. 选中Javascript
   3. 选中Code Quality Tools
   4. 选中 Eslint
   5. 右侧界面，勾选Enable，启动eslint检测
   6. 选择Eslint Packages包为本地包（此处可以设置为全局，但是担忧全局的包版本和本地包版本不同，带来一系列问题，这个可以后期迭代，确认设置全局是否没问题）

![](https://junblog.oss-cn-beijing.aliyuncs.com/webstorm%E9%85%8D%E7%BD%AEeslint.png)

3. 打开vue，js文件，右键点击`Fix ESlint Problems`，则代码正常格式化

![](https://junblog.oss-cn-beijing.aliyuncs.com/webstorm-eslint-contextmenu.png)

4. 每次需要右键格式化，太麻烦，我们需要自定义一个 webstorm的宏任务，通过`ctrl + s`自动格式化

   **核心原理：录制一段操作，并设定快捷键，之后通过快捷键将此次所有操作执行一遍。**

   1. 打开一个.js/.html/.vue文件
   2. 选中webstorm导航Edit选项
   3. 选中Macros选项
   4. 选中Start Macros Recording（关键步骤，主要是录制）

![](https://junblog.oss-cn-beijing.aliyuncs.com/webstorm-macros.png)

之后所有的操作，都是被WebStorm记录

![](https://junblog.oss-cn-beijing.aliyuncs.com/webstorm-macros-record.png)



**接下里的操作很关键，请注意：**

1. 在内容区域点击右键，找到 `Fix ESlint Problems`，并点击

2. 在`ctrl + s`保存代码
3. 最后点击上图的`停止`按钮，并保存自定义名字（此处我设置名称为**save & format**）

![](https://junblog.oss-cn-beijing.aliyuncs.com/webstorm-macros-save-record.png)

5. `command + ,`打开webstorm配置项
   1. 选中左侧栏目的 `Keymap`选项
   2. 在后侧内容区域，输入 save或者format关键字
   3. 选中刚才保存的名字，右击或者双击，选中 `Add Keyboard Shortcut`

![](https://junblog.oss-cn-beijing.aliyuncs.com/webstorm-macros-search-modify.png)

​	4. 按下`ctrl + s`，提示冲突覆盖的词语（不展示图了），覆盖即可

![](https://junblog.oss-cn-beijing.aliyuncs.com/webstorm-macros-change-keymap-name.png)

​	5. OK，找一个vue/js文件测试即可



# 3. 推荐好用的VS code插件

1. Turbo Console Log：快速打印变量

   ```
   ctrl + alt + L
   ```

2. Gitlens：查看git记录



# 4. 使用技巧

1. option + command + up/down：光标停留在多行
