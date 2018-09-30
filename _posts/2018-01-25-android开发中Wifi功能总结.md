---
layout:     post
title:      android开发中Wifi功能总结
subtitle:   深入总结wifi开发的相关知识
date:       2018-01-25
author:     yourzeromax
header-img: img/post-bg-github-cup.jpg
catalog: true
tags:
    - Android
---

# 写在前面

距离上次写博客还是在几个月以前了，最近几个月实在是太忙了，跑去做课设和ios开发，短暂地告别了Android，直到最近在某院实习做毕业设计时需要用到Android系统进行室内定位开发，本文主要是将在项目开发的过程中遇到的关于wifi开发的问题记录下来，方便大家整理。
#情景提要
Android开发过程中，应该说Wifi是很重要的一个功能，在产品中，可能需要随时监听网络状况的变化等等，最近的项目是做室内定位，需要采集各种Wifi的信号和其距离，算出一个大概的范围再结合其他的技术手段进行信息融合得到精确的定位数据，其实个人对wifi一次定位是不抱太大希望的，因为wifi信号的衰减和硬件、环境、空间等等随机因素有关，是及其不准确的，但是项目既然有这个内容，也应当仔细地去研究研究，之前看过网上很多资料，发现大家只是单纯地谈API，对新手很不友好，因此我也想换一个角度来描述一下android Wifi开发，首先我总结以下几个问题：

- **wifi的权限管理**
- **如何开关设备的wifi功能**
- **监听设备wifi状态的改变**
- **获取扫描wifi结果**
- **如何连接/断开一个wifi**
- ***在项目中的特殊用法***

我个人认为解决以上几点问题就足够了，当然，最后一个在项目中的特殊用法是一个相对“反人类“的用法，感兴趣的可以看看。

-------------------

## wifi的权限管理

```
 <uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
 <uses-permission android:name="android.permission.CHANGE_WIFI_STATE"/>
```
非隐私权限，因此不用考虑6.0的动态权限管理，直接申明就好，当然这里有一个坑，也是很多人不理解容易遗漏的：获取wifi需要定位权限。为什么需要定位权限呢？其实很好理解的，wifi其实也是一种定位手段，大家可能有用假药或者高德地图，wifi开启能够提高定位的准确性，具体细节我觉得大家不用深究，毕竟不是做通信的，只需要知道下面的重点就好：
```
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
```
动态权限申请过程，略。

## 基础说明

在此说明，面向对象的五（六）大基本原则（S.O.L.I.D）第一条就是单一职责，因此最好是在代码中wifi功能体现到一个类之中并且使用单例模式（饿汉）获取它的实例，比如我就用的MyWifiManager类。然后，大家先看一下我认为在wifi功能开发中最为重要的（3+1）个类： 



WIFI开发所关心的三个类和功能描述：

类名     | 功能
-------- | ---
WifiManager | wifi统一管理类，进行各种wifi操作
WifiInfo    | 描述当前连接的wifi热点信息
WifiConfiguration    | wifi网络配置信息

除此之外还有一个类：

类名     | 功能
-------- | ---
ScanResult | 描述扫描出的wifi热点的信息

以上的（3+1）个类是关乎各种wifi操作的最为重要的类，还有以下四个专业术语再帮助大家巩固以下：
名称     | 功能
-------- | ---
SSID | 描述wifi热点的名称，就是大家搜索到的直接名称，如ChinaNet
BSSID    | 姑且理解成热点的mac地址，但实际有所不同
networkID    | 数字型的id
RSSI|描述wifi信号强弱的值，官方叫做level


这里说一个很重要的知识点！！！可以脑补一下手机输入wifi密码连上后，下一次可以不用输入密码甚至是自动连接，就是因为手机中其实已经存储了wifi的信息，而这一个个wifi信息可以看作是一个以networkid为标识存储在手机上的队列（之后会介绍）。当然，手机查看wifi的具体信息（如wifi密码）是需要root权限的，大家就别瞎搞了（小米手机有分享密码二维码的功能）。

## 如何开关设备的wifi功能

首先获得WifiManager的实例对象：

