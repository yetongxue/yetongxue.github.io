### **一、markdown常用语法**


https://www.jianshu.com/p/88bd19ceecc7

---
### **二、文件/文件夹说明:** ##


_config.yml:配置文件

	1、public:生成的静态文件，这个目录最终会发布到服务器
	2、scaffolds:一些通用的markdown模板
	3、source:编写的markdown文件
		3.1 _drafts 草稿文件
		3.2 _posts 发布的文章
	4、themes	博客的模板


---
### **三、备注：**

	我们正常使用，修改最多的源码是_config.yml文件，不管是博客的基础配置，还是模板，都是修改这个文件。
	source是我们日常写文章要用的目录，是我们日常操作的文件夹。如果针对下载的模板修改，那么就需要操作themes了。
	hexo是用node.js编写的程序，所以theme的修改也是比较容易的。

---
###hexo 发布常用命令
	hexo n "我的博客" == hexo new "我的博客" #新建文章
	hexo g == hexo generate#生成	
	hexo s == hexo server #启动服务预览
	hexo d == hexo deploy#部署
	每次发布时，使用 hexo clean && hexo g && hexo d




