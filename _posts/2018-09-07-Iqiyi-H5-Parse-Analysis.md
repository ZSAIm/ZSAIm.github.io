---
layout: post
title:  "爱奇艺视频H5解析分析过程"
categories: JavaScript Fiddler
tags: 爱奇艺 解析
author: ZSAIm
---

* content
{:toc}

## 说明


### 文章说明

* 模拟爱奇艺视频基于H5播放方式的视频解析过程，并且有效的解析视频真实地址。
* 由于在写这文章时是基于爱奇艺游客模式分析的，所以并不具备下载VIP视频的能力。
* 如果有兴趣的可以进入以上的Github项目地址进行提交改进。






# 正文
### 文章所用工具

工具名 | 说明
-------- | -----
**Fiddler** | 抓包工具 
**Chrome** | 浏览器
Postman | HTTP请求调试
Webstorm | JS调试
Notepad++ | 文本编辑器
UltraCompare | 文本比较器


### 分析过程

#### 准备工作
***
##### 清除浏览器缓存、COOKIE
***
##### 启用Fiddler的抓包功能
***
##### 打开一个爱奇艺视频网页


本文章以 ***https://www.iqiyi.com/v_19rqz97sb0.html*** 展开分析。
***
##### 等待视频加载开始
1. 等待视频开始缓存，为了保证已经加载视频，建议等待广告结束之后再进行分析数据。
2. 使用fiddler的过滤器，将无用的抓包数据过滤掉。（\*.CSS | \*. \*（图片类））

![图0.1](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/0-1.png)
***【图0.1】***
***


#### 分析数据

##### 1、 筛选视频流数据项

可以通过以下两点筛选出视频数据流项

表头 | 说明
------- | -------
 **Body** | 数据长度较长的项
 **Content-Type** | `application/octet-stream` - 文件流格式的一般类型

通过以上方法可以找到第`121`项（如下图）
![图1.1](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/1-1.png)
***【图1.1】***


可以如下图右键保存响应数据来查看所对应项是否是所要的视频数据项

![图1.2](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/1-2.png)
***【图1.2】***

事实得到了只是一个广告视频
既然这一项不是目标项，那么继续往下寻找。
可以得到下图的第`382`项。
![图1.3](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/1-3.png)
***【图1.3】***

----------*后面你会发现第`337`项是视频资源的文件头。*

同样地保存视频文件，检查下是否是所需的视频文件。

==结果是视频文件显示损坏无法播放。==

这个时候注意你不应该就将此项排除掉，因为
> 任何未被证伪的论点都不应该被确定是错误的。

本着这个说法，所以我选择跳过这一项，而不是排除掉它。

接着往下走，发现并没有能够播放并且是所需的文件流数据。

所以我决定往回走，找会那一不能播放的一项进行更深一步的分析。

相信使用过视频解析工具的都知道很多视频网站都会将视频数据切片，那么有没有可能是由于切片的原因造成了视频不能被播放。

**当你确信视频文件是被切片了的，你就可以忽略以下列表的解释**

- 为了证明文件确实被切片了，那么可以通过以下方式来证明视频确实切片了。

- 这一次不再使用视频播放器来进行播放文件流，而是通过二进制查看器来查看数据。

- 通过查看第`382`项的响应数据发现，此文件流数据并没有文件头。[^1]

- 可以推测的是该文件头存在于某一项中。

- 找到这一项，然后使用二进制查看器查看其文件流数据的数据，查看其是否是视频文件的文件头。

既然如此那么，那么既然怀疑视频是被切片了的，那么就找同类URL。

点击打开Fiddler的第`382`项，观察其请求头（Request Headers）。

寻找比较独特的一字符串。并使用Fiddler的搜索工具进行搜索。[^2]

![图1.4](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/1-4.png)
***【图1.4】***

