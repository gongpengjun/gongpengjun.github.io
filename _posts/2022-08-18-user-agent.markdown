---
layout: post
title: UserAgent
date: 2022-08-18 09:00:00
categories: HTTP
---

关于User-Agent（简称UA），最近做了一些调研，总结一些常识。

### UA的历史

[RFC9110](https://www.rfc-editor.org/rfc/rfc9110#section-3.5)中定义了user agent，特别说明不止浏览器，爬虫、命令行程序都是user agent。

[RFC9110](https://www.rfc-editor.org/rfc/rfc9110#section-10.1.5)中定义了 User-Agent HTTP请求头字段的格式和内容。user agent发送每个HTTP请求时都应该携带User-Agent字段。

### UA的用途

- 统计分析：商业广告公司通过统计分析UA来跟踪用户的行为，然后进行个性化推荐。
- 识别爬虫：网站识别爬虫来屏蔽爬虫的爬取；爬虫伪装自己来爬取。
- 浏览器兼容：网站根据UA判断浏览器的能力并返回适合的内容格式。
- 浏览器不兼容：微软的网站针对Firefox特意不兼容，将Firefox的UA改为IE就一切正常了。

### UA解析

一句话总结：UA解析不难但是很难保证100%可靠。



Using the user agent to detect the browser looks simple, but doing it well is, in fact, a very hard problem. 
使用UA检测浏览器看起来很简单，但是做得很好，实际上是一个非常困难的问题。



As there is no uniformity of the different part of the user agent string, this is the tricky part.
由于用户代理字串的不同部分缺乏统一性，这是一个较难对付的问题。



A technological survey must be in place to adapt the script when new browser versions are coming out.
当新的浏览器版本发布的时候，必须进行一次技术调研，看是否需要和怎么修改UA解析脚本以适应新的浏览器。



**Mobile device detection**

推荐方法：优先使用功能侦测（Feature Detect），最后没办法时才fallback到UA检测。

Use Navigator.maxTouchPoints to detect if the user's device has a touchscreen. Then, default back to checking the user agent screen only if (!("maxTouchPoints" in navigator)) { /*Code here*/}. 
<img src="https://gongpengjun.com/imgs/UA_hasTouchScreen.png" width="100%" alt="Mobile device detection">



**UA解析开源库**
[ua-parser](https://github.com/ua-parser)

- https://github.com/ua-parser/uap-core
- https://github.com/ua-parser/uap-java 
- https://github.com/ua-parser/uap-go

ua-parser开源库集合支持各种常见编程语言，其中uap-core项目中的正则表达式文件[regexes.yaml](https://github.com/ua-parser/uap-core/blob/master/regexes.yaml)是关键，可以参考使用。



正则表达式工具网站在解析时很有用：https://regex101.com/

### UA演进

Google正在推动废除UA或冻结UA字段（只读），用[UA Client Hints](https://wicg.github.io/ua-client-hints/)方案替代，并已形成W3C草案。

### 参考资料

- [Browser detection using the user agent](https://developer.mozilla.org/en-US/docs/Web/HTTP/Browser_detection_using_the_user_agent)
- [使用UserAgent字段进行浏览器检测](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Browser_detection_using_the_user_agent)
- [User-Agent Client Hints - Draft Community Group Report, 1 July 2022](https://wicg.github.io/ua-client-hints/)
- Google [Chrome Platform Status - Feature: Sec-CH-UA Client Hints](https://chromestatus.com/feature/5995832180473856)
- Google Chrome [User-Agent reduction](https://developer.chrome.com/docs/privacy-sandbox/user-agent/)
- Google Chromium [User-Agent Reduction](https://www.chromium.org/updates/ua-reduction/)
- Google Chromium [User-Agent Reduction Origin Trial and Dates](https://blog.chromium.org/2021/09/user-agent-reduction-origin-trial-and-dates.html)
- Google Web.dev [通过用户代理客户端提示改善用户隐私和开发者体验](https://web.dev/i18n/zh/user-agent-client-hints/)
- Google [User-Agent Client Hints Demo](https://user-agent-client-hints.glitch.me/)
