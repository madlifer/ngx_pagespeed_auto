# 宝塔面板下使用 Nginx_Pagespeed给网站前端加速

##引子:
	这几天遇到一个网站优化小难题,老板说首页加载速度太慢了想给网站提提速. 我第一反应就是加个缓存插件，但是公司后端采用的TP3.1.1明确不支持任何缓存。
	所以只能在前端上动一动，想到之前天毅大佬帮我博客装过的ngx_pagespeed模块.但是由于宝塔面板的环境目录与lnmp的不同，当时还折腾了一段时间。
	公司网站也是宝塔的LNMP环境,现在自己搞其实心里挺没底的,不过自己在测试后发现，相比于在军哥的lnmp一键包上安装pagespeed，在宝塔面板上搞更简单。

##Ngx_Pagespeed模块简介
	ngx_pagespeed 是 Nginx 的一个扩展模块，主要的功能是针对前端页面而进行服务器端的优化，对前端设计人员来说，可以省去优化css、js以及图片的过程。
	ngx_pagespeed对nginx自身负载能力的提升基本是看不到的，甚至会因为进行服务器端的优化而使系统增加负载；
	但从减少客户请求数的角度去看，牺牲部分服务器性能还是值得的。 

	ngx_pagespeed模块的主要功能如下：

		图像优化：剥离元数据、动态调整，重新压缩

		CSS和JavaScript压缩、合并、级联、内联

		小资源内联

		推迟图像和JavaScript加载

		对HTML重写、压缩空格、去除注释等

		提升缓存周期

	让我觉得最有用的就是:图像的优化,他可以把你的图片压缩并转为webp格式,对于首页图片较多的网站(比如:我们公司的网站)有较大的优化作用。

##Ngx_PageSpeed模块的安装:
	
	实验环境为:  
		系统: Debian8 64位  管理面板: 宝塔5.9免费版 环境: nginx 1.14 编译安装

	首先要说明的是pagespeed官网提供全自动安装,但并不适合各种一键环境,所以只能采用手动安装。
	这里我自己做了个脚本,在实验环境下通过了编译,可以通过下面一行命令进行调用:
	`wget https://github.com/madlifer/ngx_pagespeed_auto/releases/download/v0.0.1/nps-auto.sh &&bash nps-auto.sh`
###声明:
    脚本来源参考了模块官网,脚本命令参考了Linpx,ZhangGe,shell参考了nanqinlang,但由于自身shell水平不行，脚本里仍然还是 full of trash. 强烈不建议用于生产环境。

##为网站配置该模块

将下面的命令粘贴于 网站-域名-配置文档- 域名下方 并保存 即可启用
```
# 启用ngx_pagespeed    
pagespeed on;    
pagespeed FileCachePath /tmp/cache/ngx_pagespeed_cache;    
# 禁用CoreFilters    
pagespeed RewriteLevel PassThrough;    
# 启用压缩空白过滤器    
pagespeed EnableFilters collapse_whitespace;    
# 启用JavaScript库卸载    
pagespeed EnableFilters canonicalize_javascript_libraries; #谷歌被墙，并不确定这个设置有没有副作用 
# 把多个CSS文件合并成一个CSS文件    
pagespeed EnableFilters combine_css;    
# 把多个JavaScript文件合并成一个JavaScript文件    
pagespeed EnableFilters combine_javascript;    
# 删除带默认属性的标签    
pagespeed EnableFilters elide_attributes;    
# 改善资源的可缓存性    
pagespeed EnableFilters extend_cache;    
# 更换被导入文件的@import，精简CSS文件    
pagespeed EnableFilters flatten_css_imports;    
pagespeed CssFlattenMaxBytes 5120;    
# 延时加载客户端看不见的图片    
pagespeed EnableFilters lazyload_images;    
# 启用JavaScript缩小机制    
pagespeed EnableFilters rewrite_javascript;    
# 启用图片优化机制    
pagespeed EnableFilters rewrite_images;    
# 预解析DNS查询    
pagespeed EnableFilters insert_dns_prefetch;    
# 重写CSS，首先加载渲染页面的CSS规则    
pagespeed EnableFilters prioritize_critical_css; 
# Example 禁止pagespeed 处理/wp-admin/目录(可选配置，可参考使用)
```
