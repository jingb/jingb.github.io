---
title: openresty请求路由
tags:
---

关于openResty+lua 项目的文件组织和请求路由的问题，我个人接触到有两种方式。

一种是[《openResty最佳实践》](https://moonbingbing.gitbooks.io/openresty-best-practices/content/web/api.html)里介绍的通过nginx的rewrite指令解析请求url再把请求路由到不同的lua文件里，举个栗子如下


    rewrite '^/([a-z]+)/(update|create|delete|query)/?([0-9]*)$' /$1/$2;   
这样比方请求url http://localhost:6699/user/query/11，则会路由到/user/query的location块里，同时通过ngx.var能取到 user、query、和请求实例的id

另一种是在lua文件里路由，接触到的是lor本身的请求路由和[lua-resty-route](https://github.com/bungle/lua-resty-route)这个第三方路由组件

lor的代码就不贴了贴一段我引用[lua-resty-route](https://github.com/bungle/lua-resty-route)的业务处理代码
routes.lua文件如下，业务请求都会跑进来，具体怎么进来这里就不说明了


    local route = require "resty.route".new() 
    local book = require "book" 

    route {"get", "post"} "~^/book/(update|create|delete|query)/?([0-9]*)$" (book.handle) 
    ………… 
    ………… 
    return route 

route是[lua-resty-route](https://github.com/bungle/lua-resty-route)里的对象，book.lua封装了关于book这个model的一些业务操作，handle是个总入口
    
    function _M.handle()
       //代码解析ngx.var.request_uri，可以解析出model、action、instanceId
       //再根据解析结果分发到不同的业务函数里处理
    end

**个人理解**，新手，业余捣鼓捣鼓，有什么理解的不对的麻烦大家不吝指出
* 方案一会造成nginx的配置文件很大，很多location
* 方案二似乎更灵活一些

**问题**  
* lor的做法是第二种，想问各位在实际项目中的二选一，还是会结合起来用
* 据说方案一开销会大一些


