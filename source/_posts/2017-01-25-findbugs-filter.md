---
layout:     post
title:      "FindBugs中的过滤"
subtitle:   "过滤不需要的检查"
date:       2017-01-25 00:00:00
author:     "Joe"
header-img: "img/post-bg-2015.jpg"
tags:
    - Java
    - FindBugs
    - Maven
---

在开发项目中我们使用了findbugs进行代码检查，使用findbugs-maven-plugin进行maven配置，在一个项目中发现Findbugs检查有以下错误：

```
Type MS_SHOULD_BE_FINAL

This static field public but not final, and could be changed by malicious code or by accident from another package. The field could be made final to avoid this vulnerability.
```

<!-- more -->

这个类型的错误属于设计上不够严谨，但由于历史代码修改比较多，所以我们可以对这个错误告警进行过滤。
具体方法如下：


### 增加一个过滤文件
我这里命名为：findbugs-exclude.xml

```xml
<FindBugsFilter>
    <Match>
        <Bug pattern="MS_SHOULD_BE_FINAL" />
    </Match>
</FindBugsFilter>
```

### 修改pom.xml
在findbugs-maven-plugin中增加配置：

```xml
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>findbugs-maven-plugin</artifactId>
    <version>3.0.4</version>
    <configuration>
         <excludeFilterFile>findbugs-exclude.xml</excludeFilterFile>
    </configuration>
</plugin>
```

现在这个问题就不会再告警了。

不过，我们还是要严谨的对待FindBugs提醒我们的问题。