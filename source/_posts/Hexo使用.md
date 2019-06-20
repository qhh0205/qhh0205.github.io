---
title: Hexo使用
date: 2017-12-03 13:26:41
categories: Hexo
tags: Hexo
---

#### Hexo相关链接
* [Hexo+github博客搭建](https://thief.one/2017/03/03/Hexo%E6%90%AD%E5%BB%BA%E5%8D%9A%E5%AE%A2%E6%95%99%E7%A8%8B/)
* [Next主题配置](http://theme-next.iissnan.com/theme-settings.html)
* [Hexo next为文章添加分类](http://whx4j8.github.io/2016/03/16/hexo-next-%E6%B7%BB%E5%8A%A0%E4%B8%BA%E6%96%87%E7%AB%A0%E6%B7%BB%E5%8A%A0%E5%88%86%E7%B1%BB/)
* [Hexo文章插入图片](https://yanyinhong.github.io/2017/05/02/How-to-insert-image-in-hexo-post/)

#### Hexo 文章中插入gif动画
今天突然看别人的文章时发现文章内有代码动画，感觉挺有意思的，于是自己琢磨了下如何向hexo文章中插入动画。其实在hexo文章中插入git动画的方法和插入图片类似，只不过动画的插入不是使用MD自带的标签，而是hexo特有的标签插件--[iframe](https://hexo.io/zh-cn/docs/tag-plugins.html#iframe)，插入过程记录如下：
1. 要想在hexo文章中插入gif动画，首先得制作好动画
关于gif动画的制作，我的系统为Mac OS，因此使用受广大网友好评的LICEcap软件，该软件确实非常好用，专门录制git动态图，几乎没有学习成本，安装好就可以直接用了。关于如何用LICEcap录制gif图就不做介绍了，因为太简单了。
[LICEcap下载地址](https://www.cockos.com/licecap)
2. 在Hexo主目录的source文件夹下新建iframe文件夹，将上面录制的gif图放到该目录
3. 用hexo标签插件iframe在文章中引入上面制作的gif动画，引入方式如下：
```markdown
语法格式：{% iframe url [width] [height] %}
由于本文是在本地引入，所以url为/iframe/filename.gif，示例如下：
{% iframe /iframe/pyhelloworld.gif %}
不指定宽度和高度则使用默认大小
```
4. 效果如下：
{% iframe /iframe/pyhelloworld.gif %}

#### Hexo Next主题站点添加搜索功能 
1. 进入Hexo站点目录执行如下命令生成站点索引文件
``` bash
npm install hexo-generator-searchdb --save
```
2. 编辑站点配置文件，新增以下内容
``` bash
search:
  path: search.xml
  field: all
  format: html
  limit: 10000
```
3. 编辑Next主题配置文件，设置local_search配置的enable项为true
``` bash
local_search:
  enable: true
  # if auto, trigger search by changing input
  # if manual, trigger search by pressing enter key or search button
  trigger: auto
  # show top n results per article, show all results by setting to -1
  top_n_per_article: 1
```

#### Hexo Next主题添加文章及站点访问量统计
关于文章访问量统计新版next主题已经支持，内置使用不蒜子统计，是一个第三方服务。由于next主题已经内置支持该统计服务，因此不需要任何其他的配置，只需要修改该主题配置文件，将不蒜子统计功能激活即可。方法如下：
修改themes/next/_config.yml主题配置文件，找到busuanzi_count配置项，将不蒜子统计打开，修改后配置如下：
``` bash
# Show PV/UV of the website/page with busuanzi.
# Get more information on http://ibruce.info/2015/04/04/busuanzi/
# 访问量统计,启用卜算子统计
busuanzi_count:
# count values only if the other configs are false
enable: true
# custom uv span for the whole site
site_uv: true
site_uv_header: <i class="fa fa-user"></i> 访问人数
site_uv_footer:
# custom pv span for the whole site
site_pv: true
site_pv_header: <i class="fa fa-eye"></i> 总访问量
site_pv_footer: 次
# custom pv span for one page only
page_pv: true
page_pv_header: <i class="fa fa-file-o"></i> 浏览
page_pv_footer: 次
```
效果如下：
1. 文章访问量
![](/images/pv.png)
2. 站点访问量
![](/images/uv.png)

参考链接：
[http://theme-next.iissnan.com/third-party-services.html#analytics-busuanzi](http://theme-next.iissnan.com/third-party-services.html#analytics-busuanzi)

#### Hexo环境迁移
实现方法：在Hexo仓库新建一个hexo分支存放Hexo原始文件，master分支存放hexo生成的静态页面，如果迁移环境后可以直接git clone仓库即可。
1. 将Hexo仓库用git管理起来：
```bash
# 注意：不需要再编写.gitignore了，在Hexo工程已经默认有.gitignore文件了，这是hexo默认生成的，也许是hexo本来就推荐用git管理hexo原始文件吧
git init
git checkout -b hexo
git add .
git commit -m "init"
git remote add origin https://github.com/qianghaohao/qianghaohao.github.io.git
git push origin hexo
```
2. 在新的环境克隆仓库
```bash
git clone https://github.com/qianghaohao/qianghaohao.github.io.git
# 切换到hexo分支即可看到hexo原始文件，此时可以编辑并提交。一般先提交原始文件到hexo分支，然后hexo d部署生成的静态文件到master分支
git checkout hexo
```
参考文章：
[http://m.blog.csdn.net/zk673820543/article/details/52698760](http://m.blog.csdn.net/zk673820543/article/details/52698760)

#### Hexo Markdown简明语法手册
[Hexo Markdown简明语法手册](https://hyxxsfwy.github.io/2016/01/15/Hexo-Markdown-%E7%AE%80%E6%98%8E%E8%AF%AD%E6%B3%95%E6%89%8B%E5%86%8C/)
