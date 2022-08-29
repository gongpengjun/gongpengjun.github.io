---
layout: post
title: IM008 - IM兼容性基础 (第二版)
date: 2022-08-27 09:00:00
categories: IM707
---

App随着时间推移，会发布不同版本，但不是每个App用户都会升级到最新版本，服务端只有最新版，所以需要做兼容处理。

## 1、一个App时怎么办？

首先想到的就是直接使用App版本号判断新老版本并进行兼容处理。

<img src="https://gongpengjun.com/imgs/im707/im008_03_app_versions_and_srv_deploy_1.svg" width="100%" alt="IM008-One IM, One App">

一般来说，不同的客户端iOS、Android、Windows、Mac都是同步迭代，多端发版时间一致，App版本号也一样。

所以用跨多端的App版本号可以很容易地让服务端只用写一遍判断和兼容逻辑。

示例：假设从V2.1.0开始应用红包消息，那么判断客户端是否支持红包的逻辑伪代码如下

```java
boolean isClientSupportRedEnvelopMessage(String appVersion) {
  return greaterThanOrEqual(appVersion, "2.1.0");
}
```

附: 版本号比对逻辑 (未充分考虑异常情况)

```java
List<Integer> versionNumbers(String version) {
  Matcher matcher = Pattern.compile("/[0-9]+\\.[0-9]+\\.[0-9]+").matcher(version);
  String versionString = matcher.find() ? matcher.group(0).substring(1) : "1.0.0";
  List<Integer> verNums = Arrays.stream(versionString.split("\\."))
                                .map(Integer::valueOf)
                                .collect(Collectors.toList());
  return verNums;
}

boolean greaterThanOrEqual(String appVersion, String targetVersion) {
  List<Integer> appVerNums = versionNumbers(appVersion);
  Integer appMajorVerNum = appVerNums.get(0);
  Integer appMinorVerNum = appVerNums.get(1);
  Integer appPatchVerNum = appVerNums.get(2);
  List<Integer> targetVerNums = versionNumbers(targetVersion);
  Integer targetMajorVerNum = targetVerNums.get(0);
  Integer targetMinorVerNum = targetVerNums.get(1);
  Integer targetPatchVerNum = targetVerNums.get(2);
  return (appMajorVerNum >= targetMajorVerNum) || 
         (appMinorVerNum >= targetMinorVerNum) || 
         (appPatchVerNum >= targetPatchVerNum);
}
```

## 2、多个App时怎么办？

只有一个App比较简单。现实是一套IM系统用于多个业务场景是很普遍的现象。业界的知名IM产品：钉钉、飞书、企业微信、美团大象等都是这样。底层逻辑大概是：IM系统比较复杂，功能繁多而且难以实现，更难以稳定，所以一个IM团队维护一套IM系统，然后应用在多个业务场景就是最具性价比的选择了。

### 2.1、使用App版本号

每个业务场景都会有自己的客户端App，每个App都有自己的版本号，那么根据App版本号判断新老版本的逻辑就不适用了。

<img src="https://gongpengjun.com/imgs/im707/im008_03_app_versions_and_srv_deploy_2.svg" width="100%" alt="IM008-One IM, One App">

_一个App时_：

```java
boolean isClientSupportRedEnvelopMessage(String appVersion) {
  return greaterThanOrEqual(appVersion, "2.1.0");
}
```

_多个App时_:

```java
boolean isClientSupportRedEnvelopMessage(String appVersion) {
  return (yaml.appName.equals("App 1") && greaterThanOrEqual(appVersion, "2.1.0")) ||
         (yaml.appName.equals("App 2") && greaterThanOrEqual(appVersion, "2.2.3")) ||
         (yaml.appName.equals("App 3") && greaterThanOrEqual(appVersion, "6.1"));
}
```

### 2.2、使用App版本号的麻烦

随着App的增多，需要的判断也越多，这会很麻烦，也很容易出错。

每个App推出新版本后，用户不可能瞬间就升级到最新版本，根据经验，每个App往往都会同时存在十个以上的不同版本。这就会形成如下图所示的局面：

