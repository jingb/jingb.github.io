---
title: go
tags:
---

[知乎链接](https://www.zhihu.com/question/30461290)

引用类型有 切片、map、接口、函数类型以及chan
函数是指不属于任何结构体、类型的方法，和方法的区别
方法的 接受者分 值接收者和指针接收者，区别是什么

```func add(a, b int) int```     只能在同包下使用，Add则能给外面的使用
```func (p person) String() string```  方法

[项目依赖详解，gopath、goroot、vendor、godep解决什么问题](https://ieevee.com/tech/2017/07/10/go-import.html)

# k8s 
## CRD
实际应用中，除了控制循环之外的所有代码，实际上都是 Kubernetes 为你自动生成的，即：pkg/client/{informers, listers, clientset}里的内容
