---
title: jemdoc
date: 2017-08-21 21:00:51
tags:
- tutorial
---

### jemdoc 介绍

[jemdoc](http://jemdoc.jaboc.net/index.html)是用来制作个人主页的工具，非常nice的[Boyd](http://stanford.edu/~boyd/)也用它来制作个人主页。上科大的老师也经常用这玩意。jemdoc制作出来的网页虽然不是特别好看，但是可以节省很多时间，效果也算简介清爽（没特色）。

先把[jemdoc.py](http://jemdoc.jaboc.net/dist/jemdoc.py)下载下来。下载好之后，可以把这个文件拷到/usr/bin下或者alias jemdoc=somepath。另外还需要一份CSS文件，官网提供了[jemdoc.css](http://jemdoc.jaboc.net/dist/jemdoc.css)。

![jemdoc-command](https://ww3.sinaimg.cn/large/006tKfTcly1firo0rzxvlj30ex02ct8v.jpg)

假如你想生成的网页为index.html，那么先新建index.jemdoc文件(和jemdoc.css在同一个文件夹下)。这个文件里面的内容是类似于markdown的格式，参考这个[网页](http://jemdoc.jaboc.net/example.html)最右栏。特殊的格式，比如加粗，公式等等可以看网站提供的[cheetsheet](http://jemdoc.jaboc.net/cheatsheet.html)。

如果想添加菜单栏，需要新建一个文件，叫MENU。举一个例子：

```
jemdoc
    home                [index.html]
    download            [download.html]
    revision history    [revision.html]
    contact             [contact.html]
```

这个例子中，jemdoc就是这栏的名字，下面的home这些是具体的超链接，最终的效果是这样的(第三行的名字应该不对)：

![menu](https://ww3.sinaimg.cn/large/006tKfTcly1firq60j2euj305k04cmx7.jpg)

后面的index.html，download.html都是用jemdoc生成的，当然也可以用其他的HTML文件代替。但是如果这些.html文件也想使用同一个菜单，则必须在对应的.jemdoc文件第一行加一下代码:

``` 
# jemdoc: menu{MENU}{index.html}
```

其中的index.html替换为当前.html文件的名字。

每次修改好.jemdoc文件后都要用```jemdoc ***.jemdoc```命令更新下，进而生成最新的***.html文件。

基本最先用的地方其实就这么多，其他内容参考官网就好。