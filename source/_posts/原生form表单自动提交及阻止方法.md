---
title: 原生form表单自动提交及阻止方法
date: 2018-04-27 13:55:46
description: 原生form表单自动提交存在一些规则，如果不知道这些规则，form表单的自动提交将会变得很诡异。下面我们来分析一下这些规则以及阻止form表单提交的方法。
categories: 前端
tags: 
- form表单
---
## form表单自动提交规则

1. form表单中只有一个type=text的input（可以有其他类型的表单），在input中按enter键，会自动提交

2. form表单中有多个type=text的input，且无type=submit的按钮元素，则在input中按enter键，不会自动提交

3. form表单中有type=submit的按钮元素，点击按钮元素或者在input中按enter键，会自动提交

3. form表单中有type=button的按钮元素且有多个input元素，点击按钮元素或者在input中按enter键，不会自动提交

## 阻止form表单自动提交方法

### 在Html中

```
    <form onsubmit="return false">
        <input type = "text">
        <input type = "submit">submit
    </form>
```
### 在Js中
我们需要给表单添加submit事件，并在事件中组织submit事件的默认事件。
```
    var form = document.getElementsByTagName('form')[0];
    form.addEventListener('submit',function(e){
        e.preventDefault();
    });
```