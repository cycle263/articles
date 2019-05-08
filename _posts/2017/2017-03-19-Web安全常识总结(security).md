---
layout:     post
title:      "Web安全常识总结(security)"
subtitle:   ""
date:       2017-03-19
author:     "Cycle263"
header-img: "img/article-bg.jpg"
tags:
    - 前端开发
    - web开发
---

## Web安全常识总结(security)

* xss(Cross Site Script, 跨站脚本攻击)

  XSS 就是攻击者在 Web 页面中插入恶意javascript脚本和html，当用户浏览该页之时，嵌入其中 Web 里面的脚本代码会被执行，从而达到攻击目的。

  **攻击危害**

  - 虚假form表单骗取用户信息，储存危害脚本或代码
  - 脚本获取cookie信息，并发起恶心请求
  - 渲染虚假信息和图片，骗取用户信任
  
  **解决办法：**

  - 从客户端和服务器端双重验证所有的输入数据，这一般能阻挡大部分注入的脚本
  - 对所有的数据进行适当的编码， HTML 转义，包括链接跳转，redirect_to重定向等，简单的做法是，对所有输入输出的内容，将引号、尖括号、斜杠进行转义
  - 设置 HTTP Header 中的 `Content-Security-Policy`， meta 标签的方式 `<meta http-equiv="Content-Security-Policy">`
  - Web 页面渲染的所有内容或者渲染的数据都必须来自于服务端，尽量不要取URL或者document.referrer等取数据渲染，尽量不要使用`eval、setTimeout`等可以直接执行字符串的方法，无法避免则需进行 escape 转义
  - 预防xss攻击获取cookie，可以在每个cookie属性里设置为HttpOnly

  **富文本**

  对于富文本来说，显然不能通过上面的办法来转义所有字符，因为这样会把需要的格式也过滤掉。对于这种情况，通常采用白名单过滤的办法。成熟的js库有，` js-xss`。

* SQL injection (SQL注入)

  SQL 注入，就是通过客户端的输入把 SQL 命令注入到一个应用的数据库中，从而得以执行恶意 SQL 语句。解决办法：

  - Prepared Statements (with Parameterized Queries)： 参数化的查询语句可以强制应用开发者首先定义所有的 sql 代码，之后再将每个参数传递给查询语句
  - Stored Procedures： 使用语言自带的存储程序，而不是自己直接操纵数据库
  - White List Input Validation： 验证用户的输入
  - Escaping All User Supplied Input： 对用户提供的所有的输入都进行编码

* DDoS（Distributed Denial of Service，分布式拒绝服务）

  DoS 攻击就是通过大量恶意流量占用带宽和计算资源以达到瘫痪对方网络的目的。

* CSRF(Cross Site Request Forgery, 跨站请求伪造)

  CSRF 就是网站 A 对用户建立信任关系后，在网站 B 上利用这种信任关系，跨站点向网站 A 发起一些伪造的用户操作请求，以达到攻击的目的。
  
  **解决方法：**

  - 检查标准头部，确认请求是否同源： 检查 source origin 和 target origin，然后比较两个值是否匹配
  - Referer 验证, 包括超链接的referrer policy
  - 不让第三方网站访问到用户 Cookie
  - 保证页面的幂等性，后端接口不要在GET页面中做用户操作
  - 检查 CSRF Token, 主要有四种推荐的方式
    + Synchronizer Tokens： 在表单里隐藏一个随机变化的 token，每当用户提交表单时，将这个 token 提交到后台进行验证，如果验证通过则可以继续执行操作。这种情况有效的主要原因是网站 B 拿不到网站 A 表单里的 token;
    + Double Cookie Defense： 当向服务器发出请求时，生成一个随机值，将这个随机值既放在 cookie 中，也放在请求的参数中，服务器同时验证这两个值是否匹配；
    + Encrypted Token Pattern： 对 token 进行加密
    + Custom Header： 使用自定义请求头部，这个方式依赖于同源策略。其中最适合的自定义头部便是： "X-Requested-With: XMLHttpRequest"


* noreferer

  `<a href="http://bank.example/withdraw?t=10" referrerpolicy="no-referrer">`

  从一个网站点击链接进入另一个页面时，浏览器会在header里加上Referer值，来标识这次访问的来源页面, 但是这种标识有可能会泄漏用户的隐私。基于HTML标准，可以在a标签内使用rel="noreferrer"来达到这一目的。提供跨浏览器支持的更好的办法是使用一个第三方的库noreferrer.js，它可以自动识别浏览器并选择最优方案。

  Http协议头中的Referer主要用来让服务器判断来源页面, 即用户是从哪个页面来的,通常被网站用来统计用户来源,是从搜索页面来的,还是从其他网站链接过来,或是从书签等访问,以便网站合理定位。Referer有时也被用作防盗链, 即下载时判断来源地址是不是在网站域名之内, 否则就不能下载或显示, 很多网站, 如天涯就是通过Referer页面来判断用户是否能够下载图片.

  - 其他方案

  ```js
  function open_without_referrer(link){
    document.body.appendChild(document.createElement('iframe')).src='javascript:"<script>top.location.replace(\''+link+'\')<\/script>"';
  }

  function open_new_window(full_link){
    window.open('javascript:window.name;', '<script>location.replace("'+full_link+'")<\/script>');
  }
  ```

  - 获取Referrer的方法，`document.referrer`

* 如何绕过图片防盗链？

  - 后端预下载，也就是将图片直接下载下来成为自己的，当然就不受防盗链的限制了，但有侵犯知识产权的问题存在

  - 第三方代理，也就是下载的过程交由第三方来处理，推荐常用的有：`https://images.weserv.nl?url=referrerUrl&w=100`，这种方式避免了产权问题，牺牲了性能。

  - 删除Header中的Referrer，MDN推荐关闭referrer方式，在meta标签中设置referrer值为no-referrer，这种方式相当于取消了文档中所有链接的referrer，也可以使用referrerPolicy，精准地指定某一个资源的referrer政策，`<img src="x.jpg" referrerPolicy="no-referrer" />`

