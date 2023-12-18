title:: Client-Go 中的 Informer 源码分析
author:: [[宋净超（Jimmy Song）]]
url:: https://jimmysong.io/kubernetes-handbook/develop/client-go-informer-sourcecode-analyse.html
tags:: #[[k8s]] #[[Readwise]]  
![](https://readwise-assets.s3.amazonaws.com/static/images/article0.00998d930354.png)

- > defaultResync: 用于初始化持有的 shareIndexInformer 的 resyncCheckPeriod 和 defaultEventHandlerResyncPeriod 字段，用于定时的将 local store 同步到 deltaFIFO ([View Highlight](https://read.readwise.io/read/01hhxt8m4esed4c8tcj1htfh2d))