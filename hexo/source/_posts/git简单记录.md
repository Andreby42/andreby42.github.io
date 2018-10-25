---
title: git简单记录
date: 2018-10-03 01:12:11
tags: [git]
categories: [git]
---

 git了解下<!--more-->

### git配置

* 用户信息编辑

  如果用了``--global``选项，那么配置文件就是更改的位于用户主目录下，就是全局配置。如果在特定项目中想使用其他的用户名和邮箱地址，只要去掉``--global``就可以了，新的配置保存在当前项目的``./git/config``文件中。

  ```
  git config --global user.name "andy"
  git config --global user.email xxx#gmail.com
  ```

* 查看配置信息

  ```
  git config --list
  ```

  ````
  core.symlinks=false
  core.autocrlf=true
  core.fscache=true
  color.diff=auto
  color.status=auto
  color.branch=auto
  color.interactive=true
  help.format=html
  rebase.autosquash=true
  http.sslcainfo=D:/Tools/Git/mingw64/ssl/certs/ca-bundle.crt
  http.sslbackend=openssl
  diff.astextplain.textconv=astextplain
  filter.lfs.clean=git-lfs clean -- %f
  filter.lfs.smudge=git-lfs smudge -- %f
  filter.lfs.process=git-lfs filter-process
  filter.lfs.required=true
  credential.helper=manager
  user.name=xxxx
  user.email=xxxx@xxx.com
  core.repositoryformatversion=0
  core.filemode=false
  core.bare=false
  core.logallrefupdates=true
  ````

* 查看某个环境变量的设定

  ````
  git config user.name
  ````

* help命令

  ```
  git hepl command
  ```

### git基础

* git创建项目

  * 在新的项目的根目录下执行命令

    ```
    git init
    Initialized empty Git repository in E:/sourcetree/test_1/.git/
    ```

  * 将项目中的文件加入版本控制

    ````
    git add *.java
    git add readme.md
    ````

* git 克隆项目

  ```
  git clone git://xxxxxx  yourNewProjectName
  ```

* 记录每次更新到仓库

  * 查看文件状态

    ```
    git status
    ```

*  **gitignore 文件编辑**

  我们可以在项目的根目录下创建一个gitignore文件来管理

  * **所有空行或者以注释符号 `＃` 开头的行都会被 Git 忽略。**
  * **可以使用标准的 glob 模式匹配。**
  * **匹配模式最后跟反斜杠（`/`）说明要忽略的是目录。**
  * **要忽略指定模式以外的文件或目录，可以在模式前加上惊叹号（`!`）取反。**

  ```
  # 此为注释 – 将被 Git 忽略
  # 忽略所有 .a 结尾的文件
  *.a
  # 但 lib.a 除外
  !lib.a
  # 仅仅忽略项目根目录下的 TODO 文件，不包括 subdir/TODO
  /TODO
  # 忽略 build/ 目录下的所有文件
  build/
  # 会忽略 doc/notes.txt 但不包括 doc/server/arch.txt
  doc/*.txt
  # 忽略 doc/ 目录下所有扩展名为 txt 的文件
  doc/**/*.txt
  ```

* 查看已暂存和未暂存的更新

  * git diff

    git diff 查看当前文件和暂存区的快照区别

    ```
    git diff
    ```

    若要看**已经暂存起来的文件和上次提交时的快照之间的差异**，可以用 `git diff --cached` 命令 

    现在可以用``git diff --staged``来替代--catched

* 提交git commit

  跳过add 将所有的文件都提交的话那么使用``git commit -a``

* 从git中一处某个文件

  从已经加入跟踪文件清单中移除某个文件，其实是从暂存区移除

  使用``git rm``，这样就可以从版本控制中去除，并连带从工作目录中删除指定的文件，以后这个文件就再也不会出现在文件清单中了

  从跟踪清单中去除的话 使用``git rm --cached xxx`` xxx 可以为reg模式

* 移动文件,重命名

  使用 ``git mv``

  ```
  git mv file_from file_to
  ```

* 查看日志

  ````
  git log
  ````

  使用 -p 来显示每次提交的内容差异，用-nums来显示最近几次更新

  ```
  git log -p -3
  ```

  使用 `--word-diff`  来显示具体的内容的差异变化历史

  ```
  git log -p -3 --word--diff
  ```

* 撤销操作

  如果有漏提交的文件，那么使用以下命令进行重新提交

  ```
  git commit --amend
  ```

* 取消已经暂存的文件

  如果有暂存的文件不想暂存了

  ```
  git reset HEAD xxxx
  ```

* 取消对文件的修改

  ```
   git checkout [<options>] <branch>
   or
   git checkout [<options>] [<branch>] -- <file>...
  ```

* 远程仓库操作

  ```
  git remote
  ```

  查看具体的克隆版本

  ```
  git remote -v
  ```

  抓取远程仓库有但是本地仓库没有的

  ```
  git fetch <branch>
  ```

* 推送数据到远程仓库

  ```
  git push [--all | --mirror | --tags] [--follow-tags] [--atomic] [-n | --dry-run] [--receive-pack=<git-receive-pack>]
  	   [--repo=<repository>] [-f | --force] [-d | --delete] [--prune] [-v | --verbose]
  	   [-u | --set-upstream] [--push-option=<string>]
  	   [--[no-]signed|--signed=(true|false|if-asked)]
  	   [--force-with-lease[=<refname>[:<expect>]]]
  	   [--no-verify] [<repository> [<refspec>…]]
  ```

* 查看远程仓库信息

  ```
  git remote show branch
  ```

* 远程仓库的删除和重命名

  重命名

  ```
  git remote rename <old> <new>
  ```

  删除

  ```
  git remote remove <name>
  ```

* 打标签

  ```
  git tag [-a | -s | -u <keyid>] [-f] [-m <msg> | -F <file>] [-e]
  	<tagname> [<commit> | <object>]
  git tag -d <tagname>…
  git tag [-n[<num>]] -l [--contains <commit>] [--no-contains <commit>]
  	[--points-at <object>] [--column[=<options>] | --no-column]
  	[--create-reflog] [--sort=<key>] [--format=<format>]
  	[--[no-]merged [<commit>]] [<pattern>…]
  git tag -v [--format=<format>] <tagname>…
  ```

  显示当前所有的标签

  ```
  git tag
  ```

  创建一个含附注类型的标签 使用-a(意思为 `annotated`  ) -m意思为标签的说明

  ```
  git tag -a v0.1 -m 'first-version'
  ```

  签署标签 

  如果有私钥的话 可以用GPG来签署标签 只需要把-a改为-s 意为signed

  **轻量级标签**

  简单的标签，直接给出标签名字就行了

  ```
  git tag v0.2
  ```

  验证标签

  ```
  git tag -v [tag-name]
  ```

  ```
  $ git tag -v v0.1
  object 064d37a19d91f61f74269995433ff13d9ebdc958
  type commit
  tag v0.1
  tagger andreby42 <38912428@qq.com> 1538508038 +0800
  
  first-version
  error: no signature found
  ```

  当然我这里没有签名

  **一次推送所有本地新增标签到远程**

  ```
  git push brach --tags
  ```

* 







