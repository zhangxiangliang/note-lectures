# Typecho 那些事
## 背景
* Wordpress 未成气候
* PHP和开源兴起
* 每个人只能有 5M 的虚拟空间
* 没有数据库

* 写 bo-blog 皮肤
* 参与 exblog 开源团队
* 自己开发 Magike
* 和小伙伴一起开发 Typecho

## 团队
* 通过网络认识
* 邮件列表 + SVN(GIT)
* 在 WordCamp 面基

## 特色
### 设计思想
* 简单
* 所想即所得

### 为什么不MVC
* 不利于扩展（耦合性）
* 使用 MVC 会使得增加扩展难度，需要修改和了解多个层次
* 框架太庞大（瘦身）
* 使用 MVC 太庞大，需要写很多代码来沟通和传递消息
* MVC 不利于理解，需要文档才利于理解

### Widget
* 变种的 Controller 和 Model
* 兼具部分 View
* 可以减少分层，只需要些一个 Widget
* 减少分层，保持独立

## Typeho
### 常量
* __TYPECHO_DEBUG__
* __TYPECHO_ADMIN_DIR__
* __TYPECHO_UPLOAD_URL__
* __TYPECHO_UPLOAD_DIR__
* __TYPECHO_SECURE__
* __TYPECHO_GRAVATAR_PREFIX__


### 皮肤
* $this->header()
* 使用 $this->need('xxx') 而不是 require 或者 include
* $this->footer()
* 自定义归档皮肤
    *  default/categroy/girl.php
    *  default/post/{id}.php
* 为皮肤定制自定义变量

### 插件
* Widget 全局钩子，让主题更原生
* DB 延迟绑定
* 备份插件
    * 第三方插件备份是直接利用 SQL 备份，导致不能跨平台。
    * 官方插件使用二进制码转换来保证跨平台。
    * 第三方插件大文件备份会占用大量内存资源。
    * 官方插件写入二进制流到缓冲区，缓冲区快满的时候进行吸入文件。
