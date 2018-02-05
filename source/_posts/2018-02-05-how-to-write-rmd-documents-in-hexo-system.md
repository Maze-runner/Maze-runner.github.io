---
title: "怎么在hexo博客系统中用Rmarkdown写文章"
author: 王诗翔
date: "2018-02-05 21:07:48"
top: false
categories:
    - 极客R
tags:
    - R
    - 写作
    - Rmd
    - md
    - Rstudio
    - Gist
---

作为一个搞数据分析的，动笔总少不了图和代码，`markdown`对代码本身就有不错的区分和高亮支持，但你永远不可能用`markdown`写出图来啊！这类东东人们称之为静态的。为了解决这类问题，现在有两个非常流行的动态文档工具，一是`Python`中的[`Jupyter Notebook`](http://jupyter.org/),另外就是`Rstudio`公司开发的 [`Rmarkdown`](http://rmarkdown.rstudio.com/rmarkdown_websites.html#overview)了。

<!-- more -->

就流行度来说，`Jupyter Notebook`是胜`Rmarkdown`良多的，前者不仅支持`Python`本身，而且支持`R`，不仅如此，它还扩展到其他一些语言中去了（当然`Rmarkdown`现在也能调用`Python`和`Shell`等语言代码）。但我不知道什么原因，`Jupyter Notebook`导出的`markdown`文档实在丑的很，虽然`Github`很神奇的能够全部进行识别并显示出应有的效果。另外，Notebook用来调用写博客并不方便。`Rmarkdown`直接用来写博客则方便得多，语法跟`markdown`差不多，又能利用谢益辉大大的`knitr`包将其转换为`markdown`文档。

总之，作为一个R爱好者，当然用R搞事情。

谢益辉已经开发了一个叫`blogdown`的包，专门用`Rmarkdown`部署生成博客，支持`hugo`所有的主题。如果读者还没有自己的博客，又使用R，推荐阅读<https://bookdown.org/yihui/blogdown/>创建自己的博客并用`Rmarkdown`发布文章。如果你喜欢使用`hexo`博客系统，并想要使用`Rmarkdown`写文章，下面就是你需要阅读的干货了，当然我这种方法不仅限于用在`hexo`博客系统。

我实际做的事情就是写了两个`R`的函数，可以通过调用的方式创建`Rmarkdown`文档，并利用`knitr`包的`knit`函数将其转换为`markdown`文档。

## 第一步

创建一个`Rmarkdown`文档模板，这样我们可以非常方便地在每次写新文章时生成YAML头信息。

其内容如下：

```
---
title: "Put your title here"
author: 王诗翔
date: "2018-02-05 21:07:48"
top: false
categories: Linux杂烩
tags:
    - Linux
---
```

我们将它保存为`template.Rmd`文件（`Rmarkdown`文件一般以`.Rmd`或者`.rmd`结尾）。


## 第二步

将下面两个函数保存到一个R文件（以`.R`结尾）中：

```R
################
## 用rmd写博客 ##
################

# 作者：王诗翔
# 更新日期：2018-02-05


#>>>>>> new_rmd_post 函数 <<<<<<<<<<
# 写好模板文档后，你可以用这个函数来创建Rmarkdown文档
# 参数说明：
# post_name:     博文名（最好英文，显示不会乱码），比如我写这篇博文用的是
#                how-to-write-rmd-documents-in-hexo-system
#                意思不一定要对，只要能跟其他博文名字有区分就行了
# template_name: 模板名，起template.Rmd最好，因为每次写文章都会用到，
#                这样你创建的时候不用每次都指定模板的名字
# template_path: 模板文档的路径，默认当前工作路径
# post_path:     你想把生成的文档放在哪个路径，默认当前工作路径
new_rmd_post <- function(post_name=NULL,template_name="template.Rmd",
                         template_path=getwd(), post_path=getwd()){
    if(is.null(post_name)){
        stop("A post name must be given!")
    }

    input_file   <- paste(template_path,template_name, sep="/")
    current_time <- Sys.Date()
    out_file     <- paste0(current_time, "-",post_name,".Rmd")
    fl_content   <- readLines(input_file)
    writeLines(fl_content, out_file)
    print("New Rmarkdown post creat successfully!")
}

#>>>>> new_md_post 函数 <<<<<<<<<<
# 你可以用这个函数来将Rmd文档转换为markdown文档
# 需要安装knitr包，命令为 install.packages("knitr")
# 参数说明：
# post_name: 文章文档名，推荐使用 年-月-日-英文名 的方式
# template_name: 模板名，你需要转换的Rmd文档
# template_path: 模板文档的路径，默认当前工作路径
# post_path:     你想把生成的文档放在哪个路径，默认当前工作路径
# time_tag:      时间标签，如果你转换的文档没有年-月-日这种标记，
#                将time_tag设定为TRUE会自动在名字前加上
new_md_post <- function(post_name=NULL,template_name="template.Rmd",template_path=getwd(),
                        post_path=getwd(),time_tag=FALSE){

    if(is.null(post_name)){
        post_name <- gsub(pattern = "^(.*)\\.[Rr]md$", "\\1", x = template_name)
    }

    input_file   <- paste(template_path,template_name, sep="/")
    # retrieve system date
    if(time_tag){
        current_time <- Sys.Date()
        out_file     <- paste0(post_path, "/", current_time, "-",post_name,".md")
    }else{
        out_file     <- paste0(post_path, "/", post_name,".md")
    }

    knitr::knit(input = input_file, output = out_file)
    print("New markdown post creat successfully!")
}

```

我把它保存为`new_post.R`，上述我进行了比较详细的注释，请在使用之前仔细阅读一下。

## 使用

我以现在以`Rmarkdown`写的这篇文章为例，简单讲一下使用。

我推荐在与你`markdown`博文目录同级创建一个`_rmd`目录，你可以将该目录设为一个项目目录，专门用来写`rmarkdown`文档。

或者你每次用`setwd()`函数设定工作目录。

将前两步创建的两个文件扔到该目录。运行R文件：

```R
source("./new_post.R")
```

这样就能在R控制台调用里面的两个函数了。

创建一个`Rmarkdown`文档：

```R
> new_rmd_post("how-to-write-rmd-documents-in-hexo-system")
[1] "New Rmarkdown post creat successfully!"
```

创建的文档名会自动添加年-月-日和后缀。

然后你就可以开始写博客了，写好后将`Rmarkdown`转换为`markdown`文档：


```R
> new_md_post(template_name = "2018-02-05-how-to-write-rmd-documents-in-hexo-system.Rmd", post_path = "../_posts")


processing file: /home/wsx/blog/source/_rmd/2018-02-05-how-to-write-rmd-documents-in-hexo-system.Rmd
  |.................................................................| 100%
   inline R code fragments


output file: ../_posts/2018-02-05-how-to-write-rmd-documents-in-hexo-system.md

[1] "New markdown post creat successfully!"
```

细心的娃娃可以感觉到我单独创建`_rmd`目录并将其设为工作目录的好处：需要键入函数的参数非常少。特别是你固定你自己的写法之后，你将两个函数中的目录路径默认参数全部对应上，再使用`Rstudio`的`TAB`键补全，运行命令简直秒秒种，专心写文章就好啦。


本文没有涉及到画图，从理论上讲是毫无问题的，因为我只是创建了一个快速的文档创建和转换接口。后续我少不了会用到绘图，到时候再讲。