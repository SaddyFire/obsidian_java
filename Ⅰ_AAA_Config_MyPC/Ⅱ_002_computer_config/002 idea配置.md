
# 二、idea配置

## 1. 常用short_cut

|     |     |
| --- | --- |
| 快速查看继承关系 | **ctrl + h** |
| 全局查找 | **ctrl + shift + f** |
| 查看父类方法 | **ctrl + o** |
| 此方法或变量的调用之间的跳跃 | **ctrl + B / ctrl + 鼠标左键** |
| 查看本类结构 | **alt + 7** |
| 查找类或方法在哪被使用 | **alt + f7** |
| 自动缩进 | **ctrl + alt + i<br>** |
| 可以跳转到上次编辑的地方 | **ctrl + shift + space<br>** |
| 查看实现类接口 | **ctrl + alt+ B / ctrl + alt + 鼠标左键** |
| 显示文档注释 | ctrl + Q |
| 打开 progect constructure | ctrl + shift + alt +s |
| 搜索类 | ctrl + N |
| 快速选择当前单词 | ctrl + w |
| 进入接口实现类 | ctrl + alt + 左键 |
| 对选中代码进行操作 | ctrl + alt + T |
| 快速切换大小写 | ctrl + shift + U |
| 快速切换(界面外观、代码风格、快捷映射等菜单) | ctrl + \`
| 打开 \ 隐藏 工程目录结构 | alt + 1 |
| 打开 \ 隐藏 控制台 | alt + 4 |
| 格式化代码 | ctrl + alt + L |
| 向下复制一行 | ctrl + d |
| 剪切当前行 | ctrl + x |
| 单行注释 | ctrl + / |
| 提醒  | ctrl + 空格 |
| 上/下移当前行 | alt + shift + ↑↓ |
| 快速活动指针 | ctrl + ←→ |
| 重构名 | shift f6 |
| 动结束代码，行末自动添加分号 | Ctrl + Shift + Enter |
| 展开/收缩所有代码 | ctrl + Shift + +/- |
| 快速封装成方法 | ctrl + alt + m |
| 成员方法使用参数提示 | ctrl + p |
| 关闭当前窗口 | ctrl + f4 |
| 快速跳转到第几行 | ctrl + g |
| 快速接收返回值 | ctrl + alt + v |
| 取消撤销 | ctrl + shift + z |


## 2.基础设置


|                      |                                                                                                                                                                                           |
| -------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 菜单字体大小设置     | Appearance -> Size                                                                                                                                                                        |
| 代码字体大小设置     | Editor -> Font                                                                                                                                                                            |
| 显示方法分割线       | Editor->Appearance->show method separators                                                                                                                                                |
| 模板生成             | Live templates -> “+” -> …                                                                                                                                                                |
| 取消自动更新         | File->Setting->Appearance&Beha->System Setting—Updates<br>取消勾选     Automatically check updates                                                                                        |
| 背景设置             | apprarence&behavior -> appearence -> background image                                                                                                                                     |
| 修改快捷键 keymap    | keymap->Main menu->Code                                                                                                                                                                   |
| **idea标签多行显示** | windows -> Editor Tabs -> Configure Editor Tabs -> Show tabs in<br>把 Show pinned tabs in a separate row 勾去掉                                                                           |
| **提醒忽略大小写**   | ctrl + alt + s -> 搜索**Code Completion** -\> Match case 勾去掉                                                                                                                           |
| **Mave仓库设置**     | ctrl + alt + s -> 搜索**maven**                                                                                                                                                           |
| **Maven全局设置**    |                                                                                                                                                                                           |
| **File Header设置**  | ctrl + alt + s -> 搜索file and code -\> Includes -> /<br>\ @author YHF<br>\ @date ${DATE} <br>**<br> *@TIME:${TIME}/                                                                      |
| **JS语言版本**       | ctrl + alt + s -> 搜索 **javascript** -\> language version 选择 ECMAScript 6+                                                                                                             |
| **热部署**           | 导入依赖 -> <br>ctrl + alt + s -> 搜索 **compiler** -\> 勾选 \*\*Build project auto \*\*-> <br>Shift+Ctrl+Alt+/ -> 选择**Registry** -\> 勾选 **compiler.automake.allow.when.app.running** |
| **git配置**          | ctrl + alt + s -> 搜索 git -> 配置路径                                                                                                                                                    |
| 代码左侧折叠线设置   | ctrl + alt + s -> Editor -> Color Scheme -> General -> Editor -> Tear line                                                                                                                |
| 代码左侧行号颜色设置 | ctrl + alt + s -> Editor -> Color Scheme -> General -> code -> Line number                                                                                                                |
| 方法分割线颜色设置   | ctrl + alt + s -> Editor -> Color Scheme -> General -> code -> method separator color                                                                                                     |
| 修改terminal         | ctrl + alt + s -> Terminal -> 改Shell path                                                                                                                                                |


- 热部署依赖


```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-devtools</artifactId>
	<scope>runtime</scope>
</dependency>
```


## 3. 常用插件下载


|                        |                                    |
| ---------------------- | ---------------------------------- |
| 翻译                   | translation                        |
| 彩虹括号               | rainbow brackets                   |
| 等待条                 | nyan progress bar                  |
| 重置注册               | ide eval reset                     |
| 高亮控制台             | Grep Console                       |
| mybatis小鸟插件        | MyBatisX                           |
| maven必备插件          | Maven Helper                       |
| 快速转换为web工程      | JBLJavaToWeb                       |
| 高亮提示               | HighlightBracketPair               |
| 驼峰命名法             | CamelCase   **Shift+alt + u** 调用 |
| 一键调用所有getset方法 | GenerateAllSetter                  |
| 代码生成器             | codehelper.generator               |
| 阿里巴巴代码规范       | Alibaba Java Coding Guidelines     |
| 显示代码缩略图         | CodeGlance2                        |
| Codota代码智能提示     | Codota AI Autocomplete for ...     |
| 快捷键提示             | Key Promeoter X                    |
| 代码质量检查           | SonarLint                          |


## 我的 short_cut


|     |     |
| --- | --- |
| 修改快捷键 keymap | keymap->Main menu->Code |
| 快速生成 generate | shift + ` |
| 快速运行 run | Edit Configurations:-> alr + r |
| 快速新建文件 new->class | alt + w |
| 定位到上/下一个错误err | alt + e/d |