<img src="https://gongpengjun.com/imgs/im707/im008_03_app_versions_and_srv_deploy_3.svg" width="100%" alt="IM008-One IM, One App">

## 3、多个App情况分析

多个App时的问题总结起来就是：一套服务端代码如何适应集成了不同IM能力的不同App客户端？

### 3.1、一套代码多个App共享

分析一下，如下图所示：假设一个IM团队维护的IM相关的客户端模块有IM Client SDK、联系人、长连接、朋友圈等四个模块。

<img src="https://gongpengjun.com/imgs/im707/im008_01_one_im_mutilple_apps.svg" width="100%" alt="IM008-One IM, Multiple Apps">

- App 1：集成了全部四个模块。
- App 2：只集成了三个模块。
- App 3：只集成了三个模块。

因为三个App面向的客户群不同，发版节奏不同，所以各自集成的IM的能力也不同。比如：

- App 1：面向内部员工办公沟通使用的App 1需要功能丰富，对于稳定性和Bug有一定的包容性，也容易沟通和修复再发版。
- App 2：面向客服场景，用于企业的客服专员和企业的C端用户沟通解决客诉问题，对于稳定性要求高，C端用户升级率不好控制，发版节奏慢，最快只能和主业务App一致。
- App 3：面向企业和B端供应商，比如美团和美团上的商户，京东和京东平台上的第三方商家，对于稳定性要求也比较高，B端商家的升级率好控制一点，发版节奏也可以快一些。

从上图可以看出，因为IM核心能力是同一个团队维护，所以Core包含的多个模块的代码必然是只有一套源代码。不同App只是Core集成打包出来的产物，或者说不同App只是Core外面套了不同的壳而已，只要Core一样，则App的IM能力就一样。

### 3.2、给Core一个版本号

那么，我们能不能给Core一个版本标识呢？

<img src="https://gongpengjun.com/imgs/im707/im008_03_app_versions_and_srv_deploy_4.svg" width="100%" alt="IM008-One IM, Multiple Apps">



### 3.3、给App打上Core版本标签

站在App的角度，每个App相当于打上了Core版本标签：

<img src="https://gongpengjun.com/imgs/im707/im008_03_app_versions_and_srv_deploy_5.svg" width="100%" alt="IM008-One IM, Multiple Apps">

### 3.4、抛开App看Core版本

如果不看App版本，只看Core版本标签：

<img src="https://gongpengjun.com/imgs/im707/im008_03_app_versions_and_srv_deploy_6.svg" width="100%" alt="IM008-One IM, Multiple Apps">

### 3.5、从一套服务端代码看Core版本

同一个IM团队，其IM Servers必然也是同一套代码集，不考虑部署的区别，那么上图逻辑上等价于下图：

<img src="https://gongpengjun.com/imgs/im707/im008_03_app_versions_and_srv_deploy_7.svg" width="100%" alt="IM008-One IM, Multiple Apps">

### 3.6、使用Core版本的兼容性判断

站在Core的视角，多个App就像单个App类似，只是使用的版本标识不同。

- 单个App时，IM服务端要区分不同的App版本
- 多个App时，IM服务端要区分不同的Core版本

还拿是否支持红包的判断举例：

_一个App时_：

```java
boolean isClientSupportRedEnvelopMessage(String appVersion) {
  return greaterThanOrEqual(appVersion, "2.1.0");
}
```

_多个App时_:

```java
boolean isClientSupportRedEnvelopMessage(Integer coreVersion) {
  return coreVersion >= 2;
}
```

通过Core版本号，我们可以把兼容逻辑判断简化到和单个App一样的简单。

### 3.7、关于Core版本的命名和取值

关于Core版本号的取值，有下列可能的选项：
- 选项一：语义版本号 1.2.0
- 选项二：整数 自然数 1 2 3
- 选项三：整数 迭代日期 20220819 或 220819

因为Core版本号不用给最终用户看的，无需遵循常见的语义版本号规范。而且Core版本号只用于版本对比，所以整数会是一个比较好的选择，方便比较，准确可靠。

