---
layout: post
title: 利用Rstudio创建R包
subtitle: 来源《R语言实战》
bigimg: /pics/20180514/01.jpg
tags: Rstudio, R包, R语言学习笔记
---

&emsp;&emsp;好久没有更新自己的学习笔记了，今天来写一下，如何利用*Rstudio*创建属于自己的*R*包。

&emsp;&emsp;首先，需要了解一下在创建*R*包前的准备工作，这里借用一下**R语言实战（第2版）**中的一张图加以说明，如下图。

![](/pics/20180514/02.png)

我们需要基于这样的一个文件夹及文件的存储结构来创建*R*包。图中展示的是`npar`包在创建之初时的文件夹及文件的存储结构。所以，在创建R包之前，我们首先需要编辑好函数存放在*.R*的文件中，然后创建函数的帮助文档，并以*.Rd*的格式保存，此外如果有数据集，需要先保存为*.rda*文件。比如一个`life`数据集保存为*rda*文件的命令是

```r
save(life, file = 'life.rda')
```

&emsp;&emsp;然后，在写好了函数及数据集之后，为了方便生成相应的*.Rd*文件，可以直接在相应的*.R*文件开头加入帮助文档的描述，然后利用`roxygen2`包直接生成相应的*.Rd*文件。例如，以下是`print.R`开头部分的内容以及`life.R`文件的内容。

![](/pics/20180514/03.png)

![](/pics/20180514/04.png)

其中注释使用的一些标签和含义见下表。

![](/pics/20180514/05.png)

&emsp;&emsp;接下来，在写好了所有的*.R*文件之后，利用*Rstudio*创建一个*project*，作为生成*R*包的工作目录。打开*Rstudio*后依次点击 *File -> New Project -> New Directory -> R Package*，就会弹出如下的对话框。

![](/pics/20180514/06.png)

填写包的名字，并且添加需要包含的文件（包括*.R*文件和*.rda*文件），最后点击*Create Project*就完成初步的制包准备了，这时候你会进入到相应的*project*的工作目录。相关的文件夹，如*man*，*data*，*R*已经被创建了。

&emsp;&emsp;那么，接下来只需要做以下几件事，*R*包就能成功被创建。

+ 在*man*文件夹下生成*.Rd*文件。
+ 编辑整个包的*DESCRIPTION*文件。
+ 安装*R*包，生成*pdf*格式的帮助文档，以及*.tar.gz*格式的安装包。

&emsp;&emsp;以`npar`包为例，将当前的工作空间设置为刚刚新建的*project*所在的文件夹。

&emsp;&emsp;生成*.Rd*文件可以使用以下的命令。

```r
roxygen2::roxygenize('npar')
```

&emsp;&emsp;编辑*DESCRIPTION*文件时，可以在*project*当中直接点开并编辑，以下是编辑的范例。

![](/pics/20180514/07.png)

&emsp;&emsp;最后运行如下代码生成*pdf*格式的关于所有函数的帮助文档（注意当前的工作空间）。

```r
system('R CMD Rd2pdf npar')
```

&emsp;&emsp;打开*project*，点击*Build*标签下的*Install and Restart*，就能生成并安装自己的*R*包了。同时，点击*Build Source Package*可以生成*.tar.gz*格式的安装包。

&emsp;&emsp;这样，*R*包就制作安装成功了。