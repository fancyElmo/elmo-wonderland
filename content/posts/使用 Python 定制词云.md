---
title: "使用python定制词云"
date: 2019-12-24T17:07:17+08:00
draft: true
toc: true
images:
tags: ["tutorial"]
---

# 一、实验简介

#### 1.1 实验内容

在互联网时代，人们获取信息的途径多种多样，大量的信息涌入到人们的视线中。如何从浩如烟海的信息中提炼出关键信息，滤除垃圾信息，一直是现代人关注的问题。在这个信息爆炸的时代，我们每时每刻都要更新自己的知识储备，而网络是最好的学习平台。对信息过滤和处理能力强，学习效率就会得到提高。“词云”就是为此而诞生的。“词云”是对网络文本中出现频率较高的“关键词”予以视觉上的突出，形成“关键词云层”或“关键词渲染”，从而过滤掉大量的无意义信息，使浏览者只要一眼扫过词云图片就可以领略文章或者网页内容的主旨。不仅如此，一幅制作精美的词云图片，可以起到一图胜千言的效果，在报告或者`PPT`中适当的使用词云，会使表达更清晰充分，为演讲者表达的意义加分。本实验将使用`Python3`的`wordcloud`扩展包制作词云，生成图片保存。并介绍如何改进`wordcloud`扩展包使其能显示中文字符，最后介绍如何使用自己喜欢的图片定制词云图片轮廓。

#### 1.2 实验知识点

- 制作词云的基本步骤和原理
- `Python3`代码实现词云制作
- `wordcloud`扩展包的使用
- 使用自定义图片制作词云，分析《三体》I、 II、 III的关键词

#### 1.3 实验环境

该实验在`ubuntu14.04`下完成，由于`Python`具有跨平台特性，该实验的代码也可以运行于`Windows`和`Mac`系统上，只需要对字体部分做相应处理即可。

- `python3.5`
- `numpy 1.14.3`
- `matplotlib 2.2.2`
- `Pillow 5.1`
- `wordcloud 1.4.1`
- `python3-tk 3.5.1`

#### 1.4 适合人群

本课程难度为一般，属于初级级别课程，适合具有`Python3`基础的用户，熟悉`python3`基础知识加深巩固。

#### 1.5 代码获取

你可以通过下面命令将代码下载到实验楼环境中，作为参照对比进行学习。

```bash
$ wget http://labfile.oss.aliyuncs.com/courses/756/simple.py
$ wget http://labfile.oss.aliyuncs.com/courses/756/my_word_cloud.py
```



# 二、实验原理

词云的原理是对输入的文本数据进行词频统计，根据词汇出现频率的不同，按不同比例显示出词汇，生成图片。频率高的词汇显示的大，频率低的词汇显示的小。文本数据可以是本地数据，也可是爬虫动态从网络中获取的。



# 三、开发准备

打开`Xfce`终端，进入 `Code` 目录，创建 `work` 文件夹, 将其作为课程的工作目录。下载并安装实验需要的扩展包 。如果大家平时想在自己的电脑上进行实验，无论是`Windows`还是`Linux`还是`Mac`，都强烈推荐安装`Anaconda`,这是一个`Python`的科学计算包，里面几乎包含了常用的所有扩展包，不用自己费力安装了，该软件由`Python`之父带头维护,三个平台同时更新。

```bash
$ cd Code && mkdir work && cd work
$ sudo apt-get update
$ sudo pip3 install numpy
$ sudo pip3 install matplotlib
$ sudo apt-get install libtiff5-dev libjpeg8-dev zlib1g-dev \
    libfreetype6-dev liblcms2-dev libwebp-dev tcl8.6-dev tk8.6-dev python-tk  #安装Pillow依赖包
$ sudo pip3 install Pillow
$ sudo apt-get install python3-tk
```

下载小说《三体》I、 II、 III。

```bash
$ wget http://labfile.oss.aliyuncs.com/courses/756/santi.txt
$ wget http://labfile.oss.aliyuncs.com/courses/756/santi2.txt
$ wget http://labfile.oss.aliyuncs.com/courses/756/santi3.txt
```