>GET /r/bdcdngdct.inter.71edge.com/videos/v0/20180831/44/87/==611f0244cfe1a64d165b2f630be2e9fc==.f4v?key=0433de19d0c1b75057f6fc64c676e6ded&dis_k=c17fe85c329bf7dcd883071bb8d55fa5&dis_t=1535880044&dis_dz=CT-GuangDong_GuangZhou&dis_st=42&src=iqiyi.com&uuid=7908d205-5b8bab6c-191&rn=1535880043829&qd_tm=1535880032356&qd_tvid=1294428000&qd_vipdyn=0&qd_k=a07fc56691bd8558b5e9f23e9119b36a&cross-domain=1&qd_aid=220327201&qd_uid=&qd_stert=0&qypid=1294428000_02020031010000000000&qd_p=7908d205&qd_src=01010031010000000000&qd_index=1&qd_vip=0&qyid=836c674bfa7e7387a323d314bfb4a875&pv=0.1&qd_vipres=0&range=8192-10431487
HTTP/1.1

既然是独特的字符串，那么下面这一串就很有特点`611f0244cfe1a64d165b2f630be2e9fc`

进行搜索如下图操作

![图1.5](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/1-5.png)
***【图1.5】***

搜索结果是`图2.3`，很多的同类项（文件流数据格式），然后你找最前的一项。会发现就我前面说的那样第`337`项。

之后你可以使用二进制查看器来查看这一项是否是文件头。

到目前为止我们已经找到了视频项了，这已经足够了吗？

当然不，我们要的不仅仅是这一视频文件，而是知道为什么浏览器能知道并且找到这视频文件流。既然如此如果我们要得到这视频文件流，我们必须先要知道这一串请求头。所以下面的工作就是要构造请求头了。

***

##### 2、 构造视频项请求头

以上筛选的视频项请求头如下

