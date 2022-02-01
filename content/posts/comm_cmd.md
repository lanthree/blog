---
title: "常用命令"
date: 2022-02-02T22:13:53+08:00
draft: false
categories: []
---

## rebase master

开发分支从较旧的master拉出，需要合并master的代码修改

1. 更新master
    ```sh
    git checkout master
    git pull
    ```
2. 切换到你的开发分支
    ```sh
    git checkout $YOUR_WORK_BRANCH 
    ```
3. rebase操作
    ```sh
    git rebase master
    
    # 若有冲突 解决冲突 然后
    git rebase --continue
    ```

## 合并开发分支的多个Commit记录

假设合并最近4次commit

1. rebase self
    ```sh
    git rebase -i HEAD~4
    ```
2. git会罗列最近4次commit的记录，例如
    ```
    pick u8sa9du
    pick 19id9is
    pick fudhf82
    pick asdj187
    ```
    从第二个开始改成s
    ```
    pick u8sa9du
    s 19id9is
    s fudhf82
    s asdj187
    ```
    保存退出
3. git自动打开COMMIT_EDITMSG，修改并保存合并后的commit msg

## git log


```sh
[alias]
    lg = log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr)%Creset' --abbrev-commit --date=relative
```

## git 中文路径乱码

```
git config core.quotepath false
```

## tig

```sh
brew install tig
```
