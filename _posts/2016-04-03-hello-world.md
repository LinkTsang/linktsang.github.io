---
layout: post
title: Hello GitHub.io
date: 2016-04-03 00:00:00
tags: "blah-blah"
---

最后还是选择了 GitHub.io (o .o
搜了一圈, 最后决定先试下Hexo, 以后有空再折腾别的, 顺便记录下Windows下Hexo搭建过程（虽然网上随便一搜一大堆 =_=

<!-- more -->
# 安装清单
- Git
- Node.js
- Hexo
参见[官方文档](https://hexo.io/zh-cn/docs)
```bash
  npm install -g hexo-cli
  npm install hexo --save

  hexo init
  npm install hexo-deployer-git --save #插件
  hexo s -i localhost -p 4242 -o       #预览
```

## 多说评论
参考 [Hexo使用多说教程](http://dev.duoshuo.com/threads/541d3b2b40b5abcd2e4df0e9)
在 `_config.yml` 中添加
```xml
duoshuo_shortname: <short_name>
```
在 `themes\<theme_name>\layout\_partial\article.ejs` 加入
```html
  <% if (!index && post.comments && config.duoshuo_shortname){ %>
  <section id="comments">
    <div class="ds-thread" data-thread-key="<%= post.layout %>-<%= post.slug %>" data-title="<%= post.title %>" data-url="<%= page.permalink %>"></div>
    <script type="text/javascript">
    var duoshuoQuery = {short_name:'<%= config.duoshuo_shortname %>'};
      (function() {
        var ds = document.createElement('script');
        ds.type = 'text/javascript';ds.async = true;
        ds.src = (document.location.protocol == 'https:' ? 'https:' : 'http:') + '//static.duoshuo.com/embed.js';
        ds.charset = 'UTF-8';
        (document.getElementsByTagName('head')[0] 
         || document.getElementsByTagName('body')[0]).appendChild(ds);
      })();
      </script>
  </section>
  <% } %>
```
## analytics
```html
<% if (theme.tencent_analytics){ %>
<script type="text/javascript" src="http://tajs.qq.com/stats?sId=<%= theme.tencent_analytics %>" charset="UTF-8"></script>
<% } %>

<% if (theme.cnzz_analytics){ %>
<div hidden>
<script src="//s95.cnzz.com/z_stat.php?id=<%= theme.cnzz_analytics %>&web_id=<%= theme.cnzz_analytics %>" language="JavaScript"></script>
</div>
<% } %>
```
# 部署到GitHub
当然在GitHub先得有个repository, `Repository name`填`<name>.github.io`, `<name>`即GitHub用户名
## 生成并添加SSH key
```bash
$ git config --global user.name <name>
$ git config --global user.email <email>
$ ssh-keygen -t rsa -C <email>

$ eval `ssh-agent -s`
$ ssh-add ~/.ssh/id_rsa
```
将公钥`id_rsa.pub`的内容扔到repository的`Settings`中的`Deploy keys`(写这篇文章时地址为`https://github.com/<username>/<username>.github.io/settings/keys`), 然后进行验证
```bash
$ ssh -T git@github.com
```
## 部署
参考[Hexo部署](https://hexo.io/zh-cn/docs/deployment.html), 修改`_config.yml`如下
```yml
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: git@github.com:<username>/<username>.github.io.git
```
执行
```
$ hexo d -g
```
完成部署


# 参考资料
http://ibruce.info/2013/11/22/hexo-your-blog/
https://xuanwo.org/2015/02/07/generate-a-ssh-key/
