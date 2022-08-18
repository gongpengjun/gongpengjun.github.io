---
layout: post
title: UserAgent的坑
date: 2022-08-18 09:00:00
categories: API
---

关于User-Agent，人尽皆知。但是我知之甚少，于是Google并整理一二。

### User-Agent的历史



### User-Agent的用途

- 统计分析：网络分析工具
- 识别爬虫：网站识别爬虫来屏蔽爬虫的爬取；爬虫伪装自己来爬取。
- 浏览器兼容：网站根据UA判断浏览器的能力并返回适合的内容格式。
- 浏览器不兼容：微软的网站针对Firefox特意不兼容，将Firefox的UA改为IE就一切正常了。

### User-Agent的弊病

- 暴露用户隐私

### User-Agent的篡改



### User-Agent的解析



### User-Agent的未来



### 参考资料

- [User-Agent Client Hints - Draft Community Group Report, 1 July 2022](https://wicg.github.io/ua-client-hints/)
- Google [Chrome Platform Status - Feature: Sec-CH-UA Client Hints](https://chromestatus.com/feature/5995832180473856)
- Google Chrome [User-Agent reduction](https://developer.chrome.com/docs/privacy-sandbox/user-agent/)
- Google Chromium [User-Agent Reduction](https://www.chromium.org/updates/ua-reduction/)
- Google Chromium [User-Agent Reduction Origin Trial and Dates](https://blog.chromium.org/2021/09/user-agent-reduction-origin-trial-and-dates.html)
- Google Web.dev [通过用户代理客户端提示改善用户隐私和开发者体验](https://web.dev/i18n/zh/user-agent-client-hints/)
- Google [User-Agent Client Hints Demo](https://user-agent-client-hints.glitch.me/)
