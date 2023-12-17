---
{"dg-publish":true,"permalink":"/Pages/怎么让自己的博客被google检索到/","title":"怎么让自己的博客被google检索到","tags":["obsidian"]}
---


# 怎么让自己的博客被 Google 检索到

- 打开 Google Search Console [here](https://search.google.com/search-console)  
- 在弹出的页面中，根据自己情况填入自己的网站, 我的网站是通过 netlify 部署的，选择了第二个  
![](https://cdn.jsdelivr.net/gh/aiyolo/imgrepo@main/test/bA4WIo5.png)
- 在验证所有权这一步中，第一项和第二项可能对一些通过 netlify 这样的网站部署的站点失效，我使用第一项验证就死活不成功，最后换成了第二个 `HTML 标记` 后终于验证成功了  
![](https://cdn.jsdelivr.net/gh/aiyolo/imgrepo@main/test/NtEobKT.png)

- 接着按照提示在你的站点主页的第一个 body 的前面一个 head 中加入它给出的内容，如下

```
<meta name="google-site-verification" content="xLMKsqEacL4fQxUh9qzuWRasQhPv8lR65yRAUU65AgE" />
    <title>{{ noteTitle }}</title>
```

![](https://cdn.jsdelivr.net/gh/aiyolo/imgrepo@main/test/3n36BPh.png)

- 成功添加，等待 google 处理数据，大概需要一天时间

![](https://cdn.jsdelivr.net/gh/aiyolo/imgrepo@main/test/T3KGoUO.png)
