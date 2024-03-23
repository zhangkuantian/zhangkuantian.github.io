---
title: 添加顶部公告栏
# 隐藏的模块
hide:
  #  - navigation # 隐藏左边导航
  #  - toc #隐藏右边导航
  #  - footer #隐藏翻页
  #  - feedback  #隐藏反馈
tags:
  - Mkdocs
comments: false  #评论，默认不开启
---
![image.png](https://s2.loli.net/2024/02/02/mvCEgeP4lANuXI8.png)

docs/overrides下新建main.html ，针对main.html文件    
树状结构如下:  
```
$ tree -a
.
├── .github
│   ├── .DS_Store
│   └── workflows
│       └── PublishMySite.yml
├── docs
│   └── index.md
│   └──overrides
│       └──assets
│       └──main.html
│       └──partials
│          └──comments.html
│
└── mkdocs.yml
```

自行修改即可
