1. clone代码

	``` bash
	git clone git@github.com:mingever/blog.git
	```

2. 切换分支

	``` bash
	git checkout dev
	```

4. 安装node依赖

	``` bash
	npm install
	#package.json里依赖已约定号了，主要是hexo和hexo-deployer-git这两个
	```

5. hexo

	``` bash
	hexo clean
	hexo g
	hexo d
	```

	