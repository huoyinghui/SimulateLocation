> 本文只使用有Mac电脑，用iPhone的骚年们。
> **本章是水文，利用Xcode模拟定位打卡很早就有教程了，这里干货只有一行，离开Xcode任然保证模拟定位不变。**

### 对象：钉钉等LBS应用

近两年很多企业和中小型公司都开始使用钉钉打卡签到。很多苦逼党因为坐公交晚了几分钟，被扣钱，晚了几分钟，全勤没了，所以这里我们可以缓解下代码狗的痛苦 —— 模拟定位（先打卡，再到公司）。已经会连Xcode模拟定位的可以忽略前面的部分内容，直接跳到最后。

公司设定打卡范围，100米，500米，1公里都可以，但是基于有模拟定位这个技术，钉钉在打卡选项里加了一项WiFi打卡，定位打卡和WiFi打开可以叠加存在以保证有人打卡作弊（后面讲解如何破解WiFi）。

### 开车

* 一台Mac （安装了Xcode）
* 一台iPhone（越狱不越狱无所谓）
* 一根数据线。

### 坐标系统

这里普及一下坐标系统：
目前我们经常接触的无非就是**原始坐标**，**火星坐标**，**二次加密坐标**。
* 原始坐标：手机上获取到的是原始的GPS坐标 —— **WGS-84**。
* 火星坐标：我大天朝自己加了飘逸搞的一套加密坐标，中国国测局（和GFW一样的傻屌组织）—— **GCJ-02**：**谷歌**、**高德**。
* 百度加密坐标：在火星坐标的基础上再次飘逸后的加密坐标 —— **BD-09**：**百度**。

> 在遥远的东方，有一个天朝。
天朝有一个测绘局，发明了一种把美国卫星的GPS的地球坐标，进行偏移的算法，计算后，得出了一个火星坐标。
为了让火星坐标能正确的显示，又给每部导航软件加入了这个算法，可以在大家的地图上还原位置。并且给每部导航收费。美其名国家安全。而且这个算法看上去很牛B的样子，还不可逆。
所以，只有这个国家的人都在用错误的坐标。正宗的掩耳盗铃。
民用卫星精度都已经让你出身冷汗了，何况军用卫星。打仗估值也不会用中国的电子地图吧。
只可惜各种LBS应用，都是个麻烦事哦。

还好黄天不负有心人，终于经过大家的模拟，计算，基本还原了[飘逸算法](https://github.com/googollee/eviltransform.git)。

### 选技师

坐标获取入口：
* [高德](http://lbs.amap.com/console/show/picker)
* [百度](http://api.map.baidu.com/lbsapi/getpoint/index.html)

首先，根据各自的喜好，选好你想要模拟的位置，这里以高德地图为例：

![高德地图](/PNG/0.png)

可以看到右边有显示坐标 ：
```
104.06521,30.589833
```

### 上钟

iPhone所需要的坐标是**WGS-84**，我们获取的是**GCJ-02**，这里我们利用最新[飘逸还原算法](https://github.com/googollee/eviltransform.git)来转换出你所需要的真实坐标。

我所在的定位**四川省成都市ACC中航城市广场**原始坐标为：
```
104.06263069384391,30.593234492328744
```

### 服务

这里我们需要新建一个**gpx**文件，包含坐标用于模拟定位。
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<gpx version="1.1"
    creator="GMapToGPX 6.4j - http://www.elsewhere.org/GMapToGPX/"
    xmlns="http://www.topografix.com/GPX/1/1"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.topografix.com/GPX/1/1 http://www.topografix.com/GPX/1/1/gpx.xsd">
    <wpt lat="30.593234492328744" lon="104.06263069384391">
    </wpt>
</gpx>
```
根据不同位置不同把转换得到的原始坐标对应到**lat**和**lon**里面即可。

> 记得先把手机定位打开。

真机运行一个新建的空的iOS项目，把上面我们新建的**gpx**文件拖到工程里，配置一下**Scheme**，然后真机运行即可。
![gpx](/PNG/1.png)
![Scheme](/PNG/2.png)

这个时候千万别点**Stop**，直接**Home**键后台，再打开带定位的应用看看你当前的位置，484超开心。
还原定位的方法，直接**Stop**即可。

### 下钟

大保健做完了，可能有的会所还会送你一点小礼品，你要逗得技师开心结束的时候还会送你点小惊喜。
> 辣么，惊喜来了：
> * **破解钉钉WiFi打卡：**把家里的WiFi名称改得和公司打卡的WiFi即可。据我测试，我们公司只配置校验了SSID，没有校验DHCP地址。
> * **随时随地打卡：**按照上面的步骤模拟定位完成之后，不要Stop，直接拔掉数据线（猜测是Xcode开发者模式开了个进程来模拟定位，如果Xcode上没有Stop，那这个进程就不会Kill掉）。
> * **WiFi破解弊端：**公司如果启动了DHCP校验，那就只能靠社工的方法搞到地址，也只能在家里打卡了。
> * **随地打卡弊端：**恢复方法只能重启手机才能还原定位，经测试，微信里地图无法使用，一片空白，所以这里阔以用不用的旧的测试机来专注打卡，我就只能帮你到这里了。

中午测试了下摩拜的红包车，如果我的手机定位没有改变，无论骑行多远，骑行距离都是0米，红包也只有一块（一中午四辆红包车均是如此）。看到这里应该不用我告诉你怎么撸红包了吧，撸红包的成本还是相对较高，必须随时背着电脑，如果公司没有开启WiFi打开，你又恰巧背了电脑，那就真的能随时随地打卡了。

**Demo：**[https://github.com/rockerhx/SimulateLocation](https://github.com/rockerhx/SimulateLocation)

#### 测试流程: 虚拟定位到 西安第一医院

#### 1.百度获取位置坐标
[百度坐标系统](http://api.map.baidu.com/lbsapi/getpoint/index.html)
![获取坐标](https://raw.githubusercontent.com/huoyinghui/SimulateLocation/master/PNG/01bd_gps.jpg)



#### 2.百度坐标转换为WGS坐标, 写入到项目gps文件
[在线坐标转换](https://tool.lu/coordinate/)

![百度坐标转换为ios的坐标](https://raw.githubusercontent.com/huoyinghui/SimulateLocation/master/PNG/02BD_2_WGS.jpg)


#### 3.手机百度地图定位

![手机百度定位验证](https://raw.githubusercontent.com/huoyinghui/SimulateLocation/master/PNG/03_test.jpg)