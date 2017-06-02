---
layout: post
title: "Git常用命令"
description: ""
category: 技术分享
tags: [blog]
---
{% include JB/setup %}
# Git常用命令
---

收集下git常用命令方便以后查看

<!--break-->

|命令|说明|
|----|----|
|git log -l|查看提交历史 -l 后边为显示条数|
|git log --pretty=oneline|查看提交历史 单行显示|
|git reset --hard HEAD^|将当前工作空间内容重置为前一个版本|
|git reset --hard 3628164 |将当前工作空间内容重置为commit id为3268164这个版本|
|git reflog|查看所有提交记录，包括相对于当前工作空间过去和未来|
|git status|当前工作空间状态|
|git diff|是工作区(work dict)和暂存区(stage)的比较|
|git diff --cached|是暂存区(stage)和分支(master)的比较|
|git checkout -- readme.txt|让这个文件回到最近一次git commit或git add时的状态|
|git reset HEAD readme.txt|丢掉暂存区修改|
|git rm test.txt   git commit -m "remove test.txt"|删除文件|
|git remote add gittest(别名) git@github.com:michaelliao/learngit.git|添加远程库|
|git branch dev|创建分支|
|git checkout dev|切换分支|
|git branch|查看分支|
|git branch -d dev | 删除分支|
|git log --graph --pretty=oneline --abbrev-commit | 查看合并分支情况|
|git merge --no-ff -m "merge with no-ff" dev | 禁用fast forword合并|
|git stash | 保存当前工作区|
|git stash list | 查看保存的工作区|
|git stash apply | 恢复工作区|
|git stash drop | 删除工作区|
|git stash pop | 恢复并删除工作区|
|git stash apply stash@{0} | 恢复指定的暂存工作区|
|git branch -D feature-vulcan | 强行删除分支|
|git checkout -b dev origin/dev | 创建本地dev分支|
|git tag v1.0 | 打标签|
|git tag | 查看标签|
|git tag v0.9 6224937 | 对commit id 打标签|
|git show v0.9 | 查看某个标签的具体信息|
|git tag -a v0.1 -m "version 0.1 released" 3628164 | v0.1： 标签名  -m '说明内容'|
|git tag -d v0.1 | 删除本地标签|
|git push origin v1.0 | 推送标签到远程|
|git push origin --tags | 推送所有未推送的标签|
|git tag -d v0.9       git push origin :refs/tags/v0.9  | 删除远程标签|
