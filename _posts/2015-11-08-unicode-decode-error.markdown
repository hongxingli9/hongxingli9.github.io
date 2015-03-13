---
layout: post
title:  "控制台cocos run -p web命令后输出UnicodeDecodeError"
date:   2014-11-08 
categories: cocos2d-js
---
这几天学习着cocos2d-js引擎，公司未来安排做些light app，H5小游戏即将成为我的任务。能提高开发效率的其中一个途径就是使用一款高效的引擎，冲着资料文档齐全的cocos2d, 决定学习学习。

这里我就不介绍cocos2d-js，其实我是一知半解的，也说不好，哈哈哈.......

首先就是安装Python,，配置Python环境变量， 才能运行cocos2d-js里的setup.py。然后我通过命令行在一个目录下新建一个project, cocos new -l js成功没问题。

接下来cocos run -p web运行的时候，输出UnicodeDecodeError: 'ascii' codec can't decode byte 0xb0 in position 1: ordinal not in range(128)。

怎么办，google到的解决方案并没有凑效。第二天晚上又试着去找答案。

很快地，找到了解决的方法，

打开C:\Python27\Lib下的 mimetypes.py 文件，'default_encoding = sys.getdefaultencoding()'。

在这行前面添加三行：

{% highlight python %}
if sys.getdefaultencoding() != 'gbk':      
    reload(sys)   
    sys.setdefaultencoding('gbk')  
default_encoding = sys.getdefaultencoding()
{% endhighlight %}

至于为什么这样我暂时不太清楚，原博主认为与注册表有关，可能与某些国产软件对注册表的改写的gbk格式导致python无法进行第三方库的安装操作。 

不管怎样，问题得到解决，感谢万能的google.
