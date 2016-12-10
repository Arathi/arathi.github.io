---
layout: post
title:  "关于舰C的Core.swf文件的解密与反编译"
date:   2016-12-10 20:58:00 +0800
categories: kancolle
tags: ActionScript
---

## 关于舰C的Core.swf文件的解密与反编译

个人感觉Core.swf就是舰C本地缓存中最神秘的一个文件，唯独这个文件在反编译器中打不开，里面也许有我想知道的所有的舰C客户端上的秘密。这次我想看看这个文件是怎么一回事。

首先用[FFDec](https://www.free-decompiler.com/flash/download/)（JPEXS Free Flash Decompiler，之前汉化一个叫《遗迹岛》的AIR游戏，因为没钱买硕思，所以找到这个开源货，想不到还挺好用的）打开mainD2.swf。里面有个mainD2类，明显就是整个程序的入口。

<!--more-->

```javascript
public function mainD2()
{
    super();
    _anim = new LoadingShipAnimation();
    _anim.x = 400;
    _anim.y = 240;
    addChild(_anim);
    _core = new Loader();
    addChild(_core);
    var _loc1_:Sprite = new Sprite();
    _loc1_.graphics.beginFill(0);
    _loc1_.graphics.drawRect(-800,-480,800,1440);
    _loc1_.graphics.drawRect(0,-480,800,480);
    _loc1_.graphics.drawRect(800,-480,800,1440);
    _loc1_.graphics.drawRect(0,480,800,480);
    _loc1_.graphics.endFill();
    addChild(_loc1_);
    addEventListener("addedToStage",_handleAddToStage);
}
```
在构造函数中，实例化了一个`LoadingShipAnimation`，大概就是播放loading动画小黑船。

在函数最后，添加了一个`addedToStage`事件的监听，回调函数是`_handleAddToStage()`。

```javascript
private function _handleAddToStage(param1:Event) : void
{
    event = param1;
    removeEventListener("addedToStage",_handleAddToStage);
    stage.addEventListener("contextMenu",function(param1:MouseEvent):void
    {
    });
    stage.stageFocusRect = false;
    var val:URLVariables = new URLVariables();
    val.version = "wodfmwagpixg";
    var req:URLRequest = new URLRequest("./Core.swf");
    req.method = "GET";
    req.data = val;
    var urlLoader:URLLoader = new URLLoader();
    urlLoader.dataFormat = "binary";
    urlLoader.addEventListener("complete",_handleLoadComplete);
    urlLoader.load(req);
}
```
进入`_handleAddToStage()`函数以后，首先删除了自身的事件监听，因此这个函数只会执行一次；
之后创建了HTTP请求相关的对象`URLVariables`、`URLRequest`，去请求获取"./Core.swf"，并创建了一个事件监听，在加载完成以后会回调`_handleLoadComplete()`函数。

```javascript
private function _handleLoadComplete(param1:Event) : void
{
    var _loc2_:URLLoader = URLLoader(param1.target);
    _loc2_.removeEventListener("complete",_handleLoadComplete);
    var _loc4_:ByteArray = _loc2_.data;
    var _loc3_:ByteArray = new ByteArray();
    var _loc6_:Object = _createKey();
    _core.contentLoaderInfo.addEventListener("complete",_handleLoadComplete2);
    _core.contentLoaderInfo.addEventListener("ioError",_handleLoadError);
    var _loc5_:LoaderContext = new LoaderContext();
    _loc5_.applicationDomain = ApplicationDomain.currentDomain;
    if(loaderInfo.loaderURL.match(/^file:/) == null)
    {
        ___(_loc4_,_loc3_,_loc6_);
        _core.loadBytes(_loc3_,_loc5_);
    }
    else
    {
        _core.loadBytes(_loc4_,_loc5_);
    }
}
```
进入到`_handleLoadComplete()`以后，跟`_handleAddToStage()`类似的，先删掉了自身的事件监听。

然后调用了`_createKey()`函数，生成key，目前不妨猜测是解密文件用的key。

接着又创建了一个`complete`事件的监听，回调函数为`_handleLoadComplete2()`。

之后检查url是否在本地（包含`file:`），对于在本地与不在本地的，有两种处理方式。不在本地时需要调用一个叫`___()`的函数，传入了三个参数分别是`_loc4_`（字节数组，响应报文内容）、`_loc3_`（一个空字节数组）、`_loc6_`（创建出来的key），然后再调用`_core.loadBytes()`，传入的参数是`_loc3_`（大概是上个函数的输出参数，应该存放了解密后文件内容）和`_loc5_`（LoaderContext对象）；至于另一种情况，直接调用`_core.loadBytes()`，传入的参数是`_loc4_`和`_loc5_`。两者的区别就是，是否经过了`___()`这个邪恶函数的加工。

可以猜测，以`file:`开头的，是为了娇喘程序员自己在本地开发调试时用着方便的，因此没有加密，直接加载；而实际使用时，URL不是本地的，所以都要使用`___()`来进行解密。

接下来看看`___()`的逻辑：

```javascript
private function ___(param1:ByteArray, param2:ByteArray, param3:Object) : void
{
    var _loc4_:* = this;
    param3[param3[/.$/(/..../(param3))](_loc4_,"")] = param3[/.$/(/ .../({}))](/./(/.. /({})),/.$/(/../({})),/./(/./([])),/./(/..$/(!{})),/.../(!!{}),/../(/.. /({})),/.$/(/../({})),/.$/(/../(!!{})));
    param3[~_loc4_] = ~_loc4_ >>> ~_loc4_;
    param3[param3[/./(/.....$/(param3))]("",param3)] = ""[param3[_loc4_]];
    param3[/./(param3)] = ~_loc4_ / ~_loc4_ << ~_loc4_ / ~_loc4_ << ~_loc4_ / ~_loc4_;
    var _loc5_:* = param3[/./(param3)] << ~_loc4_ / ~_loc4_ << ~_loc4_ / ~_loc4_ | ~_loc4_ / ~_loc4_;
    param3[/ ./((~~_loc4_)[param3[_loc4_]])] = _loc5_;
    param3[/./({})] = _loc5_;
    param3[/../([][{}])] = param3[/./(param3)][param3[/.$/(/..../({}))](/./(!!{}),/.$/(/../({})),/ (.*)]$/(param3[param3])[~_loc4_ / ~_loc4_])](~(~_loc4_ << (/./(/..$/(~_loc4_ << ~_loc4_)) | ~_loc4_ << ~_loc4_ >>> ~_loc4_)));
    param3[/.$/(!!{})] = param3[/.$/(/ .../({}))](/.$/(/./([])),/.$/(!{}),/./(/./([])),/./(/.....$/(/./[param3[_loc4_]])),/./(!!{}),param3[/../([][{}])]);
    param3[/./({})] = param3[/./(/.....$/({}))](/./(!{}),/.$/(/../(!!{})),/.$/(/../(param3)),/.$/(/ .../((~_loc4_)[param3[_loc4_]])),/.$/(/ ./([][param3[_loc4_]][param3[_loc4_]])),param3[/../([][{}])],/.$/(/../(!{})),/.$/(/../(!!{})),/.$/(/ ./(param3[param3[_loc4_]][param3[_loc4_]])),/.$/(/../(param3)),/.$/([][{}]),/.$/(!{}));
    param3[/./({})] = param3[param3][param3[/./(param3)]];
    param3[/..$/(___)] = param3[/.$/(/..../({}))](param3[param3](param3[_loc4_])[param3[/.$/(!!{})]],/./(/..$/(~_loc4_ >>> _loc4_)));
    param3[/..$/(___)] = param3[/./(param3[param3])](param3[/..$/(param3[/./(param3[param3])])]);
    param3[~~_loc4_] = param3[/.$/(/../(!{}))](param3[~_loc4_],param3[~_loc4_]);
    param3[/ ./({})] = param3[/./(/.....$/(param3))](param3[/./({})](param3[/ ./((~~_loc4_)[param3[_loc4_]])] * (param3[~_loc4_] << param3[~_loc4_] << param3[~_loc4_])),param3[/.$/(/..../(""[param3[_loc4_]]))](param3[~~_loc4_] | param3[~_loc4_],~_loc4_),/./("."),/./(/. /([][param3[_loc4_]][param3[_loc4_]])),param3[/..$/(param3[/./(param3[param3])])],/./(!{}));
    param3[{}] = param3[/.$/(/ .../({}))](param3[/..$/(___)],/.$/(/../(!!{})),/./(/... /(___)),/./(!!{}),/.$/(!{}),/.$/(/ ./(param2[param3[_loc4_]])),/./(/..$/([][param3[_loc4_]])),/./(!!{}),/.$/(!{}),/./(/. /([][param3[_loc4_]])));
    param3[/.$/(___)] = param3[~_loc4_] << (param3[/.$/(/../(!{}))]("."[param3[_loc4_]](!{})[param3[/.$/(!!{})]],param3[/./(!!{})](~_loc4_,"_"[param3[_loc4_]](!!{})[param3[/.$/(!!{})]])) | param3[~~_loc4_] | param3[~_loc4_] << param3[~~_loc4_]);
    param3[/./(/..$/(___))] = param3[~~~_loc4_] << (param3[~_loc4_] | param3[param3[/.$/(/../(!{}))](~_loc4_,param3[/./(!!{})](~_loc4_,~_loc4_))]);
    param3[/ ./([][param3[_loc4_]])] = param3[/./(/..$/(!{}))](param1[param3[/.$/(!!{})]],param3[/.$/(param3[/.$/(/../(!{}))])]);
    param3[/ ./([][param3[_loc4_]][param3[_loc4_]])] = param3[/.$/([][{}])](param3[/ ./([""][param3[_loc4_]])],param3[/./(/..$/(param3[/./(/..$/(!{}))]))]);
    param3[/./([][{}])] = param3[/.$/(___)];
    param3[/.$/[param3[_loc4_]]] = /./(/....$/(!{}));
    param3[/../((!{})[param3[_loc4_]])] = /./(!!{});
    param2[param3[param3]](param1,~~_loc4_,param3[/./([][{}])]);
    param2[param3[param3]](param1,param3[/./(/..$/(!!{}))],param3[/ ./([][param3[_loc4_]][param3[_loc4_]])]);
    param2[param3[param3]](param1,param3[param3[/_/[param3[_loc4_]]]](param3[/./(/..$/(!!{}))],param3[param3[/../([][param3[_loc4_]])]](param3[/ ./([][param3[_loc4_]][param3[_loc4_]])],param3[~_loc4_] << param3[~~_loc4_] | param3[~~_loc4_] | param3[~_loc4_])),param3[/ ./({}[param3[_loc4_]][param3[_loc4_]])]);
    param2[param3[param3]](param1,param3[param3[/../[param3[_loc4_]]]](param3[/./([][{}])],param3[param3[/../(""[param3[_loc4_]])]](param3[/ ./([][param3[_loc4_]][param3[_loc4_]])],param3[~~_loc4_])),param3[/ ./({}[param3[_loc4_]][param3[_loc4_]])]);
    param3[/..$/(/./([]))] = param3[/.$/(/..../(param3))](/.$/(/.../(""[param3[_loc4_]])),/.$/(/../({})),/.$/(/../(!{})),/.$/(/.../([][{}])),/.$/(!!{}),/.$/(/ ../([][param3[_loc4_]])),param3[/./({})](param3[param3[/./[param3[_loc4_]]]](param3[param3[/../(/./[param3[_loc4_]])]](param3[/ ./((~~_loc4_)[param3[_loc4_]])],""[param3[_loc4_]](!!{})[param3[/.$/(!!{})]]),""[param3[_loc4_]](!{})[param3[/.$/(!!{})]])),/./(/./([])),/./(!{}),/.$/(/../({})));
    _loc4_[param3[/..$/(/_/([]))]][param3[/.$/(/..../({}))](/./([][{}]),/./(/....$/([][param3[_loc4_]])),/./(/...$/(!{})))][param3[/.$/(/..../({}))](/..$/(!{}),/.$/(/../(!{})),/.$/(/../(!!{})),/.$/(/../([][param3[_loc4_]])),param3[/../([][{}])])](param3[/ ./(param3)]) > 0?param2[param3[param3]](param1,param3[param3[/./[param3[_loc4_]]]](param3[/./([][{}])],param3[param3[/../(""[param3[_loc4_]])]](param3[/ ./({}[param3[_loc4_]][param3[_loc4_]])],param3[~~_loc4_] << param3[~_loc4_] | param3[~_loc4_])),param3[/ ./({}[param3[_loc4_]][param3[_loc4_]])]):param2[param3[param3]](param1,param3[param3[/|/[param3[_loc4_]]]](0,param3[param3[/../((~_loc4_)[param3[_loc4_]])]](param3[/ ./([][param3[_loc4_]][param3[_loc4_]])],param3[""[param3[/.$/(!!{})]]] << param3[~_loc4_] | param3[~_loc4_])),param3[/ ./([][param3[_loc4_]][param3[_loc4_]])]);
    param2[param3[param3]](param1,param3[param3[/$/[param3[_loc4_]]]](param3[/./([][{}])],param3[param3[/../([][param3[_loc4_]])]](param3[/ ./([][param3[_loc4_]][param3[_loc4_]])],param3[/./(/..$/(___))] >> param3[~_loc4_])),param3[/ ./(""[param3[_loc4_]][param3[_loc4_]])]);
    param2[param3[param3]](param1,param3[param3[/$/[param3[_loc4_]]]](param3[/./(/..$/(!!{}))],param3[param3[/../([][param3[_loc4_]])]](param3[/ ./(""[param3[_loc4_]][param3[_loc4_]])],param3[~_loc4_] | param3[~~_loc4_])),param3[/ ./({}[param3[_loc4_]][param3[_loc4_]])]);
    param2[param3[param3]](param1,param3[param3[/..$/[param3[_loc4_]]]](param3[/.$/(/ ../((~_loc4_)[param3[_loc4_]]))],param3[param3[/../((~_loc4_)[param3[_loc4_]])]](param3[/ ./("."[param3[_loc4_]][param3[_loc4_]])],param3[/.$/(/ ../((~_loc4_)[param3[_loc4_]]))] >> (param3[~_loc4_] << param3[~~_loc4_] | param3[~_loc4_]) | param3[~~_loc4_])),param3[/ ./("|"[param3[_loc4_]][param3[_loc4_]])]);
    param2[param3[param3]](param1,param3[param3[/__/[param3[_loc4_]]]](param3[/./([][{}])],param3[param3[/../(""[param3[_loc4_]])]](param3[/ ./("."[param3[_loc4_]][param3[_loc4_]])],param3[~_loc4_])),param3[/ ./({}[param3[_loc4_]][param3[_loc4_]])]);
}
```
不管你们看了上面的代码有什么感受，反正我是“What the fuck”。经过混淆以后这段代码恶心的一逼，`param3`是之前认为是key，却出现了很多对`param3`的操作，还是先看看`_createKey()`的逻辑吧：

```javascript
private function _createKey() : Object
{
    var key:Object = {};
    key.a = function(param1:int, param2:int):int
    {
        return param1 + param2;
    };
    key.s = function(param1:int, param2:int):int
    {
        return param1 - param2;
    };
    key.t = function(param1:int, param2:int):int
    {
        return param1 * param2;
    };
    key.d = function(param1:int, param2:int):int
    {
        return param1 / param2;
    };
    key.m = function(param1:int, param2:int):int
    {
        return param1 % param2;
    };
    key.f = function(param1:Number):int
    {
        return Math.floor(param1);
    };
    key.r = function():int
    {
        return Math.random();
    };
    key.j = function(... rest):String
    {
        return rest.join("");
    };
    key.n = function():Number
    {
        return new Date().getTime();
    };
    return key;
}
```
所以其实这个`_createKey()`是生成了一个对象，对象中封装了一些加、减、乘、除、模、随机、字符串链接等方法。

解密算法既然看不懂就不看了，绕过吧，直接输出解密后的`Core.swf`吧。

本来想用FFDec的修改功能直接改，然而FFDec毕竟图样，一改就出错……还是用搞个Flash吧……

下了一个Flash CS6，使用FFDec把mainD2.swf反编译成fla文件，使用Flash CS6打开，随便改点代码，调试一下……呃，不行，说什么“不能访问本地资源”，好像是传说中的AS3以后的安全沙箱机制。

网上找了一下，有个简单的办法，就是在`C:\Windows\System32\Macromed\Flash\FlashPlayerTrust`目录下随便创建一个文本文件，写上可以受到信任的路径。这回不报沙箱错误了，打开后无限小黑船。

接着把

```javascript
if(loaderInfo.loaderURL.match(/^file:/) == null)
{
    ___(_loc4_,_loc3_,_loc6_);
    _core.loadBytes(_loc3_,_loc5_);
}
else
{
    _core.loadBytes(_loc4_,_loc5_);
}
```
替换成

```javascript
___(_loc4_,_loc3_,_loc6_);
var fs:FileReference = new FileReference();
fs.save(_loc3_, "Core2.swf");
_core.loadBytes(_loc3_,_loc5_);
```
执行，弹出一个对话框，保存，反编译……想得美，还是打不开。

还是接着看代码吧，刚才提到，`_core`加载完成以后会回调`_handleLoadComplete2()`，那么就在`_handleLoadComplete2()`做点手脚吧，于是给`_handleLoadComplete2()`第一行下个断点，然而调试一下却发现这个断点根本不触发……

查了一下AS3的文档，了解到[LoaderInfo](http://help.adobe.com/en_US/FlashPlatform/reference/actionscript/3/flash/display/LoaderInfo.html)有个bytes属性，里面装的就是要加载的内容；还有7个事件，其中`_handleLoadComplete`中监听了`complete`和`ioError`，两个在调试时都没有触发，于是我监听了所有的事件：

```javascript
_core.contentLoaderInfo.addEventListener("complete",_handleLoadComplete2);
_core.contentLoaderInfo.addEventListener("httpStatus",_handleLoadHttpStatus);
_core.contentLoaderInfo.addEventListener("init",_handleLoadInit);
_core.contentLoaderInfo.addEventListener("ioError",_handleLoadError);
_core.contentLoaderInfo.addEventListener("open",_handleLoadOpen);
_core.contentLoaderInfo.addEventListener("progress",_handleLoadProgress);
```
新增的5个回调函数都只输出事件名称并加上了断点。

调试后发现只有`_handleLoadProgress`触发了两次，一次是开始时，一次是加载完成，估计如果文件比较大，会触发很多次的。

那就在最后一次触发`_handleLoadProgress`时输出bytes吧：

```javascript
private function _handleLoadProgress(_event:Event) : void
{
    trace("progress");
    var progressEvent:ProgressEvent = ProgressEvent(_event);
    if (progressEvent.bytesLoaded >= progressEvent.bytesTotal)
    {
        var fileRef:FileReference = new FileReference();
        var loaderInfo:LoaderInfo = LoaderInfo(_event.target);
        fileRef.save(loaderInfo.bytes, "Core3.swf");
    }
}
```
鉴于Core.swf本身肯定也有要加载的东西，在此猜测，也许是Core.swf要加载的资源没加载成功Core.swf内部报错，而这个错误不受到mainD2.swf的控制，最终导致了无限小黑船却不报错。总之尽量模拟一下实际环境吧。

把输出的`mainD2.swf`放到岛风Go的缓存目录下替换掉原先的，然后用Flash Player打开，这次不一样了，会进入有进度条的Loading画面，虽然进度条满了以后猫了，不过生成出来的Core3.swf是好使的，在FFDec中打开成功。