安装`wordcloud`扩展包。

```bash
$ sudo apt-get install python3-dev
$ sudo pip3 install wordcloud
```



# 四、实验步骤

### 4.1 简单测试

运行一个简单工程，测试扩展包安装是否正常

在对《三体》进行处理之前，我们先运行一下官方的示例程序，确保扩展包安装正常，程序能够正常工作。在`work`目录下新建`simple.py`文件：

```bash
$ cd /home/shiyanlou/Code/work
$ touch simple.py
```

代码如下：

```python
#!/usr/bin/env python3
"""
Minimal Example
===============
Generating a square wordcloud from the US constitution using default arguments.
"""

from os import path
from wordcloud import WordCloud

d = path.dirname(__file__)

# Read the whole text.
text = open(path.join(d, 'constitution.txt')).read()

# Generate a word cloud image
wordcloud = WordCloud().generate(text)

# Display the generated image:
# the matplotlib way:
import matplotlib.pyplot as plt
plt.imshow(wordcloud)
plt.axis("off")

# lower max_font_size
wordcloud = WordCloud(max_font_size=40).generate(text)
plt.figure()
plt.imshow(wordcloud)
plt.axis("off")
plt.show()
```

由代码可见，程序运行时会搜寻脚本所在的路径下的文本文件`“constitution.txt”`，所以我们在运行脚本前需要将这个文本放入`work`文件夹下面。 通过下面的命令下载文本：

```bash
$ wget http://labfile.oss.aliyuncs.com/courses/756/constitution.txt
```

运行脚本：

```bash
$ python3 simple.py
```

如果扩展包安装一切正常，程序将输出如下窗口：

