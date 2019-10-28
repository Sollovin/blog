---
title: "电子书的命名与归类"
date: 2018-08-29 22:02:58
updated: 2019-02-26
draft: false
toc: false
images:
tags: 
  - untagged
markup: pandoc
---

现在我们肯定会在电脑上存储各种各样的电子资源，为这些资源进行有逻辑的命名与归类肯定有助于我们在需要的时候进行查找，下面我以电子书为例，谈谈我的思路，仅供参考。
<!-- more -->

------

# 命名
电子书名称的命名按照如下规则：
```
[丛书名]作者_书名-副书名[上下册](版别)_特殊
```
说具体一点:

-   `作者` 一项只填第一作者或最有名的作者，如果实在需要，多名作者之间用英文逗号和空格即 `, ` 进行分隔，另外外国人的姓名用其 lastname 就行，因为 firstname 极易重名；
-   `上下册`一项可能的命名有 `上册`, `上中册`, `上中下册`, `第1部`, `第1-3部`, `卷一`, `卷一二`, `卷一至五`, `VOL I`, `VOL II` 等;
-   `版别` 一项中文用类似 `第一版` 或 `第1版`，英文用类似 `2nd ed`;
-   `特殊` 一项用于标注一些额外的信息，比如这个电子书仅仅是某本书的目录，则相应填入 `目录` 或 `TOC`

这样，具体的命名实例如下：
```
Cohen_Quantum Mechanics_TOC.pdf
Cohen_Quantum Mechanics[VOL II].pdf
Cohen_Quantum Mechanics[VOL I].pdf
Griffiths_Introduction to Quantum Mechanics(2nd ed).pdf
Griffiths_Introduction to Quantum Mechanics(2nd ed)_Solutions.pdf
Nazarov, Blanter_Quantum Transport-Introduction to Nanoscience.pdf
袁松柳_固体物理_讲义.pdf
鞠国兴_热物理概念-热力学与统计物理(第2版).pdf
Kaku_超越时空-通过平行宇宙、时间卷曲和第十维度的科学之旅
[理论物理基础系列教程 3]長岡洋介_电磁学[上册]
Zinsser_On Writing Well-The Classic Guide to Writing Nonfiction(30th Anniversary Edition)
Strunk_The Elements of Style(2011 Revised Edition)
```

可能会碰到附带有音频、图像或视频等资源的情形，那么就应该单独为该书建立一个文件夹，文件夹的命名与书籍的命名一致，再建立子目录 `附件` 存放附带的资源，例如：
```
----赖世雄_美版音标_扫描版
    |--赖世雄_美版音标_扫描版.pdf
    |--附件
        |--p.002-p.003.mp3
        |--p.004.mp3
        |--p.005-p.006.mp3
           ...
```

也可能碰到一本书含多个版本或同属一个系列的情况，那么也应为该书建立独立的文件夹，如:
```
----南派三叔_盗墓笔记[第一二季]
    |--南派三叔_盗墓笔记[第一季].pdf
    |--南派三叔_盗墓笔记[第二季].pdf
        ...
```

# 归类
To be cotinued...

# 参考
-   [统一的电子书命名规则|dazhuzai8](!https://dazhuzai8.blogspot.com/2012/09/3.html)
-   [电子图书命名规范 - 简书](!https://www.jianshu.com/p/b82231b06881)
