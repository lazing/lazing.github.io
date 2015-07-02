---
layout: post
title:  "Rails Assets Pipe 配置无效的原因和解决"
date:   2015-07-02 12:06:41
categories: rails assets pipe
---
### 背景说明

因功能需要，需修改 js_compressor 配置，让 assets pipe 编译时不要碾碎变量

````ruby
  config.assets.js_compressor = Uglifier.new(mangle: false)
````

本地开发没有问题，在测试环境不起作用。试图修改配置获得调试信息也没有效果。

### 故障发现

经过反复尝试，发现在编译资源文件时，极为快速。而压缩 js 通常应当耗时数分钟。

判断是部署缓存问题。

### 问题原因和解决

因 assets pipe 编译时，根据资源文件时间戳做缓存。

本案中，资源文件未做修改，配置修改。因此编译时使用了原有缓存，造成配置不起作用。

手工删除缓存重新部署即可

````bash
rm -rf /path/to/project/tmp/cache/assets
````
