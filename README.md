1. clone

  ```bash
  git clone git@github.com:mingever/blog.git
  ```

2. 切换分支

  ```bash
  git checkout dev
  ```

3. 安装node依赖

  ```bash
  npm install
  #package.json里依赖已约定好了
  
  #这样就仅局部安装hexo包了，可以使用以下两种方式执行 Hexo
  #1.使用 npx 来运行，npx hexo <command>
  #2.将 Hexo 所在的目录下的 node_modules\.bin 添加到环境变量之中即可直接使用 hexo <command>，如D:\Blog\node_modules\.bin
  ```

4. hexo  [文档地址](https://hexo.io/zh-cn/index.html)

  ```bash
  hexo clean  # 清除缓存
  hexo g      # 生成静态网页
  hexo d      # 部署到Github
  
  ==================================================================================
  hexo new page tags #添加标签  别忘记在index.md里添加type: "tags"
  hexo new page categories #添加分类  别忘记在index.md里添加type: "categories"
  hexo new page about #添加关于
  
  #分类和标签
  #只有文章支持分类和标签，可以在 Front-matter 中设置。
  #在其他系统中，分类和标签听起来很接近，但是在 Hexo 中两者有着明显的差别：分类具有顺序性和层次性而标签没有顺序和层次。
  categories:
  - Diary
  tags:
  - PS3
  - Games
  ```

5. Front-matter

  ```bash
  #Front-matter是文件最上方以 --- 分隔的区域，用于指定个别文件的变量,如
  ---
  title: Hello World
  date: 2023/4/25 20:46:25
  ---
  #一般Front-matter使用的yaml语法，yaml语法需要注意空格
  
  #分类和标签
  #只有文章支持分类和标签，可以在 Front-matter 中设置。
  #在其他系统中，分类和标签听起来很接近，但是在 Hexo 中两者有着明显的差别：分类具有顺序性和层次性而标签没有顺序和层次。
  tags: Games
  #使用数组
  categories:
  - Diary
  tags:
  - PS3
  - Games
  
  #文章摘要
  #1.在Front-matter里添加 description
  description: 这是显示在首页的概述，正文内容均会被隐藏。
  #2.在想显示为摘要的内容之后添 <!-- more --> 即可 （测试发现本主题无法使用）
  ```

6. 更换hexo默认markdown （此项目已更换，package.json里已配置好了）

   [教程](https://blog.csdn.net/qq_42951560/article/details/123596899) 

  因为hexo默认markdown渲染器hexo-renderer-marked功能太弱，所以使用其他hexo渲染器替换它

  不需要的插件可以用 # 注释掉，如果hexo d报错说有没有找到的module，可以安装它，如`npm i markdown-it-checkbox`

  ```bash
  #卸载 hexo-renderer-marked
  npm un hexo-renderer-marked --save
  
  #安装 hexo-renderer-markdown-it
  npm i hexo-renderer-markdown-it --save
  
  #配置，在hexo的配置文件_config.yml尾部添加
  # exo-renderer-markdown-it
  markdown:
    preset: "default"
    render:
      html: true
      xhtmlOut: false
      langPrefix: "language-"
      breaks: true
      linkify: true
      typographer: true
      quotes: "“”‘’"
    enable_rules:
    disable_rules:
    plugins:
    # 不需要的插件可以注释掉
      #- markdown-it-abbr
      - markdown-it-cjk-breaks
      - markdown-it-deflist
      - markdown-it-emoji
      #- markdown-it-footnote
      #- markdown-it-ins
      - markdown-it-mark
      - markdown-it-sub
      - markdown-it-sup
      - markdown-it-checkbox
      #- markdown-it-imsize
      #- markdown-it-expandable
      - name: markdown-it-container
        options: success
      - name: markdown-it-container
        options: tips
      - name: markdown-it-container
        options: warning
      - name: markdown-it-container
        options: danger
    anchors:
      level: 2
      collisionSuffix: ""
      permalink: false
      permalinkClass: "header-anchor"
      permalinkSide: "left"
      permalinkSymbol: "¶"
      case: 0
      separator: "-"
  ```







