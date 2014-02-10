---
layout: post
title: "豆瓣喜爱文章下载器"
date: 2013-06-12 23:31
comments: true
categories: Programe 
---
收藏一些不错的文章

在豆瓣阅读到一些好的文章，通常会点击加心喜爱。久而久之，就淡忘了，于是想把他们下载下来，用了一天的时间写了具有爬虫和分析功能的下载器。使用了BeautifulSoup,urllib模块.可以一次性抓取其他人的文章。

使用方法：

```python
if __name__ == "__main__":
    #用户名,可以写入douban ID
    usrnames = ["laiyonghao","fenng"]
        for name in usrnames:
                cwl = Crawler(name)
        cwl.start()
        parser = Parser(cwl)
        parser.run()
```
>运行 python main.py

结果 

```
tmp
    ├── fenng
    │   ├── Got talent?
    │   ├── QCon Beijing 2013 乱记
    │   ├── 北京生存指南
    │   ├── 发现「讷客」
    │   ├── 父亲谈创新
    │   ├── 和菜头对豆瓣用户骂声的回应
    │   ├── 胡扯两句
    │   ├── 年会：公司人的新民俗
    │   ├── 一个关于嘲笑的小故事
    │   ├── 有些事，蹲在家里是永远实现不了的。
    │   ├── 赞
    │   └── 中国说唱教父--尹相杰&lt;转&gt;
    └── laiyonghao
        ├── Live Together, Hate Each Other
        ├── 当初的愿望实现了么？
        ├── 豆瓣阅读即将开售（已上线）
        ├── 婚礼主题曲背后的故事
        └── 科普也需严谨---对《数学之美》密码部分的评论
```

源代码[链接](https://github.com/zheng-ji/douban-likes-crawler.git)
Happy Hacking : )


