---
title: nginx
tags:
---
# nginx

## include
包含外部文件

## directive blocks
directive的作用域问题

## Module variables
### 种类
#### 代表请求头来的信息
#### 代表响应头的信息
#### 完全由nginx生成的信息

## location [=|~|~*|^~|@] pattern { ... }
> (location =) > (location 完整路径) > (location ^~ 路径) > (location ~,~\* 正则顺序) > (location 部分起始路径) > (/)

## Module Configuration

## agentzh教程纪要

<span id="agentzh var"></span>
### 变量漫谈 
> * Nginx 变量一旦创建，其变量名的可见范围就是整个 Nginx 配置，甚至可以跨越不同虚拟主机的 server 配置块。**可见范围虽然是整个配置，但每个请求都有所有变量的独立副本，或者说都有各变量用来存放值的容器的独立副本，彼此互不干扰。**
* 变量容器的生命期，不是与 location 配置块绑定的，而是与请求绑定的。比方从locationA内部跳转到locationB，变量值依然是locationA中的值
* ngx_set_misc这个第三方模块可以提供解析url中类似%20这样编码的序列
* $arg_XXX、$cookie_XXX、$http_XXX(读请求头)……这些功能均由ngx_http_core提供
* 子请求和父请求的变量各自独立，但有一些第三方模块如ngx_auth_request会共享子、父请求间的变量
* 并非所有的内建变量都作用于当前请求，少数内建变量只作用于“主请求”。比方$uri在子、父请求间可以正确输出，但像$request_method这样的内建变量，如果父请求是post，在子请求里输出的时候还是post，哪怕是用get的方式请求到子请求的时候
* ngx_lua模块，ngx.var.arg_name可以直接读nginx的变量$arg_name，ngx.var.VARIABLE可以直接读Nginx 变量 $VARIABLE

### 指令执行顺序
> * rewrite、access、content三个常用阶段，所有 Nginx 模块提供的配置指令一般只会注册并运行在其中的某一个处理阶段
  ```
  location /test {
      set $a 32;
      echo $a;

      set $a 56;
      echo $a;
  }
  输出结果是两行56，因为set在rewrite阶段执行而echo在content阶段执行，
  而rewrite阶段先于content阶段执行，
  所以实际相当于执行了set $a 32;set $a 56;echo $a;echo $a;
  ```

## rewrite
> http://website.com/article-1234-32-US-economy-strengthens.html类似这样
语义明晰的url比带参数的url对用户更友好，**也对seo更友好**


# OpenResty
## 子请求
* ngx.location.capture
 * option可选参数，像变量传递，可见性等
* ngx.location.capture_multi
* exec


## 访问数据库
> * lua-resty-mysql 库更加灵活，功能也更丰富（比如有多结果集查询和事务的支持），同时在需要对 Lua 对结果进行加工处理的场合，效率更高。对于比较简单的场景，即不需要事务和存储过程调用，同时不需要用 Lua 对结果进行加工处理的场景，ngx_drizzle 模块更轻便，效率更高。

## 缓存、变量的共享范围
### Lua shared dict
* Nginx 所有 worker 之间共享的
* 由于 ngx shared dict 是 workers 之间共享的，所以在多 worker 的情况下，内存占用比较少

### Lua LRU cache
* worker 级别的，不会在 Nginx wokers 之间共享
* 提供的 API 比较少，现在只有 get、set 和 delete

### ngx.ctx
* This table can be used to store per-request Lua context data and has a life time identical to the current request (as with the Nginx variables).
* The ngx.ctx lookup requires relatively expensive metamethod calls and it is much slower than explicitly passing per-request data along by your own function arguments. So do not abuse this API for saving your own function arguments because it usually has quite some performance impact.

## 执行阶段
## json库
### cjson
  目前没Windows版，dkjson效率没cjson好，但是是纯lua，可以跨平台
### dkjson

## 测压
https://github.com/iresty/Mio 

## ngx.var与 ngx.ctx 区别
* ngx.var可跨location共享，即父请求使用ngx.exec到另外一个location依然可以使用
[agentzh变量漫谈](#agentzh var)
* ngx.ctx 比 ngx.var 性能要好很多


# lua
## metatable
元表 (metatable) 的表现行为类似于 C++ 语言中的操作符重载，例如我们可以重载 "\__add" 元方法 (metamethod)，来计算两个 Lua 数组的并集；或者重载 "\__index" 方法，来定义我们自己的 Hash 函数

## lua web framework
两个主要的框架，主要用于快速web开发
### lor
* html的渲染用的还是lua-resty-template

### Lapis 