>GET /r/bdcdngdct.inter.71edge.com/videos/v0/20180831/44/87/**611f0244cfe1a64d165b2f630be2e9fc**.f4v?key=0433de19d0c1b75057f6fc64c676e6ded&dis_k=c17fe85c329bf7dcd883071bb8d55fa5&dis_t=1535880044&dis_dz=CT-GuangDong_GuangZhou&dis_st=42&src=iqiyi.com&uuid=7908d205-5b8bab6c-191&rn=1535880043829&qd_tm=1535880032356&qd_tvid=1294428000&qd_vipdyn=0&qd_k=a07fc56691bd8558b5e9f23e9119b36a&cross-domain=1&qd_aid=220327201&qd_uid=&qd_stert=0&qypid=1294428000_02020031010000000000&qd_p=7908d205&qd_src=01010031010000000000&qd_index=1&qd_vip=0&qyid=836c674bfa7e7387a323d314bfb4a875&pv=0.1&qd_vipres=0&range=8192-10431487
HTTP/1.1

那么先从最后一层路径开始构造
###### 进入第一层构造
我们将以下构造称为 ==**构造①**==
> 611f0244cfe1a64d165b2f630be2e9fc.f4v?key=0433de19d0c1b75057f6fc64c676e6ded&dis_k=c17fe85c329bf7dcd883071bb8d55fa5&dis_t=1535880044&dis_dz=CT-GuangDong_GuangZhou&dis_st=42&src=iqiyi.com&uuid=7908d205-5b8bab6c-191&rn=1535880043829&qd_tm=1535880032356&qd_tvid=1294428000&qd_vipdyn=0&qd_k=a07fc56691bd8558b5e9f23e9119b36a&cross-domain=1&qd_aid=220327201&qd_uid=&qd_stert=0&qypid=1294428000_02020031010000000000&qd_p=7908d205&qd_src=01010031010000000000&qd_index=1&qd_vip=0&qyid=836c674bfa7e7387a323d314bfb4a875&pv=0.1&qd_vipres=0&range=8192-10431487

既然是要构造他那么肯定是要知道这一串字符到底是哪里得到的，或者说是怎么得到的啦，当然我们不能直接去搜索这一整串字符，既然这样，不如先从特殊的字符串开始。

（其实这一部分上面已经做了，也就是搜索`611f0244cfe1a64d165b2f630be2e9fc`）

那么就找第一出现这个字符串的这一项，（注意前面说的第一项和这里的第一项不一样，前面的第一项是文件流形式的第一项）

结果如下图

![图2.1](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-1.png)
***【图2.1】***

点开它，查看这一字符串出现在请求数据还是响应数据。

你可以通过以下方式查找

![图2.2](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-2.png)
***【图2.2】***

![图2.3](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-3.png)

***【图2.3】***

![图2.4](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-4.png)
***【图2.4】***

既然发现了第一次出现的字符串在这个地方，那么你就必须要知道这个请求链接才能知道这一字符串，所以现在是进行了下一层的构造。

###### 挂起第一层构造
所以我们先挂起 ==**构造①**== 。

---
###### 进入第二层构造
 进行如下请求头 ==**构造②**== ，

> GET
> /jp/dash?tvid=1294428000&bid=300&vid=5e54f1fec36034f67521abf755dd3f93&src=01010031010000000000&vt=0&rs=1&uid=&ori=pcw&ps=0&tm=1535880031201&qd_v=1&k_uid=836c674bfa7e7387a323d314bfb4a875&pt=0&d=0&s=&lid=&cf=&ct=&authKey=4c31a25989ea7e678b670125e6ee5acf&k_tag=1&ost=0&ppt=0&dfp=&locale=zh_cn&prio=%7B%22ff%22%3A%22f4v%22%2C%22code%22%3A2%7D&pck=&k_err_retries=0&k_ft1=549755813888&bop=%7B%22version%22%3A%227.0%22%2C%22dfp%22%3A%22%22%7D&callback=Q838114a04d5d4d3adb265513f3244a36&ut=0&vf=a07fc56691bd8558b5e9f23e9119b36a
HTTP/1.1

整理键和值如下表格

index | key | value
------- | ----- | ---------
1 | tvid | 1294428000
2 | bid | 300
3 | vid | 5e54f1fec36034f67521abf755dd3f93
4 | src | 01010031010000000000
5 | vt | 0
6 | rs | 1
7 | uid | 
8 | ori | pcw
9 | ps | 0
10 | tm | 1535880031201
11 | qd_v | 1
12 | k_uid | 836c674bfa7e7387a323d314bfb4a875
13 | pt | 0
14 | d | 0
15 | s | 
17 | lid | 
18 | cf | 
19 | ct | 
20 | authKey | 4c31a25989ea7e678b670125e6ee5acf
21 | k_tag | 1
22 | ost | 0
23 | ppt | 0
24 | dfp | 
25 | locale | zh_cn
26 | prio | %7B%22ff%22%3A%22f4v%22%2C%22code%22%3A2%7D
27 | pck | 
28 | k_err_retries | 0
29 | k_ft1 | 549755813888
29 | bop | %7B%22version%22%3A%227.0%22%2C%22dfp%22%3A%22%22%7D
30 | callback | Q838114a04d5d4d3adb265513f3244a36
31 | ut | 0
32 | vf | a07fc56691bd8558b5e9f23e9119b36a

同样的也可以寻找比较“特殊”的字符串，这一次我们要先找`authKey`这一参数，因为通过英文auth - key可以大概明白这是一个加密密钥。

所以，可以搜索`autuKey`，为什么不是后面的值`4c31a25989ea7e678b670125e6ee5acf`呢？因为既然是密钥，有怎么会是明文呢？如果你还是不相信，可以尝试搜索一下。

![图2.5](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-5.png)
***【图2.5】***

![图2.6](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-6.png)
***【图2.6】***

第`12`项是一个JS脚本文件，所以推测autuKey的值是通过脚本生成的，所以我们需要找到生成这一密钥的原理。

（以下是这一项的JS代码，搜索autuKey如下位置）
![图2.7](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-7.png)
***【图2.7】***

回头看一下上面的表格，然后对比一下这些参数是不是很眼熟呢？大部分的参数都已经给出来了。

而我们要找的`4401`行的`authKey:n(n("") + I + A)`

所以我们可以分析`n(n("") + I + A)`返回的值，就能知道`authKey`的值了。

可不可以直接去分析JS呢？答案是当然可以的，但是要知道这不仅仅是时间的问题，当你分析压缩了的函数变量名的时候，你就该头疼了。

所以我们使用谷歌的调试工具去分析。

下图底部的 `{}`可以对代码进行格式化，这样分析起来会轻松点。
![图2.8](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-8.png)
***【图2.8】***

搜索找到`authKey`，然后在此处下断点`5420`行，如下图。
（如果你能够顾及的话，你可以同时分析多个参数。然后你要刷新页面（可以直接按F5刷新），让他停在断点处。）
![图2.9](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-9.png)
***【图2.9】***

刷新后停在e处，你也可以直接让指针触碰函数n，然后你就能看到

![图2.10](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-10.png)
***【图2.10】***

点击上面的链接进去看这个函数

![图2.11](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-11.png)
***【图2.11】***

看来这个n是调用了一串这些函数返回的结果。`u(o(c(e),e.length * g))`继续往前面找函数`c()`，触碰停留或者单步执行，进去得到如下图

![图2.12](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-12.png)
***【图2.12】***

看来函数c是到底了（也就是说没有再往下调用的自定义函数了）

然后是函数o，其实往前看就能找到函数o了。

![图2.13](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-13.png)
***【图2.13】***

然后到函数u，在`图2.12`就能知道函数u

目前为止了解了`authKey:n(n("") + I + A)`中的函数n。

那么接下来就要知道变量`I`和`A`的值了

![图2.14](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-14.png)
***【图2.14】***

看参数名字`tm`大概可以知道这是一个时间戳，而且停留在`I`也发现

![图2.15](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-15.png)
***【图2.15】***

所以这确实是一个时间戳，当然也会返回去查看给`I`赋值的那一部分代码。

由`tvid: A`可以得`A`就是参数`tvid`的值。

我们先返回去fiddler看下这个`tvid`在前面的项中有没有提到过。（当然也可以分析生成A的那一部分代码，但是如果能够直接得到的就不要傻乎乎的去分析代码了）

通过请求头，也就是 ==**构造②**== 中的`tvid`项的值去搜索。

![图2.16](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-16.png)
***【图2.16】***

![图2.17](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-17.png)
***【图2.17】***

![图2.18](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-18.png)
***【图2.18】***

所以我们得到了`tvid`的值

同样的，我们要知道`tvid`的值就要知道其请求头，很高兴的发现这一个请求头就是我们的播放的爱奇艺视频的地址，所以这个`tvid`是白送的，既然如此接着往下构造其他参数。

![图2.19](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-19.png)
***【图2.19】***

下一步应该分析`k_uid=836c674bfa7e7387a323d314bfb4a875`

既然这里的参数和我们的 ==**构造②**== 需求很相像，那么在进行下一步之前不如先看一下还缺什么吧。

![图2.20](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-20.png)
***【图2.20】***

通过对比发现，如上图的嘴边标注有蓝点的五个参数是缺少的。

上面哪一些已经列出来的参数，同理可以通过分析JS或者搜索得到，为了减少篇幅，所以不再分析，而把重点放在那五个没有在这上面的参数上。

第一个： `k_ft1=549755813888`

首先要搜索一下值有没有在fiddler之前项中出现过，结果是没有找到。

所以我们只能去搜索`k_ft1`，结果如下图

![图2.21](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-21.png)
***【图2.21】***
![图2.22](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-22.png)
***【图2.22】***
![图2.23](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-23.png)
***【图2.23】***

看到这里`e.k_ft1 = v.getFT1()`，这里，`getFT1`是没有经过压缩的，所以你不一定遇到参数就马上用谷歌浏览器的开发者工具取调试他，在这种情况下你可以在notepad++里面看一下，找一下这个函数。

![图2.24](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-24.png)
***【图2.24】***

![图2.25](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-25.png)
***【图2.25】***

![图2.26](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-26.png)
***【图2.26】***

第二个参数：`bop=%7B%22version%22%3A%227.0%22%2C%22dfp%22%3A%22%22%7D`

同样的方法，先搜索值，但是注意的是这里面的参数值是通过urlencode的，所以先解码再搜索。
可以使用fiddler里面的textwizard工具来解码
![图2.27](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-27.png)
***【图2.27】***

发现并没有这个，但是你可以搜索里面的`version`或者`dfp`，为什么呢？因为这很有可能是类似的JS片段。
`{‘version’:e.version,‘dfp’:e.dfp}`
，我建议搜索`dfp`。但是说道理，我们也可以直接搜索参数名`bop`而不纠结于参数值。

然后有意思的是，我们想太多了，发现其实都在下面如图这里。

![图2.28](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-28.png)
***【图2.28】***

所以上图能找到前面的几个参数的依据了。

那么就剩下的`callback`和`vf`了

`callback`可以使用同样的方法即使用谷歌浏览器的开发者工具调试一下，看一下什么时候获得了`callback`。

下图是`vf`的

![图2.29](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-29.png)
***【图2.29】***

所以到目前为止，完成了第二层构造，也就是 ==**构造②**==。

> 到现在，我们已经构造出来了
GET
/jp/dash?tvid=1294428000&bid=300&vid=5e54f1fec36034f67521abf755dd3f93&src=01010031010000000000&vt=0&rs=1&uid=&ori=pcw&ps=0&tm=1535880031201&qd_v=1&k_uid=836c674bfa7e7387a323d314bfb4a875&pt=0&d=0&s=&lid=&cf=&ct=&authKey=4c31a25989ea7e678b670125e6ee5acf&k_tag=1&ost=0&ppt=0&dfp=&locale=zh_cn&prio=%7B%22ff%22%3A%22f4v%22%2C%22code%22%3A2%7D&pck=&k_err_retries=0&k_ft1=549755813888&bop=%7B%22version%22%3A%227.0%22%2C%22dfp%22%3A%22%22%7D&callback=Q838114a04d5d4d3adb265513f3244a36&ut=0&vf=a07fc56691bd8558b5e9f23e9119b36a
HTTP/1.1 
###### 结束第二层构造

---

###### 继续第一层构造
>GET /r/bdcdngdct.inter.71edge.com/videos/v0/20180831/44/87/**611f0244cfe1a64d165b2f630be2e9fc**.f4v?key=0433de19d0c1b75057f6fc64c676e6ded&dis_k=c17fe85c329bf7dcd883071bb8d55fa5&dis_t=1535880044&dis_dz=CT-GuangDong_GuangZhou&dis_st=42&src=iqiyi.com&uuid=7908d205-5b8bab6c-191&rn=1535880043829&qd_tm=1535880032356&qd_tvid=1294428000&qd_vipdyn=0&qd_k=a07fc56691bd8558b5e9f23e9119b36a&cross-domain=1&qd_aid=220327201&qd_uid=&qd_stert=0&qypid=1294428000_02020031010000000000&qd_p=7908d205&qd_src=01010031010000000000&qd_index=1&qd_vip=0&qyid=836c674bfa7e7387a323d314bfb4a875&pv=0.1&qd_vipres=0&range=8192-10431487
HTTP/1.1

通过构造第二层请求头，我们得到了字符串`611f0244cfe1a64d165b2f630be2e9fc`和其响应中剩余的数据

![图2.30](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-30.png)
***【图2.30】***

而要知道的是，我们的目标就是构造一的请求头，而构造二只是为了寻找字符串`611f0244cfe1a64d165b2f630be2e9fc `。

现在回过头来发现第二层的构造不仅得到了字符串`611f0244cfe1a64d165b2f630be2e9fc `，还得到了第一层构造的很多参数。

比较一下

![图2.31](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-31.png)
***【图2.31】***

通过对比发现还确实缺了不少。不如先从key开始找
`key=0433de19d0c1b75057f6fc64c676e6ded`

寻找`0433de19d0c1b75057f6fc64c676e6ded`，得到下图

![图2.32](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-32.png)
***【图2.32】***

![图2.33](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-33.png)
***【图2.33】***

下面是第一层构造
>GET /r/bdcdngdct.inter.71edge.com/videos/v0/20180831/44/87/==611f0244cfe1a64d165b2f630be2e9fc==.f4v?key=0433de19d0c1b75057f6fc64c676e6ded&dis_k=c17fe85c329bf7dcd883071bb8d55fa5&dis_t=1535880044&dis_dz=CT-GuangDong_GuangZhou&dis_st=42&src=iqiyi.com&uuid=7908d205-5b8bab6c-191&rn=1535880043829&qd_tm=1535880032356&qd_tvid=1294428000&qd_vipdyn=0&qd_k=a07fc56691bd8558b5e9f23e9119b36a&cross-domain=1&qd_aid=220327201&qd_uid=&qd_stert=0&qypid=1294428000_02020031010000000000&qd_p=7908d205&qd_src=01010031010000000000&qd_index=1&qd_vip=0&qyid=836c674bfa7e7387a323d314bfb4a875&pv=0.1&qd_vipres=0&range=8192-10431487
HTTP/1.1


发现这简直就是了好吧，第一层构造的请求头的所需的构造就在这里面（对比`图2.33`和第一层请求头）

所以我们需要知道得到这段数据的请求头，也就是构造如下请求头， 称为 ==**构造③**==

> GET /videos/v0/20180831/44/87/611f0244cfe1a64d165b2f630be2e9fc.f4v?qd_tvid=1294428000&qd_vipres=0&qd_index=1&qd_aid=220327201&qd_stert=0&qd_scc=ec9c19fe1c7863a46bd3245b238c2745&qd_sc=e1e1493d64de4db9c4d6c27f8220b24c&qd_p=7908d205&qd_k=a07fc56691bd8558b5e9f23e9119b36a&qd_src=01010031010000000000&qd_vipdyn=0&qd_uid=&qd_tm=1535880032356&qd_vip=0&cross-domain=1&qyid=836c674bfa7e7387a323d314bfb4a875&qypid=1294428000_02020031010000000000&qypid=1294428000_02020031010000000000&rn=1535880043829&pv=0.1&cross-domain=1
> HTTP/1.1 

我们又发现了，这个请求头跟第一层构造很相像，想必是孪生兄弟。

如下图
![图2.34](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-34.png)
***【图2.34】***

所以，事实上，如下面一段是第三层构造比第一层要多的数据
`&cross-domain=1&qyid=836c674bfa7e7387a323d314bfb4a875&qypid=1294428000_02020031010000000000&qypid=1294428000_02020031010000000000&rn=1535880043829&pv=0.1&cross-domain=1`

###### 结束第一层构造
***
#### 结束分析

### 简短说明

***接下来你可以一个参数一个参数的找依据，但是有时候这是不必要的，你可以通过在不同的爱奇艺视频网页请求抓包，把一些不变的量你就当然是常量就行了。当然这也有弊端：比如说这样你就无法了解更深一层的含义。或者这些参数有可能就是破解vip限制的关键。（讲道理这些参数是什么意义我就没研究，所以这个破解vip我是随便说的，就是为了表达这样一个意思，就是这些参数对仅仅解析这些视频可能是无关重要的，但是如果你不仅仅只要这些，这些就很重要了）。***

接下来的几个参数我不需要讲，因为这个前面的一些参数值有重复。

所以后面就大概简单的讲一下如何验证你构造出来的结果对不对。 你需要如下软件

下面软件可以帮助你运行js脚本，这样你就能运行上面的函数得到加密结果。
![图2.35](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-35.png)
***【图2.35】***

然后你可以使用postman可以帮助你提交请求数据
![图2.36](https://raw.githubusercontent.com/ZSAIm/ZSAIm.github.io/master/image/2019-05-09/2-36.png)
***【图2.36】***

至于这些软件如何使用我就不讲了。（如有需要自己可以去网上查教程资料什么的）



### 项目实现

> ***https://github.com/ZSAIm/iqiyi-parser/blob/master/core/iqiyi.py***


***

# 参考和说明

- 本文首次是在吾爱论坛发布： [***点这***](https://www.52pojie.cn/thread-792958-1-1.html)

- CSDN博客文章：[***点这***](https://blog.csdn.net/qq405935987/article/details/83789697)

- Github项目链接: [**点这**](https://github.com/ZSAIm/iqiyi-parser)

- 本文章仅用于技术交流。





