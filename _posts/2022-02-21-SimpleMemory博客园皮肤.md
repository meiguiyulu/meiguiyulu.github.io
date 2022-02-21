---
layout: post
title: "SimpleMemory博客园皮肤"
date: 2022-02-21 09:00:00 +0800 
categories: 杂谈
tag: Blog
---
* content
{:toc}

# SimpleMemory博客园皮肤

## 1. 声明

这个主题是一位大佬做的。

大佬博客园地址：https://github.com/BNDong/Cnblogs-Theme-SimpleMemory。

大佬Github地址：https://www.cnblogs.com/BNDong。

## 2. 安装

1. 申请JS权限

   ![image-20220221124324269](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220221124324269.png)

2. 博客皮肤选择SimpleMemory

   ![image-20220221124416435](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220221124416435.png)

3. 在GitHub上下载源码

   ![image-20220221124558912](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220221124558912.png)

4. 设置页面定制CSS代码，同时禁用模板默认CSS

   将`/dist/simpleMemory.css` 拷贝此文件代码至页面定制CSS代码文本框处。

   ![image-20220221124803064](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220221124803064.png)

5. 设置博客侧边栏

   输入一下代码：

   ```html
   <script type="text/javascript">
       window.cnblogsConfig = {
         info: {
           name: 'userName', // 用户名
           startDate: '2021-01-01', // 入园时间，年-月-日。入园时间查看方法：鼠标停留园龄时间上，会显示入园时间
           avatar: 'http://xxxx.png', // 用户头像
         },
       }
   </script>
   <script src="https://cdn.jsdelivr.net/gh/BNDong/Cnblogs-Theme-SimpleMemory@v2.1.0/dist/simpleMemory.js" defer></script>
   ```

   > 更多更详细的设置可见https://bndong.github.io/Cnblogs-Theme-SimpleMemory/v2/#/Docs/Customization/config?id=%e7%a4%ba%e4%be%8b%e9%85%8d%e7%bd%ae。

6. 设置页脚HTML代码

   ```html
   <!--代码复制-->
   <script src="https://cdn.bootcss.com/clipboard.js/2.0.4/clipboard.min.js"></script>
   <!--主题-->
   <script src="https://blog-static.cnblogs.com/files/gshang/gshang.bilibili.big.2020.02.27.4.js" ></script>
   <!--scrollTo-->
   <script src="https://cdn.bootcss.com/jquery-scrollTo/2.1.2/jquery.scrollTo.js"></script>
   <!--owo表情-->
   <script src="https://blog-static.cnblogs.com/files/gshang/gshang.owo.2020.01.05.1.js"></script>
   <link rel="stylesheet" href="https://blog-static.cnblogs.com/files/gshang/gshang.OwO.3.css" />
   <!-- import Vue.js -->
   <script src="https://cdn.staticfile.org/vue/2.2.2/vue.min.js"></script>
   <!-- 引入样式 -->
   <link rel="stylesheet" href="https://blog-static.cnblogs.com/files/gshang/notiflix-2.0.0.min.css">
   <!-- 引入组件库 -->
   <script src="https://blog-static.cnblogs.com/files/gshang/notiflix-2.0.0.min.js"></script>
   ```

7. 勾选公告

   ![image-20220221125128312](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220221125128312.png)

8. 成功

   ![image-20220221125156697](https://gitee.com/yun-xiaojie/blog-image/raw/master/img/image-20220221125156697.png)

   

   