```
WifiManager mWifiManager= (WifiManager) context.getApplicationContext().getSystemService(Context.WIFI_SERVICE);
```
然后再直接调用就行了：
```
boolean isOpen=mWifiManager.setWifiEnabled(true);
```
传入的参数是你想开启的状态，返回值是返回你操作是否执行，不代表wifi状态的变化（切记！）虽然代码简单，但是wifi的开启关闭过程是有延时性的，因此需要进一步对wifi变化过程进行适当判断，确保你之后的功能代码是在完全开启/关闭过后执行，也就是下面部分的内容。
## 监听设备wifi状态的改变
wifi状态的改变是会导致广播事件的发生，但是实际上对于上部分的问题没有必要用广播，因此这部分我先介绍代码判断，再介绍广播事件的使用方法。
###代码监听连接状态：
WifiManager之中有当前状态的enum类型，可以看下表：
名称     | 值 |描述
-------- | ---
WIFI_STATE_DISABLING  | 0|wifi正在关闭
WIFI_STATE_DISABLED    | 1|wifi关闭
WIFI_STATE_ENABLING    | 2|wifi正在开启
WIFI_STATE_ENABLED    | 3|wifi开启
WIFI_STATE_UNKNOWN    | 4|wifi未知
在一般的状态监听，可以直接通过WifiManager对象来获取,再进行相应的判断就可以满足，看下面的代码：

```
mWifiManager.setWifiEnabled(true);
        if (mWifiManager.getWifiState() == WifiManager.WIFI_STATE_DISABLING) {
            //do someThing;
        } else if (mWifiManager.getWifiState() == WifiManager.WIFI_STATE_DISABLED) {
            //do someThing;
        } else if (mWifiManager.getWifiState() == WifiManager.WIFI_STATE_ENABLING) {
            //do someThing;
        } else if (mWifiManager.getWifiState() == WifiManager.WIFI_STATE_ENABLED) {
            //do someThing;
        } else if (mWifiManager.getWifiState() == WifiManager.WIFI_STATE_UNKNOWN) {
            //do someThing;
        }
```
### 使用广播事件监听wifi状态的改变
wifi状态的变化会发出下列的广播事件：
名称     | 描述
-------- | ---
WifiManager.WIFI_STATE_CHANGED_ACTION  | wifi开关变化通知
WifiManager.SCAN_RESULTS_AVAILABLE_ACTION    | wifi扫描结果通知
WifiManager.SUPPLICANT_STATE_CHANGED_ACTION    | wifi连接结果通知
WifiManager.NETWORK_STATE_CHANGED_ACTION    | 网络状态变化通知
采用广播事件来进行监听的话，就可以做很多事情，可以通过查看上表描述得知各个广播事件的作用，这里的话，还是以wifi开关变化通知广播为例进行说明，广播是自动发出的，因此，我们只需要注册相应的BroadCastReceiver就可以了，具体看下文的代码：

```
    class WifiBroadCastReceiver extends BroadcastReceiver {
        @Override
        public void onReceive(Context context, Intent intent) {
            switch (intent.getAction()) {
                case WifiManager.WIFI_STATE_CHANGED_ACTION:
                    int wifiState = intent.getIntExtra(WifiManager.EXTRA_WIFI_STATE,
                            WifiManager.WIFI_STATE_DISABLED);
                    switch (wifiState) {
                        case WifiManager.WIFI_STATE_DISABLED:
                            doSomething();
                            break;
                        case WifiManager.WIFI_STATE_ENABLED:
                            doSomething();
                            break;
                    }
                    break;
                case WifiManager.SCAN_RESULTS_AVAILABLE_ACTION:
                    break;
            }
        }
    }
```
当然，别忘了创建自定义Receiver对象并注册到context中，略。




## 获取扫描wifi结果

>在我实际测试的时候，wifi扫描的速度是很快的，通常Android应用程序是肯定不会像我一样这么频繁的扫描，同时，wifi扫描是非常耗电的，因此在普通的应用开发时，一定要慎重地使用WiFi扫描，并且尽可能在扫描获取到自己有用的信息后及时关闭，当然，如果跟我一样有特殊的用途，当我没说咯。

这一小节会主要介绍如何运用设备的wifi扫描，并且简要地介绍获取这些wifi的一些基础信息，比如SSID、RSSI等，除了这些以外，还会介绍一些我碰到的坑。

wifi扫描运用到的类就是ScanResult类，每一个被扫描到的wifi的信息几乎都会存在这个类的对象之中，换言之，我们获取一些基本信息就是要通过ScanResult对象：

```
 mWifiManager.startScan();
List<ScanResult> Results = mWifiManager.getScanResults();
 for (ScanResult Result : Results) {
       Log.d(TAG, "wifi结果: " + Result.SSID + "  RSSI:" + Result.level + "  Time：" + System.currentTimeMillis());
        }
```
一句简单的代码就能开启扫描，第一行代码是让设备开始扫描第二行代码是获取扫描结果，我建议大家最好是在广播通知中进行结果处理，它返回的是一个List列表，之后的话，可以遍历List中的对象读取相关的信息，具体的信息获取方法也是一目了然，不再赘述。

>提醒（坑）：getScanResults（）返回的队列是累计的！这句话大家好好理解。

## 如何连接/断开一个wifi

 如何获取已经在设备上配置过的wifi：
 
