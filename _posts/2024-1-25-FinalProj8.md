---
layout: post
title: "毕设记录（八）"
description: ""
date: 2024-1-25
feature_image: images/2024.1.25/0.png
tags: [毕设, Unreal]
---

<!--more-->

## Build From Source Code

### 下载仓库并设置

- Fork UnrealEngine 原仓库
- `git clone --branch 5.3 https://github.com/Zydiii/UnrealEngine`
- 检查远程仓库 `git remote -v`
- 设置上游仓库 `git remote add upstream https://github.com/EpicGames/UnrealEngine.git`
- 检查确认 `git remote -v`，应当看到 origin 和 upstream
  
  ```shell
    origin  https://github.com/Zydiii/UnrealEngine (fetch)
    origin  https://github.com/Zydiii/UnrealEngine (push)
    upstream        https://github.com/EpicGames/UnrealEngine.git (fetch)
    upstream        https://github.com/EpicGames/UnrealEngine.git (push)
  ```

### 更新，保持和上游分支同步 ⭐

❗❗❗ 每天修改前都进行以下操作防止与上游仓库差异过大❗❗❗ 

- `git status` 检查本地是否有修改
- 提交本地修改

  ```shell
    git add -A 或者 git add filename
    git commit -m "your note"
    git push origin "master"
  ```

- `git status` 再次检查确认是否有未提交的修改
- 抓取上游分支的更新 `git fetch upstream`
- 切换分支 `git checkout "5.3"`
- 合并上游分支 `git merge upstream/5.3`
- 提交修改 `git push`




## 小结



## References

- [Github进行fork后如何与原仓库同步](https://github.com/selfteaching/the-craft-of-selfteaching/issues/67)