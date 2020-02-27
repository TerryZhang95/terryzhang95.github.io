# Hugo 个人静态网页推荐


# Hugo 个人主页笔记

## 背景
- 为什么选择静态网页？
  
  因为可以配合github page轻松部署，并且因为贫穷，无力支撑服务器费用

- 为什么选择hugo？
  
  主流的静态网页，试过Jekyll，Hexo以及wordpress，各有千秋，但是：
  - Jekyll的文档真的复杂，gem安装依赖和ruby的语法，非常影响拓展，但是插件还算丰富，是github亲儿子，应该兼容最好


## 踏平的坑

- 为什么执行hugo指令，build不成功？如何debug？ 
  - 执行本地server测试看看
  - hugo运行在localhost:1313，不成功 
  - 是时间问题！！！
  - 超过当前时间都不会显示
  - 统一为格林威治时间
  - 解决方式
  - hugo new 创建文章和配置
  - hugo --buildFuture
  
