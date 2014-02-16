---
layout: post
title: "搭建博客环境"
date: 2014-02-16 00:33:40 +0800
comments: true
categories: 随记
---

###1.Octopress需要Ruby环境，RVM(Ruby Version Manager)负责安装和管理Ruby的环境。所以我们先在终端输入如下命令，来安装RVM

``` 
curl -L https://get.rvm.io | bash -s stable --ruby
```
安装ruby1.9.3
``` 
rvm install 1.9.3
rvm use 1.9.3
rvm rubygems lastest
```
###2.安装Octpress
在安装octpress之前，确保安装了git。
下载octpress
```
git clone git://github.com/imathis/octopress.git octopress
cd octpress
```
安装相关依赖
```
gem install bundler
rbenv rehash
bundle install
```
安装默认的Octpress主题
```
rake install
```
###3.将博客部署到Github上
Github的Page service可以免费托管博客，并且还可以自定义域名。

首先需要在GitHub上创建一个仓库，并将仓库名称按照这样的方式进行命名：```username.github.com```或```organization.github.com```。等后面配置完毕之后，我们就可以在浏览器中使用页面地址```http://username.github.com```来访问我们的博客。一般来说，我们希望在将博客的源码放到source分支下，并把生成的内容提交到master分支。

创建好仓库之后，我们需要利用octopress的一个配置rake任务来自动配置上面创建的仓库：可以让我们方便的部署GitHub page。在终端输入如下命令：
```
rake setup_github_pages
```
完成上面的命令之后，我们就可以生成博客并真正的部署到仓库中了。执行如下命令：
``` 
rake generate
rake deploy
```
现在可以访问```http://username.github.com```了。注意：有时候可能会有延时，要等等10分钟左右才能打开。

不过博客的source需要单独提交，执行如下命令就可以将source提交到仓库的source分支下。
```
git add .
git commit -m 'Initial souece comment'
git push origin source
```
如果在部署到仓库之前，需要先预览一下博客，可以在终端输入
```
rake preview
```
然后就能在浏览器中进行本地预览访问了：```http://127.0.0.1:4000/```或```http://localhost:4000/```，效果跟仓库中的一样。

###4.开始写博客
新建一篇文章
```
rake 'new_post[title]'
其中title为博文的文件名，创建出来的文件默认是markdown格式。上面的命令会创建出这样一个文件：source/_posts/2014-2-16-title.markdown
```
打开这篇文章，可以看到
```
---
layout: post
title: title
date: 2014-0-16 00:33:40 +0800
comments: true
categoryies:
---
```
categories 填写文章的分类名

接着我们就可以在这个文件中写我们的博文。完成之后，我们可以预览和部署博文。下面是创建并部署博文的一个完整过程
```
rake 'new_post[title]'
rake generate
git add .
git commit -am "add new post"
git push origin source
rake deploy
```

###5.添加分类列表

保存一下代码到 plugins/category_list_tag.rb:
``` 
module Jekyll
  class CategoryListTag < Liquid::Tag
    def render(context)
      html = ""
      categories = context.registers[:site].categories.keys
      categories.sort.each do |category|
        posts_in_category = context.registers[:site].categories[category].size
        category_dir = context.registers[:site].config['category_dir']
        category_url = File.join(category_dir, category.gsub(/_|\P{Word}/, '-').gsub(/-{2,}/, '-').downcase)
        html << "<li class='category'><a href='/#{category_url}/'>#{category} (#{posts_in_category})</a></li>\n"
      end
      html
    end
  end
end

Liquid::Template.register_tag('category_list', Jekyll::CategoryListTag)
```

这个插件会向liquid注册一个名为category_list的tag，该tag就是以li的形式将站点所有的category组织起来。如果要将category加入到侧边导航栏，需要增加一个aside。
增加aside
复制以下代码到source/_includes/asides/category_list.html。
```
{% raw %}
<section>
  <h1>Categories</h1>
  <ul id="categories">
    {% category_list %}
  </ul>
</section>
{% endraw %}
```

配置侧边栏需要修改_config.yml文件，修改其default_asides项:
```
default_asides: [asides/category_list.html, asides/recent_posts.html]
```

###6.添加微博秀
先到微博秀里面生成自己的微博秀嵌入代码
```
<iframe width="100%"
 height="550" 
 class="share_self"  
 frameborder="0" 
 scrolling="no" 
 src="http://widget.weibo.com/weiboshow/index.php?language=&width=0&height=550&fansRow=2
&ptype=1&speed=0&skin=1&isTitle=1&noborder=1
&isWeibo=1&isFans=1&uid=1706688983
&verifier=1a722b18&dpc=1">
</iframe>
```

新建一个weibo.html 到source/_include/asides的下面
```
{% raw %}
{% if site.weibo_uid %}
<section>
  <h1>新浪微博</h1>
  <ul id="weibo">
    <li>
      <iframe 
        width="100%" 
        height="550" 
        class="share_self" 
        frameborder="0" 
        scrolling="no" 
        src="http://widget.weibo.com/weiboshow/index.php?width=0&height=550
        &ptype={% if site.weibo_pic %}1{% else %}0{% endif %}&speed=0&skin={{weibo_skin}}&isTitle=0&noborder=1
&isWeibo={% if site.weibo_show %}1{% else %}0{% endif %}&isFans={{weibo_fansline}}&uid={{site.weibo_uid}}&verifier={{site.weibo_verifier}}">
      </iframe>
    </li>
  </ul>
</section>
{% endif %}
{% endraw %}
```
修改_config.xml中default_aside部分
```
... 
default_asides: [asides/recent_posts.html, asides/weibo.html, asides/github.html, asides/[Twitter][].html, asides/googleplus.html]
...
weibo_uid: 1098907490
weibo_verifier: abd54ad9
weibo_fansline: 0   # 粉丝显示多少行
weibo_show: true    # 是否显示最近微博内容
weibo_pic: true     # 是否显示微博中的图片
weibo_skin: 10      # 使用哪种配色风格，数字为从1开始的微博秀风格序号
...
```


###7.评论和分享
在_config.yml中增加一项： weibo_share: true
修改 source/_includes/post/sharing.html ，增加：
```
{% raw %}
{% if site.weibo_share %}
     {% include post/weibo.html %}
 {% endif %}
 {% endraw %}
```
增加文件：source/_includes/post/weibo.html

访问http://www.jiathis.com/ , 获取分享的代码

访问http://uyan.cc/, 获取评论的代码

把代码都加到weibo.html中
















