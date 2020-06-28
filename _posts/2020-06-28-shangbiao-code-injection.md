# 中国商标网JS调试 - 动态代码注入

## 前言

1. 中国商标网地址：http://wcjs.sbj.cnipa.gov.cn/txnT01.do
2. 本文的主要目的并不是对中国商标网的爬虫实现，而是对其反爬机制的核心突破。由于反爬策略的更新频繁，不保证本文中的代码的时效性。
3. 本文不会对JS脚本从头开始逆向分析，而是有选择性的挑选一些有趣反爬机制进行描述其解决思路。
4. 本文主要是记录中国商标网的调试过程，不会提供任何关于爬虫实现的代码。__！！！！！！！！！别问我要商标网的爬虫代码!!!!!!!，因为我也没有。__

### 背景

1. ``中国商标网``在反爬安全上是属于标杆的存在。其反爬技术提供者是``瑞数信息``，反爬机制是属于``动态安全``。
2. 对于动态反爬机制最核心的地方不是token加密算法，而是其反调试策略。
3. 任何动态的事物都是基于静态产生的，所以只要找到静态点，那么动态就会坍缩为静态。

### 工具

- Fiddler: 尽可能使用新版本，如果必要，至少要带Fiddler Script功能。
- 谷歌浏览器：字面意思，也是尽可能使用新版本。

### 知识点

- JavaScript
- Fiddler Script
- 谷歌浏览器

## 正文

在背景里面我有说过中国商标网的反爬机制是动态反爬，那么根据定理：

> 必须用魔法打败魔法。

我们必须要用动态打破动态。

本文章的核心思路是代码注入。代码注入是字面意思，其作用就是在脚本执行之前插入自己的代码，以实现对代码的查看修改替换等各种操作。通过 `Fiddler` 可以实现代码注入，下面会对其实现原理进行简单的介绍。如对该知识点了解请跳过。

### 了解 Fiddler Script

众所周知``Fiddler``是一个很强大的抓包工具，但是大多数人对它的印象主要是抓包，但其实其最强大的功能实属``FiddlerScript``，下面引用``《Fiddler调试权威指南 - Debugging with Fiddler》``对``FiddlerScript``的描述。

> ``Fiddler``在处理每个``Session``时，脚本文件``CostomRules.js``中的方法都会运行，该脚本使得你可以隐藏、表示或任意修改复杂的``Session``。规则脚本在运行状态下就可以修改并重新编译，不需要重新启动``Fiddler``。

上面一句话简单的来说就是``Fiddler``作为一个``中间者（代理服务器）``，其``FiddlerScript``可以实现在客户端与服务器之间的数据请求转发过程中进行修改。当然作为规则脚本还可以对``Fiddler``的业务功能进行拓展。

所以，根据上面所说，我们就可以理解代码注入就是在``服务器``返回请求响应之后，``Fiddler``作为中间者根据预先编写的``FiddlerScript``来将代码注入到响应中，之后再将响应转发给 ``客户端(本文所说的客户端主要是浏览器)``。以上的描述的过程体现在下面的流程图中。

