---
title: github pages 部署总结
date: 2016-07-25 16:30:23
tags: [blog]
published: true
hideInList: false
feature: 
---


截止今天为止，凡是我遇到的部署博客的问题，无论是 Jekyll 还是 Hexo 方面的，都一一有了答案，想着，写一篇博客，当做总结 。

### 部署前提

在 Github 上面申请一个仓库，仓库的名词必须是 username.github.io

* Jekyll 部署方案

        Jekyll 是一个简单的博客形态的静态站点生产机器。 它有一个模版目录，其中包含原始文本格式的文档，通过一个转换器（如Markdown）
        和我们的 Liquid 渲染器转化成一个完整的可发布的静态网站，你可以发布在任何你喜爱的服务器上。
这个引擎在 github 的服务器上就有，所有我们要做的仅仅是将自己的文件按照 jekyll 引擎可以渲染的目录结构 push 上 github 就可以了
    
    * Jekyll 的目录结构
    
            .   
            |--_config.yml>  
            |--_drafts  
                   |--articles1.textile 
                |--articles2.md
            |--_includes
                |--footer.html
                |--header.html
            |--_layouts
                |--default.html
                |--post.html
            |--_posts
                |--2014-06-17-articles1.textile
                |--2014-06-17-articles1.md
            |--_site
            |--index.html
            |--other files
    
    
    网上关于 jekyll 目录结构的讲解文章有很多，我就不在这里赘述了，文末会有参考资料。
    
    * Jekyll 解析流程
            
        * 首先会加载_posts及文件夹下的所有文章，将其参数和文章内容组织保存在内存中，所有的文章的内容、参数都在site.posts对象（其他文件夹下的文章不会放入site.posts中）。
        * 其次加载_layouts文件夹下的所有模板。
        * 再次加载_includes文件夹下的所有需要被引入的内容。
        * 最后根据每一篇需要编译的文章选择的其参数定义的模板来创建一个模板，并将当前文章的内容、参数等进行扩展后放在page对象、content对象中，然后进行模板的编译，生成html文件，并按照一定规则放在_site文件夹下。也就是说在创建一篇文章时，其实所有文章的内容都已经被读取出来了，这也为文章相互之间的关联提供了可能。
         
        
        正是由于 jekyll 会在渲染一篇文章时将其他文章也一并加载，所以 jekyll 只适合搭建中小型博客。 
        
        * 由于 Github Pages 完美支持 Jekyll ，所以你只需要安照 Jekyll 的文件目录结构，托管在 Github 中即可，不需要在本机安装 Jekyll ，只需要关注于 Github，可以借鉴大师们设计的 Jekyll 模板，然后 push 或 pull 你的文章即可。
            * 新建一个文件夹，并且初始化仓库。
            * git clone 你的 username.github.io 仓库
            * 找一个大师的[模板](https://github.com/jekyll/jekyll/wiki)，然后放进自己的本地文件夹
            * 更改大师的配置，push 到 Github 中
            
        * 访问 username.github.io
        * 关于评论
            评论我使用的是 Disqus ，你只需要注册一个账号，然后把插件代码复制到自己的 _layouts 里的模板中就可以了，不过这里要注意布局，建议使用 [footer 布局](https://sfmonkey.github.io/blog/2016/07/footer.html)中的推拉退布局
    
    * Hexo 部署方案
    
            Hexo 是一个快速、简洁且高效的博客框架。 Hexo 使用Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。
        
        * 准备环境
            * Nodejs
                
                * Windows
                    * 前往[官网](https://nodejs.org/en/)下载安装
                * ubantu
                    * sudo apt-get install nodejs
        
        * 安装 Hexo
        
            创建一个文件夹并初始化它，下面的一些命令不清楚的，文末有相关资料，注意 npm 的使用位置，一定要是你创建的文件夹，nodejs 的安装源很重要，否则可能出现错误。
            
            *  npm install hexo-cli -g
                
                可能你会看到一个WARN，但是不用担心，这不会影响你的正常使用。
           
            *  npm install hexo --save
            
                然后你会看到命令行窗口刷了一大堆白字，下面我们来看一看Hexo是不是已经安装好了
                
            *  hexo -v        
            *  hexo init
                
                初始化
            *  npm install
            
                安装所需组件
            *  hexo g
            *  hexo s
                在浏览器中打开http://localhost:4000/，本地 Hexo 搭建成功
            
            *  配置 Deployment
                
                    git config --global user.name "yourname"
                    git config --global user.email "youremail"
            *   同样在_config.yml文件中，找到Deployment，然后按照如下修改
            
                    deploy:
                      type: git
                      repo: git@github.com:yourname/yourname.github.io.git
                      branch: master
            *  npm install hexo-deployer-git --save
                如果使用git方式进行部署，执行npm install hexo-deployer-git --save来安装所需的插件
            
            *  Hexo d
            *  hexo d -g
                
                部署博客
            *  username.github.io
    
    关于更改域名等不在这里说了，网上有很多，文末我会附上资料。
    
    
    
    
    
    补充: 前天部署的时候突然出现问题，由于 Github 没有发来邮件提醒，而我的文章又只出现在 Github 上而未出现在博客上，导致我以为是时区问题造成的 future 未发布，其实不然，问题出现在了编码问题上，由于 Github 不知为何未发错误邮件，所以好久都没找到问题所在。 
    
### 相关资料: 
[Git+GitHub+Markdown+Jekyll=Perfect Personal Blog](http://www.devtalking.com/articles/git-gitHub-markdown-jekyll/)

[如何搭建一个独立博客——简明Github Pages与Hexo教程](http://www.jianshu.com/p/05289a4bc8b2)

[史上最详细的Hexo博客搭建图文教程](https://xuanwo.org/2015/03/26/hexo-intor/)

[文档 Hexo](https://hexo.io/zh-cn/docs/)
          