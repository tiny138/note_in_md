<!--
 * @Author: your name
 * @Date: 2021-01-23 21:18:15
 * @LastEditTime: 2021-01-23 21:24:13
 * @LastEditors: Please set LastEditors
 * @Description: In User Settings Edit
 * @FilePath: \markdown\Git基本命令.md
-->
# Git 基本命令
## 官方
create a new repository on the command line

    echo "# -" >> README.md
    git init
    git add README.md
    git commit -m "first commit"
    git branch -M main
    git remote add origin https://github.com/tiny138/-.git
    git push -u origin main

push an existing repository from the command line  

    git remote add origin https://github.com/tiny138/-.git
    git branch -M main
    git push -u origin main

import code from another repository

## 自己摸索的

1. Git bash here 

2. 代码

        git init    
        git add .
        git commit -m "描述"
        git branch -M master
        git remote add origin https://github.com/tiny138/-.git
        git push -u origin master