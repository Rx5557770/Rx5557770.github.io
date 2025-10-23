
---
title: git
published: 2025-10-23
description: git轻松更新你的代码仓库。
image: ''
tags: ['code', 'git']
category: 代码日常
draft: false
lang: zh-CN
---
    ```
# 生成ssh密钥
ssh-keygen -t rsa -b 4096 -C "email@example.com"

# 验证
ssh -T git@github.com

git init
# 设置信息
git config --global user.email "onlynnn0@gmail.com"
git config --global user.name "Rx5557770"


git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin <远程仓库URL>
git push -u origin main
```


| **序号**         | **终端命令**                          | **作用**                                                 |
| -------------- | --------------------------------- | ------------------------------------------------------ |
| **1. 初始化本地仓库** | `git init`                        | 在当前项目文件夹中创建 Git 仓库。                                    |
| **2. 添加文件**    | `git add .`                       | 将所有文件添加到暂存区。                                           |
| **3. 提交到本地**   | `git commit -m "Initial commit"`  | 创建第一个本地版本记录。                                           |
| **4. 关联远程仓库**  | `git remote add origin <远程仓库URL>` | 将本地仓库与 GitHub 上的远程仓库关联起来。                              |
| **5. 推送并设置跟踪** | `git push -u origin main`         | 将本地的 `main` 分支推送到远程 `origin` 仓库，并将其设置为默认跟踪分支（只需第一次执行）。 |
