---
title: Intellij Idea Plugin 开发
date: 2019-03-23 20:53:00
tags: DevPlugin
---

做CPE的时候，常常需要给客户提供hotfix，每次都需要手动把改动的java文件编译后，再把所有的`.class`文件以及包含`.class`目录打包成zip包，后再附上hotfix的reademe文件。于是查了[Intellij Plugin Dev Document](http://www.jetbrains.org/intellij/sdk/docs/welcome.html)，动手设计了下面的原型，并实现了基本能用的`HotfixWorkshop`插件。

![](/images/plugin/hotfixworkshop.png)

## IntelliJ Platform 简介

Plugin的开发只是`IntelliJ Platform`功能的一部分，你其实可以用这个`Platform`去开发自己的IDE，例如`Android Studio`就是基于`IntelliJ平台`开发的。
简单来说，`Platform`提供了IDE开发的SDK。用官方的话来说，它提供了以下设施：

- UI组件: 提供基于`JVM`的跨平台的UI组件 (tool windows, tree views, lists, popup menu, dialog)
- 编辑器: 例如包含了image editor，以及text editor，其中 text editor 实现了语法高亮，代码提示等等功能
- Debug: 提供了表达式计算，调用栈分析等等

## Plugin 开发

`IntelliJ Platform`是基于`JVM`的应用，因此Plugin可以使用`Java`或者`Kotlin`来开发，这意味着不能使用`非JVM`的语言开发。
我们来看看使用SDK能开发哪些插件。

- 自定义编程语言的辅助支持: 例如，文件类型的识别，词法分析，语法高亮，格式化，代码提示，快速修复等。
- 与框架集成：例如，与Spring Framework的集成插件。
- 与工具集成：例如，与[Gerrit](https://plugins.jetbrains.com/plugin/7272-gerrit)的集成插件
- UI类插件：这类插件常常用于改变标准的IDE UI，一个很好的例子是[BackgroundImage](https://plugins.jetbrains.com/plugin/72-backgroundimage)

关于插件开发入门，可以直接看看官方的[Getting Started](http://www.jetbrains.org/intellij/sdk/docs/basics/getting_started.html)

官方对SDK描述的文档并不多，所以很多功能大多数只能靠开源的例子和自己摸索去开发。如果你要实现一个复杂一些的UI，需要对Java的`swing`有所熟悉。

**添加一个窗体**

```xml
<toolWindow id="Console"
            anchor="right"
            icon="/images/icon.png"
            factoryClass="com.intellij.execution.dashboard.RunDashboardToolWindowFactory" />
```

**为编辑器添加contextmenu**

```xml
<action id="AddFileAction" class="com.microfocus.hw.actions.AddFileAction" text="Add as HF">
    <add-to-group group-id="EditorPopupMenu" anchor="first"/>
</action>
```

**为项目添加contexmenu**

```xml
<action class="com.microfocus.hw.actions.PackHFAction" id="PackHFAction" text="Pack HF">
      <add-to-group group-id="ProjectViewPopupMenu" anchor="first"/>
</action>
```

**获取当前项目**

```java
public Project getCurrentProject() {
    DataContext dataContext = DataManager.getInstance().getDataContextFromFocus().getResult();
    Project project = DataKeys.PROJECT.getData(dataContext);
    return project;
}
```

**自定义支持checkbox的列表**

创建`List Panel`

```java
// list panel
JBScrollPane scrollPane = new JBScrollPane();
scrollPane.setAlignmentX(Component.LEFT_ALIGNMENT);
scrollPane.setBounds(83, 132, 369, 213);
this.changeFileList = new JBList<>();
this.changeFileList.setListData(ApplicationManager.getIns().getAllHotfixNames());
this.changeFileList.setCellRenderer(new FileCellRenderer());
this.changeFileList.setSelectionModel(new DefaultListSelectionModel() {
    @Override
    public void setSelectionInterval(int index0, int index1) {
        if (super.isSelectedIndex(index0)) {
            super.removeIndexInterval(index0, index1);
        } else {
            super.addSelectionInterval(index0, index1);
        }
    }
});
scrollPane.setViewportView(this.changeFileList);
scrollPane.setBorder(null);
```

自定义渲染列表项

```java
/**
 * Created by atom on 2019/2/9.
 */
public class FileCellRenderer extends JCheckBox implements ListCellRenderer {
    @Override
    public Component getListCellRendererComponent(JList list, Object value, int index, boolean isSelected, boolean cellHasFocus) {
        this.setText(value.toString());
        setBackground(isSelected ? list.getSelectionBackground() : list.getBackground());
        setForeground(isSelected ? list.getSelectionForeground() : list.getForeground());
        this.setSelected(isSelected);
        return this;
    }
}
```

EOF