# 自动生成标题编号

打开Typora，然后依次点击：文件→偏好设置→外观→打开主题文件夹
在该文件夹中创建`base.user.css`文件，内容如下，保存后重启Typora即可看到效果

```css
/** initialize css counter */
#write,
.sidebar-content {
    counter-reset: h1;
}

h1,
.outline-h1 {
    counter-reset: h2;
}

h2,
.outline-h2 {
    counter-reset: h3;
}

h3,
.outline-h3 {
    counter-reset: h4;
}

h4,
.outline-h4 {
    counter-reset: h5;
}

h5,
.outline-h5 {
    counter-reset: h6;
}

/** put counter result into headings */
#write h1:before,
.outline-h1 > .outline-item > .outline-label:before {
    counter-increment: h1;
    content: counter(h1) " ";
}

#write h2:before,
.outline-h2 > .outline-item > .outline-label:before {
    counter-increment: h2;
    content: counter(h1) "." counter(h2) " ";
}

#write h3:before,
h3.md-focus.md-heading:before, /** override the default style for focused headings */
.outline-h3 > .outline-item > .outline-label:before {
    counter-increment: h3;
    content: counter(h1) "." counter(h2) "." counter(h3) " ";
}

#write h4:before,
h4.md-focus.md-heading:before,
.outline-h4 > .outline-item > .outline-label:before {
    counter-increment: h4;
    content: counter(h1) "." counter(h2) "." counter(h3) "." counter(h4) " ";
}

#write h5:before,
h5.md-focus.md-heading:before,
.outline-h5 > .outline-item > .outline-label:before {
    counter-increment: h5;
    content: counter(h1) "." counter(h2) "." counter(h3) "." counter(h4) "." counter(h5) " ";
}

#write h6:before,
h6.md-focus.md-heading:before,
.outline-h6 > .outline-item > .outline-label:before {
    counter-increment: h6;
    content: counter(h1) "." counter(h2) "." counter(h3) "." counter(h4) "." counter(h5) "." counter(h6) " ";
}

/** override the default style for focused headings */
#write > h3.md-focus:before,
#write > h4.md-focus:before,
#write > h5.md-focus:before,
#write > h6.md-focus:before,
h3.md-focus:before,
h4.md-focus:before,
h5.md-focus:before,
h6.md-focus:before {
    color: inherit;
    border: inherit;
    border-radius: inherit;
    position: inherit;
    left: initial;
    float: none;
    top: initial;
    font-size: inherit;
    padding-left: inherit;
    padding-right: inherit;
    vertical-align: inherit;
    font-weight: inherit;
    line-height: inherit;
}
```

![](../attachments/Pasted%20image%2020241113151503.png)