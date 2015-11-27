# DavidDay's Blog

### [View Blog →](http://daiweilai.github.io)

## Document

#### Environment

If you have jekyll installed, simply run `jekyll serve` in Command Line

#### Get Started

You can easily get started by modifying `_config.yml`:

``` 
# Site settings
title: DavidDay’s Blog
SEOTitle: 戴伟来的博客|DavidDay's Blog
header-img: img/common/home-bg.jpg
email: daiweilai@foxmail.com
description: "在这里探索iOS的一切。"
keyword: ""
url: "http://daiweilai.github.io"              # your host, for absolute URL
baseurl: ""                             # for example, '/blog' if your blog hosted on 'host/blog'



# SNS settings
RSS: true
github_username:    daiweilai
#weibo_username:     daiweilai
#zhihu_username:     daiweilai
#twitter_username:  daiweilai
#facebook_username:  daiweilai


# Build settings
gems: [jekyll-paginate]
#highlighter: pygments
permalink: pretty
paginate: 10
exclude: ["less","node_modules","Gruntfile.js","package.json","README.md"]
anchorjs: true                          # if you want to customize anchor. check out line:181 of `post.html`



# Markdown settings
# replace redcarpet to kramdown,
# although redcarpet can auto highlight code, the lack of header-id make the catalog impossible, so I switch to kramdown
# document: http://jekyllrb.com/docs/configuration/#kramdown
markdown: kramdown
kramdown:
  input: GFM                            # use Github Flavored Markdown !important



# Duoshuo settings
duoshuo_username: XXX # Share component is depend on Comment so we can NOT use share only.
duoshuo_share: true                     # set to false if you want to use Comment without Sharing



# Analytics settings
# Baidu Analytics
#ba_track_id: xxx
# Google Analytics
#ga_track_id: 'xxx'            # Format: UA-xxxxxx-xx
#ga_domain: xxx



# Sidebar settings
sidebar: true                           # whether or not using Sidebar.
sidebar-about-description: "正在努力探索iOS开发的最佳实践。"
sidebar-avatar: http://daiweilai.github.io/img/common/avatar.jpg      # use absolute URL, seeing it's used in both `/` and `/about/`



# Featured Tags
featured-tags: true                     # whether or not using Feature-Tags
featured-condition-size: 0             # A tag will be featured if the size of it is more than this condition value



# Friends
friends: [
    {
        title: "",
        href: ""
    },{
        title: "",
        href: ""
    }
]
```





#### Write Posts

Feel free to checkout Markdown files in the `_posts/`, you will quickly realized how to post your articles with magical markdown plus this nice theme.

The **front-matter** of a post looks like that:

``` 
---
layout:     post
title:      "半年的iOS代码生活"
subtitle:   "加班的领悟"
date:       2015-11-11 12:00:00
author:     "DavidDay"
header-img: "img/post/2015-11-11-bg.jpg"
tags:
    - 闲暇集
---

contents...
```

#### Code hightlight

Using [Prism.js](http://prismjs.com/),you can simply change highlight style by modifying or replacing `js/prism.js` and `css/prism.css`



#### SEO Title

Before V1.4, site setting `title` is not only used for displayed in Home Page and Navbar, but also used to generate the `<title>` in HTML.

It's possible that you want the two things different. For me, my site-title is **“DavidDay's Blog”** but I want the title shows in search engine is **“戴伟来的博客|DavidDay's Blog”** which is multi-language.

So, the SEO Title is introduced to solve this problem, you can set `SEOTitle` different from `title`, and it would be only used to generate HTML `<title>` and setting DuoShuo Sharing.

## Thanks

This theme is forked from  [Hux blog](https://github.com/Huxpro/huxpro.github.io)

Thanks Jekyll and Github Pages!



## LICENSE

``` 
The MIT License (MIT)

Copyright (c) 2015 戴伟来

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

