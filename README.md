# 宝塔面板下使用 Nginx_Pagespeed给网站前端加速

## 引子:
  这几天遇到一个网站优化问题，老板需求是给网站提速，但是我司网站后端采用的thinkphp3.1.1版本，并不支持任何缓存插件，所以php缓存可以排除。
        
  想了想，想起来之前了解过的一个Nginx模块: PageSpeed,可以给给网站的前端进行加速,尤其适用于我司网站这种首页存在大量图片及沉余请求的情况.
        
  参考了ngx_pagespeed官方提供的安装文档后，发现提供的自动安装脚本不适用于LNMP面板，宝塔面板，OneInStack等一键安装的LNMP环境。由于我司网站是通过宝塔面板进行搭建维护的，一开始很担心搞不定(怕重新编译后这些一键环境无法控制Nginx)，经过摸索后竟然发现，在宝塔上安装这个脚本比在LNMP面板上安装还要简单！

## Nginx_PageSpeed模块介绍
  ngx_pagespeed 是谷歌开发的一个Nginx扩展模块，其实这个模块很早就开发出来了，但之前总有bug，一直处于beta版本，16年(疑)才有stable版本。
    
ngx_pagespeed官网： [http://ngxpagespeed.com/](http://ngxpagespeed.com/)
项目Github主页： [https://github.com/pagespeed/ngx_pagespeed](https://github.com/pagespeed/ngx_pagespeed)
GoogleDevelopers：[https://developers.google.com/speed/docs/mod_pagespeed](https://developers.google.com/speed/docs/mod_pagespeed/build_ngx_pagespeed_from_source)

### 主要功能

`*`图像优化：剥离元数据、动态调整，重新压缩
`*`CSS和JavaScript压缩、合并、级联、内联
`*`小资源内联
`*`推迟图像和JavaScript加载
`*`对HTML重写、压缩空格、去除注释等
`*`提升缓存周期
`*`and so on...


## 模块的安装:

### 实验环境

**操作系统:** Debian8 64位
**管理面板:** 宝塔5.9免费版
**环境: **nginx 1.14 编译安装

### 安装流程
`*`**一.安装前准备**
  * 检查当前状态: 系统 & root权限 & GCC版本(>=4.8)
  * 更新yum/apt源 & 安装pagespeed所需依赖
  * 增加swap空间 （很多小内存VPS会出现内存不足导致的编译崩溃）

`*`**二.下载模块**
  * 下载ngx_pagespeed模块
  * 下载psol(模块优化管理)

`*`**三.安装模块**
  * 获取当前nginx配置文件
  * 在配置中添加ngx_pagespeed模块
  * make & make install

`*`**四.验证模块**
  * 重启nginx(重载配置不行,必需重启)
  * 验证模块是否运行

基于以上手动安装过程，我写了个**自动化安装的shell**，可以通过一行命令进行安装:

```shell
 wget https://raw.githubusercontent.com/madlifer/ngx_pagespeed_auto/master/nps-auto.sh && bash nps-auto.sh
```

## 模块的配置
  由于采用的nginx版本为1.14，貌似不支持动态模块，所以不需要写入动态模块。直接在网站的nginx配置文档中 server段内粘贴所需要的功能命令，然后重载Nginx配置就可以了。

```TXT
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

```