用自然数 1、 2、 3作为Core版本号是可以的，每个迭代发布新的Core版本时递增一下就可以了。

但是考虑到有多个终端平台iOS、Android、Windows、Mac，如果某个平台的Core发布后发现小Bug需要HotFix，那么要递增版本号，就会挤占其它端的下一个自然数。究其原因，在于自然数是连续的，没办法在两个常规的版本间插入一个HotFix版本。

<img src="https://gongpengjun.com/imgs/im707/im008_04_feature_and_core_versions.svg" width="100%" alt="IM008-One IM, Multiple Apps">

选项三就可以解决这个问题，因为Core的迭代发布日期是稀疏的，若干天才会发布一个Core版本，那么当某个端需要一个HotFix版本时，选择HotFix当天的日期作为版本号即可。

总体上，多个端的主要版本号都是约定的统一的发布日期，多端一致，同时允许某个端临时HotFix插入一个新的版本号，保留弹性。

参考 Google 对[Android SDK API版本](https://apilevels.com/)的实践，我们可以把Core版本号命名为`core_level`，取值为Core的发布日期的整数表示。

### 3.8、其它版本标识

#### 3.8.1、platform

一套Core，不同端在实际开发中，可能存在差异，为了针对具体端进行特定的兼容，需要知道当前是哪个端，可以约定`platform`字段表示端。取值可以是：`ios`、`android`、`win`、`mac`、`linux`等。

#### 3.8.2、App版本号

在IM相关逻辑的兼容性判断中，只需使用跨App的多端一致的`core_level`了。但是为了和最终用户、产品经理等沟通方便，保留App版本号`app_version`用于人和人之间沟通交流。`core_level`主要用于研发工程师之间，还有工程师和程序之间的沟通。两者各取所长。

### 3.9、版本标识传输方式

每个API和每条长连接数据包都携带Core版本，这样服务端可以无状态得处理每一个请求。如果需要在服务端主动推送时区分目标端的版本，可以在App登录时将其携带的Core版本落库存储，然后推送时查询使用。

#### 3.9.1、短连接

HTTP短连接通过新增Header字段方式传输：

```shell
curl "https://{domain}/api/v1/xxx" \
  -H "platform: ios" \
  -H "app_version: 8.0.25" \
  -H "core_level: 220819" \
  -H "traceid: 0ad1348f1403169275002100356696"
```

#### 3.9.2、长连接

长连接SDK通过类似HTTP Header的方式传输：

```json
{
  "platform": "ios",
  "app_version": "8.0.25",
  "core_level": "220819",
  "traceid": "0ad1348f1403169275002100356696"
}
```

#### 3.9.3、短转长

短转长时HTTP Header会转换为长连接数据body里的header通过长链传递。

这样就同时存在长连接header和长连接body.header两套字段，最终以长连接body.header为准即可。

#### 3.9.4、其它

IM系统里的浏览器和小程序，如果可以新增HTTP Header则新增Header传输，实在没有办法可以通过User-Agent传输该信息，服务端优先解析Header，没有找到时再解析User-Agent。

服务端解析UA的[正则表达式](https://regex101.com/r/BONapT/1)：

```java
/ platform\/(ios|android|mac|win|linux) app_version\/([0-9]\.[0-9]+\.[0-9]+) core_level\/([1-9][0-9]+)( |$)/
```

## 4、多个App解决方案总结

至此，我们找到了一个适用于多个App、多个子模块、多个功能点、临时BugFix的版本标识：Core版本号，可以很好地解决多App兼容性问题。

```java
boolean isClientSupportRedEnvelopMessage(Integer coreLevel) {
  return coreLevel >= 220819;
}
```

## 5、参考资料

- [Browser vs Engine Version](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Browser_detection_using_the_user_agent)：Browser对应本文的App，Engine对应本文的Core
- [Node.js ABI version number](https://nodejs.org/en/download/releases/)：NODE_MODULE_VERSION
- [Android SDK API Level](https://apilevels.com/)：API Level整数便于判断、使用和准确沟通