```
List<WifiConfiguration> configurations = mWifiManager.getConfiguredNetworks();
```
返回的仍然是一个List列表，存储的对象是WifiConfiguration，在开始就描述过这个类，它所存储的就是设备之前连接过的所有wifi热点的信息，同样的，可以通过遍历它的实例来获取wifi热点的信息，其中SSID和networkId一定是存在的，BSSID，password这些是null的，这点一定要注意，具体代码就不贴了。

如何获取当前wifi的连接信息：

```
WifiInfo info = mWifiManager.getConnectionInfo();
```
几乎和WifiConfiguration一样的，但是如果当前没有连接wifi的话，就会返回Null，它包括了SSID、networkId、BSSID的，切记一个问题：它的SSID是带双引号的，这点和ScanResult对象不一样哟！

### 连接/断开热点
步骤：

 1. 创建一个包含SSID、password等信息的自定义类封装
 2. 创建wifiConfigruation对象，并获取到networkId
 3. 用WifiManager同一管理，开始连接
 4. 断开热点

这里我就偷一个懒了，直接摘选网上的代码了，因为我觉得这代码的作者已经写的很好了：

```
public WifiConfiguration createConfiguration(AccessPoint ap) {
        String SSID = ap.getSsid();
        WifiConfiguration config = new WifiConfiguration();
        config.SSID = "\"" + SSID + "\"";

        String encryptionType = ap.getEncryptionType();
        String password = ap.getPassword();
        if (encryptionType.contains("wep")) {
            /**
             * special handling according to password length is a must for wep
             */
            int i = password.length();
            if (((i == 10 || (i == 26) || (i == 58))) && (password.matches("[0-9A-Fa-f]*"))) {
                config.wepKeys[0] = password;
            } else {
                config.wepKeys[0] = "\"" + password + "\"";
            }
            config.allowedAuthAlgorithms
                    .set(WifiConfiguration.AuthAlgorithm.SHARED);
            config.allowedAuthAlgorithms.set(WifiConfiguration.AuthAlgorithm.OPEN);
            config.allowedKeyManagement.set(WifiConfiguration.KeyMgmt.NONE);
            config.wepTxKeyIndex = 0;
        } else if (encryptionType.contains("wpa")) {
            config.preSharedKey = "\"" + password + "\"";
            config.allowedKeyManagement.set(WifiConfiguration.KeyMgmt.WPA_PSK);
        } else {
            config.allowedKeyManagement.set(WifiConfiguration.KeyMgmt.NONE);
        }
        return config;
    }
```
然后就是通过上述方法创建一个WifiConfigruation对象并获取到它的networkId：

```
 WifiConfiguration config = createConfiguration(ap);
 //如果你设置的wifi是设备已经存储过的，那么这个networkId会返回小于0的值。
 int networkId = networkId = wifiManager.addNetwork(config);
```
再然后就是连接了：

```
wifiManager.enableNetwork(networkId, true)
```
如果要连接已经存储过的wifi，就可以直接获取WifiConfigruation对象中的networkId进行连接，networkId在一台设备之中，是唯一的。

当然，不管你输入的密码正确或错误，该wifi连上或者未连上，都会发送广播事件进行通知，也可以在广播事件中进行相应的处理，忘记了广播类型的同学，可以翻看前文。

最后就是断开连接，很简单的一句：

```
mWifiManager.disconnect();
```
至此，wifi的基础讲述完毕。

## 在项目中的特殊用法
wifi定位采用的是电磁指纹的方式，也就是说不停地采集、扫描周围wifi的信息，再不停地获取它们的RSSI，这个RSSI值基本符合距离远近，但是还有很大的优化空间，不能直接用作定位，技术原因我在最开始已经说明，但还有个原因就是，我没钱买工业级大功率路由器群，家用级的wifi信号顶多20m，人眼就能看见，我还定个P的位。。。

我在采用这种策略的时候，遇到最大的问题就是扫描频率问题，我们的项目实际要求至少能够达到10hz频率，虽然wifi扫描速率是毫秒级的，但是我在测试的时候发现，wifi扫描的确是毫秒级，然而RSSI变化，在有的android设备中15s才更新一次，有的设备12s更新一次，并且偶尔会频繁变更，这个频率太规律了，所以我怀疑应该有地方是我没有注意到的，如果各位读者有兴趣或者有经验的话，恳请能够给予点拨或者与我讨论，感激不尽！


----------


# 后记
前段时间有幸能在google中国开发者的Android Things沙龙中分享自己的项目经验，在会上也谈到了我的项目未来的展望和奋斗方向，之后的话，也想陆续地分享和项目有关的知识，包括蓝牙啊，惯导、传感器开发等等，享受写博客的时光，因为总能使我保持一颗对知识敬畏的心。