![1-0](http://baidu.com)

#### Session 处理函数

> 编译``FiddlerScript``时，``Fiddler``保留了某些关键静态函数的引用，这些函数都能在``Handlers``类中找到。

除了``OnReturningError``外，以下的处理函数``Handlers``按照了其执行顺序进行描述：

- __OnPeekAtRequestHeaders__: 在客户端发送请求头``(Request Header)``之后，中间者接收到其请求头后进入该处理函数。
- __OnBeforeRequest__: 在客户端发送请求体``(Request Body)``之后，中间者接收到请求体后进入该处理函数。之后将新的请求头和请求体转发给服务器。
- __OnPeekAtResponseHeaders__: 在服务器返回响应头``(Response Headers)``之后，中间者接收到响应头后进入该处理函数，之后将请求头转发给客户端。
- __OnBeforeResponse__: 在服务器返回响应体``(Response Body)``之后，中间者接收到响应体后进入该处理函数，之后将请求体转发给客户端。
- OnReturningError: 在``Fiddler``生成的错误信息（如``“DNS Lookup Failed”``）返回给客户端时被调用。通过这个函数可以定制客户端应用看到的错误信息。

下图描述了``Session``处理函数的流程

![1-1](http://baidu.com)

在本文中，代码注入是在``OnBeforeResponse``处理函数中进行。

为了加深对代码注入方法的理解，我们不妨拿中国商标网来练练手。

### 反调试策略

浏览器打开商标网，刚打开开发者工具（F12），发现调试工具触发了中断，无一例外都停在了debugger处。

>  debugger指令是调试器断点，在未进入调试器的情况下，JS引擎会忽略任何debugger指令。

有很多人其实看到这种情况就已经束手无策，这反调试机制是建立在动态脚本的前提下的，这在传统方法来看确实是没有任何招架之力。如非要逆向这些脚本，就必须要硬啃代码。

![1-2](/image/2020-06-28/1-2.png)

#### 问题分析

事实上，存在两处``debugger``指令，一处是鼠标事件``(MouseEvent)``触发的，另一处是``定时器(setInterval)``触发的，在``debugger``指令的前后进行了时间差``new Date().getTime()``方式判断，通常只要时间差大于几百毫秒或者更低时间差内就会触发反调试防御，在这几百毫秒内一般来说无法当然只要你不继续执行代码，就不会被检测到。

既然这是为调试器准备的反调试策略，那么我们直接把他删掉或者注释掉就能避免调试器中断。透过``Fiddler``这个中间者，我们就能动态注入代码。

有一个问题是，要注入什么地方，要修改什么地方？没错，只要删掉``debugger``指令就能卸除``debugger``反调试防御。但是``debugger``指令在哪里？

我们可以注意到上图中的标签页的名称是``VM+数字``，这种组合的名称通常这是为了区别原网页的JS脚本，是由``eval()``方法产生的或者是``ajax``方法获取的。商标网的``debugger``所处的脚本是通过``eval()``动态执行的脚本，在本文我们不讨论如何一步一步找``eval()``执行的位置[<sup>1</sup>](#补充说明)。这里直接说结果，``eval()``在下图中的下红框中执行。

![1-3](/image/2020-06-28/1-3.png)

#### 解决思路

__如果``debugger``是在原网页的JS脚本中，即使是动态脚本，也可以直接修改脚本代码以删除``debugger``。但是这里经过了在脚本中``eval(string)``来代理执行的代码，``debugger``存在于``string``中，所以只能通过将代码注入到``string``才能修改删除``debugger``。所以我们需要在脚本中的``eval(string)``的执行之前通过注入修改``string``的代码来达到修改删除``debugger``，之后执行的``eval(new_string)``就是经过了去``debugger``的代码，这样反调试防御就能卸下来了。__

这里面的一个技巧是，在``eval(string)``执行的刚好之前，``string``必然是已经经过了解密了的代码字符串，这样在这恰好之前注入代码可以直接跳过脚本解密这个过程，完全可以不用在意解密算法。

如下代码，既然这是动态的脚本，那么怎么定位呢？一个最简单的办法其实就是找特征信息，然后使用正则匹配就行了。

```javascript
... 
} else if (_$71 < 75) {	// 注意，这里的75和除了ret外的所有变量名也是动态生成的，不应作为特性值
     ret = _$Az.call(_$j1, _$8t);
 } else {
     ...
 } ...
```

既然有这么多特征值，那么__``/ret\s*=\s*[\w\$]+\.call\([\w\$]+,\s*([\w\$]+)\)/``__，这一正则就能定位。

#### 注入代码

既然是通过``Fiddler``来实现代码注入，我们肯定要先搞清楚如何进行编写``FiddlerScript``来实现代码注入，关于``FiddlerScript``的介绍可以参考书《Fiddler调试权威指南 - Debugging with Fiddler》和[ModifyRequestOrResponse](https://docs.telerik.com/fiddler/KnowledgeBase/FiddlerScript/ModifyRequestOrResponse)。

> 通过 ``Rule - Customize Rules`` 进入Fiddler脚本编辑器 ``Fiddler ScriptEditor``

``FiddlerScript``的脚本编写语言是``JScript.NET``，这是微软自己开发的``IE``脚本执行引擎，也是按照``ECMAScript``标准的，所以一般我直接把它当``JavaScript``来用，当然其中存在一定的使用差异。

![1-4](/image/2020-06-28/1-4.png)

好了，我们进入``Fiddler ScriptEditor``，找到``OnBeforeResponse``处理函数。加入如下代码。

```javascript
static function OnBeforeResponse(oSession: Session) {
    if (m_Hide304s && oSession.responseCode == 304) {
        oSession["ui-hide"] = "true";
    }

    // 以下为商标网的代码注入
    // 注意的是，对Fiddler来说，所有经过Fiddler的请求都会经过该处理函数，这就要求你自己对请求进行筛选，否则将会对所有请求都执行对应的操作。我这里仅仅是对域名为 "wcjs.sbj.cnipa.gov.cn" 和其请求头中"Content-Type" 包含 "html"的请求进行响应的处理。关于oSession对象，可以参见参考处的链接或书籍。
    if (oSession.HostnameIs("wcjs.sbj.cnipa.gov.cn") && oSession.oResponse.headers.ExistsAndContains("Content-Type", "html")){
        // 这一步我们将响应将Body作为字符串解析。
        var oBody = oSession.GetResponseBodyAsString();
        var newBody = oBody;
        var oRegEx = /\bret\s*=\s*[\w\$]+\.call\([\w\$]+,\s*([\w\$]+)\)/;
        var oEvalLineRes = oBody.match(oRegEx);
        if(oEvalLineRes !== null){
            var oLine = oEvalLineRes[0];
            var oCodeVar = oEvalLineRes[1];

            // 创建被注入的代码，通过修改脚本代码string来实现取出debugger指令。
            var rmDbg1Code = "var rmDbg1Res = " + oCodeVar + ".replace(/\\bdebugger\\s*;/, '') ;";
            var rmDbg2Code = "var dbg2RegEx = /\\{\\s*\\bvar\\s*([\\w\\$]+)\\s*=\\s*[\\w\\$]+\\[[\\w\\$]+\\[\\d+\\]\\]\\([\\w\\$]+\\(.*?\\)\\);\\s*\\}/;" + 
                "var dbg2assignVarName = rmDbg1Res.match(dbg2RegEx)[1];" + 
                "var rmDbg2Res = rmDbg1Res.replace(dbg2RegEx, '{var ' + dbg2assignVarName + '=false;}');";
            // 替换成经过处理后的代码所在的变量
            var newEvalCode = oLine.replace(oCodeVar, "rmDbg2Res");

            // 注入代码到脚本。
            newBody = newBody.replace(oLine, 
                                      rmDbg1Code + 
                                      rmDbg2Code + 
                                      newEvalCode);
        }

        // 替换响应体
        oSession.utilSetResponseBody(newBody);
    }
}

```

将以上代码添加完成后，保存即可生效。这时再尝试在商标网内打开F12就会发现不再发生中断，因为``debugger``已被移除。如下图就是注入代码后的新响应。

![1-5](/image/2020-06-28/1-5.png)

到目前为止，我们无法被``debugger``指令所限制，但是我们真的就能随心所欲的进行调试了吗？不是的，还不能，前面说了这是动态生成的脚本[<sup>2</sup>](#补充说明)，我们无法由谷歌浏览器的开发工具帮我们记录断点位置，这样我们就不能反复的进行调试。

既然调试器无法帮助我们准确的记录断点位置，我们只能另寻他法。既然反调试防御是通过``debugger``指令实现的，我们要以牙还牙，通过正则表达式定位并注入``debugger``进行准确的动态断点调试。

为了让文章尽量简洁易懂，这里不通过举大量例子来演示，不然容易让人产生困惑而且写起来累人。。只要你能领悟上面的代码注入示例，后续注入``debugger``不会是问题，所以这里不再进行演示。

在本文中我们都是通过正则表达式进行定位，如果有必要，可以通过构建RPC调用任何你想要实现的第三方程序处理，进行语义树解析定位什么的。

事实上，掌握以上的方法后，只要你够__肝__，你完全可以逆向整个提交流程。

如果是通过``Selenium``实现可能会更简单点，可以直接注入代码，将``Selenium``等自动化测试软件的判断屏蔽掉，什么无头检测之类的，还有鼠标事件动态产生的``Cookie``算法。如果不打算进行模拟鼠标则需要逆向其算法，所以工作量也不会少太多。

#### 关于 7cLOtPi5wrHA.5780574.js

```javascript
(function() {
    var _$l5 = 0
      , _$Q1 = $_ts.scj
      , _$_d = $_ts.aebi;
    // 下面这类函数属于事件或定时器触发函数
    function _$WM() {
        var _$UC = [459];
        Array.prototype.push.apply(_$UC, arguments);
        return _$Tb.apply(this, _$UC);
    }
    // ...忽略多个同类... //
    
    var _$Jv = []
      , _$X9 = String.fromCharCode;
    
    // 下列函数主要用于生成对象的混淆属性名数组，每一个元素对应一个属性。
    _$30('rtrc0ccavodcr`ch}r`.`uars`,`}a|c|ch}r`b.........'); 
    
    // 下列一堆变量经过二次代理内置函数/方法，也属于混淆的一种。
    var _$Sp, _$0x = null;
    var _$uR = window
      , _$8J = String;
    var _$Xc = _$uR["XMLHttpRequest"];
    var _$1q = _$uR["ActiveXObject"];
    // ...忽略多个同类... //
    
    // 以下是简单的通过简单的浏览器端判断等操作，同时启动了准备了一个反调试判断的时间戳
    _$iU();
  
    // ...
    
    // 由下面该字符串通过charCodeAt()进行多种操作后生成的数组，将参与后续很多算法运算。 
    var _$2H = _$ZH["call"]("qrcklmDoExthWJiHAp1sVYKU3RFMQw8IGfPO92bvLNj.7zXBaSnu0TC6gy_4Ze5d{}|~ !#$%()*+,-;=?@[]^", '');
    _$S3();
    
    // 以下调用将<meta>元素的content进行解密，并且将meta节点删掉，这一节点的内容将影响后续生成的input#__onload__节点，并用于生成/?kmcmNx0Q=...
    _$Vt(_$9k());
   
    
    _$Tb(768, \d); // Cookie 加密处理
    // 后面的太多了，不分析了
```

注意：以上代码我注入了属性名反混淆的代码，所以显示的是原属性名。其实还可以将内置函数/方法进行复原对象，可读性更高。

##### 如何调试``while(1)``内的执行流程？

对于上面这些流程走向明显的都很好分析，但是对于``while(1)``或者``for(;;)``没有调用栈记录的，我们该怎么办？怎么去分析调试呢？相信看过 [爱奇艺视频cmd5x解析算法的移植分析和实现.2019-08](#文章)的应该知道，当时我在调试``for(;;)``过程中使用的是对做决策的变量进行记录，也就是说，通过记录做决策的变量的走向来判断其执行流程。那么对于现在这个情况，我们需要通过注入代码到``while(1)``内的决策变量来进行分析流程。

# 最后

没了

# 补充说明

[1] ``eval(string)``方法中，大家可以想一下``string``从何而来？可以发现的是网页就引入的JS脚本不多，其中一个脚本比较可疑 - ``7cLOtPi5wrHA.5780574.js``中的``$_ts['5780574']``变量，可以看到该变量的字符串内容明显存在不合语法的内容，有理由猜测是首先经过一层解密后再进行的``eval()``，如果你不是瞎调的一派，其实可以尝试下写入破坏性数据进入该字符串，然后在``eval()``过程中就会发现语法错误并抛出异常就能发现是哪一步执行的。事实上这里在调用栈图1的调用栈就能追溯到``eval()``的位置，看调用栈所对应的文件就能找到。

[2] 这里所说的动态生成的脚本事实上是这样的，``7cLOtPi5wrHA.5780574.js``脚本是静态的这毫无疑问。为什么最后说是动态脚本呢？因为真正的动态只有原网页``txnT01.do``的源代码，其中包括``<meta>``再内的content，里面的内容很奇怪对吧？是的，它经过解密后用于生成临时表单等东西，它也是动态生成的。第二行的5维数组的变量包括下面的``if`` ``else``决策树等也是动态产生的，这里面的数组里面的数值决定着后续的执行顺序，这非常重要，那怎么办？到底静态点在哪里？发现第二级数组的长度是不变的吗？是的，数组的值是动态的，但位置的作用其实是固定的。比如变量的\[1\]\[14\]位置开始就是用来链接一个动态生成的字符串，并且由该字符串构建静态脚本``7cLOtPi5wrHA.5780574.js``的变量名，这样，一个静态脚本由动态脚本转成了动态。

# 参考

[1] Fiddler调试权威指南(Debugging with Fiddler), Eric Lawrence, 祝洪凯 / 李妹芳 

[2] FiddlerScript CookBook, [http://www.fiddlerbook.com/Fiddler/dev/ScriptSamples.asp](http://www.fiddlerbook.com/Fiddler/dev/ScriptSamples.asp)

[3] ModifyRequestOrResponse, [https://docs.telerik.com/fiddler/KnowledgeBase/FiddlerScript/ModifyRequestOrResponse](https://docs.telerik.com/fiddler/KnowledgeBase/FiddlerScript/ModifyRequestOrResponse)

](https://github.com/zsaim)

# 相关

## 作者

【1】 [GitHub](https://github.com/zsaim)

【2】[GitHub Pages](https://zsaim.github.io/)

【3】 [CSDN](https://blog.csdn.net/qq405935987)

## 文章

【1】 [爱奇艺视频cmd5x解析算法的移植分析和实现.2019-08](https://zsaim.github.io/2019/08/23/Iqiyi-cmd5x-Analysis/)

【2】 [腾讯视频cKey9.1的生成分析和实现](https://zsaim.github.io/2019/05/06/Tencent-cKey9.1-Analysis/)

【3】[爱奇艺视频H5解析分析过程](https://zsaim.github.io/2018/09/07/Iqiyi-H5-Parse-Analysis/)











