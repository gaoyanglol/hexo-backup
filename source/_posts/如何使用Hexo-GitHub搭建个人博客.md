---
title: 如何使用Hexo+GitHub搭建个人博客
date: 2018-03-07 00:21:25
tags:
---
## 什么是[Hexo](https://hexo.io/zh-cn/docs/)？

> Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

我们可以使用Hexo将博客搭建在GitHub上，而且可以通过GitHub Pages来展示自己的博客。同时我们可以享受到git的便利：只需修改Hexo的配置，将部署方式设置为git，就可以通过向仓库提交变更的方式来发布新文章。全程不需打开浏览器。

（PS:需要前置知识Git命令及安装,本文在此不作说明）

---

## 如何安装Hexo

*安装前请先确保电脑中已安装Node.js和Git。*

1. 在GitHub上新建一个空的repository，名称为 `xxxx.github.io` 。（xxx替换为你的用户名）
2. 打开Git Bash，运行以下命令安装Hexo。

       $ npm install -g hexo-cli

3. 进入一个安全的目录，如：

        $ cd ~/Desktop
4. 运行：

        $ hexo init myBlog

5. 进入 `myBlog` 目录：
        
        $ cd myBlog

6. 下载库文件：

        $ npm i

7. 创建一片新博客：

        $ hexo new 开博大吉

第一篇博客就这样创建成功了。

这时候你可以在 `myBlog\source\_posts\` 目录下看到 `开博大吉.md` 这个 `markdown` 格式的文件。

可以用编辑器如 `VS Code` 打开它进行编辑。

---

## 配置Hexo

1. 打开 `myBlog` 目录下的 `_config.yml` 文件。找到5-11行：

         # Site
        title: 
        subtitle:
        description:
        author: 
        language:
        timezone:

- 在 `title:` 后面输入你想要的博客名称。

- 在 `author` 后面输入你的笔名。（真名也可以，随你）

- 找到79-80行：

        deploy:
          type: 

- 在 `type:` 后面输入 `git` ；

- 在 `type:` 下新增一行 `repo:` 。后面跟你前面在GitHub上创建repository后生成的SSH地址，如：

        deploy:
          type: git
          repo: git@github.com:你的用户名/xxx.github.io.git

  *`repo:` 左边需和 `type:` **齐平**。*

  *`type:` 和 `repo:` 和后面的值间都有一个**空格**。*

2. 安装git部署插件，在Git Bash中运行：

        $ npm install hexo-deployer-git --save

3. 让Hexo进行部署：

        $ hexo deploy

4. 进入GitHub中 `xxx.github.io` 对应的repository。在 `settings`中开启 GitHub Pages。如果已经开启，直接点击链接预览。

---

## 更新博客

1. 生成一篇新博客：

        $ hexo new 第二篇博客

2. 编辑博客内容。

3. 生成博客：

        $ hexo generate

4. 部署：

        $ hexo deploy

5. 刷新博客，应该可以看到新的内容了。

---

## 更换主题

时间太晚了我就不自己写了，引用一下 [方应杭](https://www.zhihu.com/people/zhihusucks/activities) 老师的总结。

> 1. https://github.com/hexojs/hexo/wiki/Themes 上面有主题合集
> 2. 随便找一个主题，进入主题的 GitHub 首页，比如我找的是 https://github.com/iissnan/hexo-theme-next
> 3. 复制它的 SSH 地址或 HTTPS 地址，假设地址为 git@github.com:iissnan/hexo-theme-next.git
> 4. `cd themes`
> 5. `git clone git@github.com:iissnan/hexo-theme-next.git`
> 6. `cd ..`
> 7. 将 _config.yml 的第 75 行改为  `theme: hexo-theme-next` ，保存
> 8. `hexo generate`
> 9. `hexo deploy`
> 10. 等一分钟，然后刷新你的博客页面，你会看到一个新的外观。如果不喜欢这个主题，就回到第 1 步，重选一个主题。

---

## 备份博客源码

由于在 `xxx.github.io` 上保存的只是由Hexo部署时提交的博客的内容，并不包含 `myBlog` 这个里项目的源码。为了以防万一，我们最好将 `myBlog` 中的程序代码也上传到GitHub进行备份和管理。

1. 在GitHub上新建一个空的repository，名字随便取。我们暂且叫 `blog-generator`。

2. 在Git Bash中进入 `myBlog` 目录：

        $ cd ~/Desktop/myBlog

3. 依次输入下面几行命令：

        $ echo "# blog-generator" >> README.md
        $ git init
        $ git add README.md
        $ git commit -m "first commit"
        $ git remote add origin git@github.com:你的用户名/blog-generator.git
        $ git push -u origin master

这样你的 `myBlog` 目录就Git（这里不知道怎么形容这个操作，映射？备份？）到了 `blog-generator` 上。但是在 `blog-generator` 仓库中你会发现现在只有一个 `README.md` 文件。

因为我们并没有将 `myBlog` 目录下的其他文件 `add` 后 `commit`。此时如果你输入：

         $ git status -sb

会发现许多行以 <font color="red">??</font> 开头的路径。这些都是没有被git追踪的文件，需要 `add` 一下下。

        $ git add 文件名或目录

全部 `add` 完后，再 `git status -sb` 检查下。如果最前面都变成了绿色的 <font color="green">A</font> ，就表示这些文件都被Git成功地添加到了缓存区。万事俱备，只差 `commit`：

        $ git commit -m "此次提交的备注信息"

最后：

        $ git push -u origin master

再到 `blog-generator` 里面刷新一下，应该就能看到新增的许多文件了。

以后每次更新博客后，再 `add` - `commit` - `push` 一下变更的文件。就万无一失了。

---

-完-
