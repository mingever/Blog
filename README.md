1. clone代码

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
	#package.json里依赖已约定好了，主要是hexo和hexo-deployer-git这两个
	
	#这样就仅局部安装hexo包了，可以使用以下两种方式执行 Hexo
	#1.使用 npx 来运行，npx hexo <command>
	#2.将 Hexo 所在的目录下的 node_modules\.bin 添加到环境变量之中即可直接使用 hexo <command>，如D:\Blog\node_modules\.bin
	```

5. hexo  [文档地址](https://hexo.io/zh-cn/index.html)

	``` bash
	hexo clean  # 清除缓存
	hexo g      # 生成静态网页
	hexo d      # 部署到Github
	```

	