![4.1-1](https://doc.shiyanlou.com/document-uid600404labid2521timestamp1528268968172.png)

至此，我们得到了一个英文词云。



### 4.2 解决中文显示问题

我们已经成功安装了`wordcloud`扩展包，并成功运行了一个示例文件。但是这个示例文件有很多问题，首先，显示的是英文字符，在面对中国同事或者老板做报告和分享时，使用英文的词云明显不合适，而且很多文本本身就是中文词汇，没法制作成英文词云；词云的外轮廓显示的方方正正中规中矩，比较呆板，没有美感。以上问题，我们一一来解决。

首先，我们先解决中文显示的问题。 我们先尝试一下，把一本小说输入到`simple.py`文件中并声明`gbk`编码，看看输出结果是什么样子的。我们只需要修改源文件成为下面这样，注意在文件头部要声明文件用`utf-8`编码：

```python
#!/usr/bin/env python3
#-*- coding: utf-8 -*-
"""
Minimal Example
===============
Generating a square wordcloud from the US constitution using default arguments.
"""

from os import path
from wordcloud import WordCloud

d = path.dirname(__file__)

# Read the whole text.
#text = open(path.join(d, 'constitution.txt')).read()
text = open('santi.txt','r',encoding='gbk').read()

# Generate a word cloud image
wordcloud = WordCloud().generate(text)

# Display the generated image:
# the matplotlib way:
import matplotlib.pyplot as plt
plt.imshow(wordcloud)
plt.axis("off")

# lower max_font_size
wordcloud = WordCloud(max_font_size=40).generate(text)
plt.figure()
plt.imshow(wordcloud)
plt.axis("off")
plt.show()
```

运行代码得到的输出是这个样子的：

![4.2-1](https://doc.shiyanlou.com/document-uid359948labid2521timestamp1486332784125.png) ![4.2-2](https://doc.shiyanlou.com/document-uid359948labid2521timestamp1486332794449.png)

这怎么回事呢? 仔细看图片中出现的都是框框，说明我们正确识别了汉字，但是汉字怎么显示成这样了呢?这是因为`wordcloud`没有找到用于显示汉字的字体啊！ 我们都知道`Ubuntu`是外国人搞出来的系统，所以对中文的支持肯定没有对英文那么好，尤其是字体什么的，更少了。而`wordcloud`也是个外国人开发的词云库，这俩货都没考虑到显示中文的问题。 但是呢，既然`python`能显示中文，那么`wordcloud`就应该也可以的。我们来想想办法解决这个问题。 仔细的看一下我们的代码，我们发现，生成词云的关键环节是这一句：

```python
wordcloud = WordCloud(max_font_size=40).generate(text)
```

我们仔细看看这个类，传给它的参数里有个关键参数：

```python
    font_path : string
        Font path to the font that will be used (OTF or TTF).
        Defaults to DroidSansMono path on a Linux machine. If you are on
        another OS or don't have this font, you need to adjust this path.
```

恩，我们发现可以指定一个字体文件给它，代替默认的字体显示词云。问题转化为我们要找一个`linux`下支持汉字的字体文件。我这边随手找了一个`ubuntu`系统中安装的字体`DroidSansFallbackFull.ttf`，为了防止在别的`linux`系统中没有这个字体而带来麻烦，干脆直接把字体文件放在`work`文件夹下让它跟着源文件走，这样就不会出现找不到的情况啦。 通过下面的命令下载字体文件：

```bash
$ wget http://labfile.oss.aliyuncs.com/courses/756/DroidSansFallbackFull.ttf
```

将字体文件放在`/home/shiyanlou/Code/work`目录下，然后修改源代码，首先找到我们自己的字体文件：

```python
import os
font=os.path.join(os.path.dirname(__file__), "DroidSansFallbackFull.ttf")
```

然后在我们的源程序中，实例化`wordcloud`类的两个地方，指定`wordcloud`使用我们自己的字体文件：

```python
wordcloud = WordCloud(font_path=font).generate(text)
wordcloud = WordCloud(font_path=font,max_font_size=40).generate(text)
```

好的，修改完以后我们再看一下执行效果：

![4.2-3](https://doc.shiyanlou.com/document-uid359948labid2521timestamp1486410625924.png) ![4.2-4](https://doc.shiyanlou.com/document-uid359948labid2521timestamp1486410637257.png)

OK,我们期望的目的达到了！



### 4.3 定制词云

我们经常在网上看到别人家的词云都是奇形怪状的，像下面这样：

![4.3-1](https://doc.shiyanlou.com/document-uid359948labid2521timestamp1486411305118.png)

所以看着我们自己方方正正的词云，是不是感觉太中规中矩了？都不好意思拿出手了吧？ 没关系，我们也可以做一个不规则边缘的词云！ 为了达到一个定制词云的效果，我们需要一个图片作为`mask`，这个`mask`的作用就是为我们的词云提供一个空间，让我们的词云只在这个空间里显示，这就达到了类似上面词云的效果。 我们的`mask`图片是星球大战的士兵的头盔，长成这个样子：

![4.3-2](https://doc.shiyanlou.com/document-uid359948labid2521timestamp1486431670827.png)

做实验时请在这里下载：

```bash
$ wget http://labfile.oss.aliyuncs.com/courses/756/stormtrooper_mask.png
```

为此，我们需要修改我们的代码，增加图片`mask`，修改后的代码如下：

```python
#!/usr/bin/python3
#-*- coding: utf-8 -*-
"""
Using custom colors
====================
Using the recolor method and custom coloring functions.
"""

import numpy as np
from PIL import Image
from os import path
import matplotlib.pyplot as plt
import random
import os

from wordcloud import WordCloud, STOPWORDS


font=os.path.join(os.path.dirname(__file__), "DroidSansFallbackFull.ttf")

def grey_color_func(word, font_size, position, orientation, random_state=None, **kwargs):
    return "hsl(0, 0%%, %d%%)" % random.randint(60, 100)

d = path.dirname(__file__)


mask = np.array(Image.open(path.join(d, "stormtrooper_mask.png")))


text = open('santi.txt','r',encoding='gbk').read()

# preprocessing the text a little bit
text = text.replace("程心说", "程心")
text = text.replace("程心和", "程心")
text = text.replace("程心问", "程心")

# adding movie script specific stopwords
stopwords = set(STOPWORDS)
stopwords.add("int")
stopwords.add("ext")

wc = WordCloud(font_path=font,max_words=2000, mask=mask, stopwords=stopwords, margin=10,
               random_state=1).generate(text)
# store default colored image
default_colors = wc.to_array()
plt.title("Custom colors")
plt.imshow(wc.recolor(color_func=grey_color_func, random_state=3))
wc.to_file("a_new_hope.png")
plt.axis("off")
plt.figure()
plt.title("三体-词频统计")
plt.imshow(default_colors)
plt.axis("off")
plt.show()
```

这段代码对词云的显示结果做了一点点修改，例如，“程心说”应该算是“程心”的词频，而不应该独立计算，所以程序中做了一个简单的替换。 这段代码的运行结果如下： ![4.3-3](https://doc.shiyanlou.com/document-uid359948labid2521timestamp1486411917043.png) ![4.3-4](https://doc.shiyanlou.com/document-uid359948labid2521timestamp1486411927369.png)

我们使用了星球大战的武士的头盔作为词云形状，怎么样，看起来还不错吧？

然而，我们的任务并没有结束。我们说了要自己定制词云，那么就连这个mask图片也可以根据我们自己的喜好进行定制。 为了在`ubuntu`下将我们喜欢的图片作为`mask`图片，我们需要安装`gimp`软件：

```bash
$ sudo apt-get install gimp
```

然后我随意上网百度了一个美女图片，就是下面这张：

![4.3-5](https://doc.shiyanlou.com/document-uid359948labid2521timestamp1486412304687.png)

请使用下面命令获取该美女图片的`jpg`格式：

```bash
$ wget http://labfile.oss.aliyuncs.com/courses/756/04.jpg
```

图片有什么要求呢？恩，最好背景是纯色的，这样好处理，当然如果有`PS`高手帮忙的话，背景是什么也无所谓了。我们用`gimp`软件对这幅图片进行处理，把除了女孩之外的背景改为白色，就可以作为mask图片使用了。好下面我们动手开始制作。

在命令行中输入`gimp`就可以进入`gimp`界面：

首先，在图层页面下，右键这张图，增加`alpha`图层：

![4.3-6](https://doc.shiyanlou.com/document-uid359948labid2521timestamp1486412821276.png)

然后，使用左侧工具栏中的魔术棒工具，在这个姑娘身上随便画一下，就选中了这个姑娘：

![4.3-7](https://doc.shiyanlou.com/document-uid359948labid2521timestamp1486412904085.png)

然后，在空白区域，右键，编辑，使用白色作为背景颜色填充图片：

![4.3-8](https://doc.shiyanlou.com/document-uid18510labid2521timestamp1487060801110.png)

搞定之后，我们把图片导出来：

![4.3-9](https://doc.shiyanlou.com/document-uid18510labid2521timestamp1487060863131.png)

注意导出的时候选择`png`格式，如下设置:

![4.3-10](https://doc.shiyanlou.com/document-uid18510labid2521timestamp1487061095829.png)

然后我们就得到了这个mask图片。我将其重命名为`04.png`我们修改代码使用这个图片作为`mask`图片：

```python
mask = np.array(Image.open(path.join(d, "04.png")))
```

请使用如下命令获取图片`04.png`：

```bash
$ wget http://labfile.oss.aliyuncs.com/courses/756/04.png
```

再看我们的运行结果：

![4.3-11](https://doc.shiyanlou.com/document-uid359948labid2521timestamp1486413524530.png)

![4.3-12](https://doc.shiyanlou.com/document-uid359948labid2521timestamp1486413535392.png)

好的，至此我们就得到了定制的词云。



# 五、实验总结

我们使用`wordcloud`扩展包实现了定制词云，通过指定字体文件解决了`wordcloud`默认不能显示中文的问题，进一步，使用`gimp`软件实现了自定义`mask`图片的功能。最后，使用自定义的词云分析了小说《三体》的词频，制作了词云。