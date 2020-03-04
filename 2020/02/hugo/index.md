# Hugo 个人静态网页推荐


# Hugo 个人主页笔记

## 背景
- 为什么选择静态网页？<br/>
  
  可以配合github page轻松部署，并且因为贫穷，无力支撑服务器费用

- 为什么选择hugo？
  
  主流的静态网页，试过Jekyll，Hexo以及wordpress，各有千秋，但是：

  - 之前的github一直使用Jekyll，使用RubyGem安装依赖的方式始终不习惯，预设环境难上加难，文档也是很复杂
  - 用过一段时间wordpress，但是后期还是想改用markdown写文章
  - Hexo有大量的中文支持，而且对于JS开发者来说很友好，插件丰富，但是速度慢，发到github page上就更慢了
   
很多人更换静态主页框架，都是因为发现了更喜欢的模板，我也不例外

此外环境配置到部署都很简单，速度快，一次编译不到1s

还有怀揣着对go的学习精神，最后选择了hugo


## 踏平的坑

- hugo编译不报错，但更新不编译在public/中
  - 文章的时间不能超过当前时间
  - hugo server的时间默认是格林威治标准时间，会有时差
  - 解决方法
    ```
    hugo new // 使用hugo new创建新文章和配置来确保时间
    hugo --buildFuture //可编译未来时间
    ```

- hugo每次编译直接复制public/，会导致github page的CNAME不工作
  - 使用域名无法访问，但github.io域名下可以
  - 

  