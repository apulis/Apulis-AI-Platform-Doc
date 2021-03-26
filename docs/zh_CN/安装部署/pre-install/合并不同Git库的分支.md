git合并不同库的分支
--------------------------------------------------------------

*合并开源仓库的方法*

* 将私有仓库的主分支（如develop）同步到其开放分支（如to_public）

   + 主分支`develop`或是`master`是预发布代码
   + 开放分支`to_public`，是经过处理后可开放的代码

   1. 拉取`to_public`
   
   2. 配置或更新（开放代码）过滤条件
   新建一个 `.gitattributes` 用于指定非文本文件的对比合并方式。
      ```
      # 假如你要避免这个文件，config.xml
      vim .gitattributes

      docs    merge=ours
      Readme* merge=ours
      README* merge=ours
      readme* merge=ours
      License* merge=ours
      LICENSE merge=ours
      ```
   3. 合并`develop`更新（开放代码）

   `git merge origin/develop -X theirs --no-ff`
   
   4. 处理合并冲突后同步`to_public`

   `git push origin to_public`

* 同步`to_public`到开源仓库 `apulis platform`

   1. 添加github开放仓库链接
   `git remote add github https://github.com/apulis/apulis_platform.git`
   2. 强制push到apulis_platform
   `git push github master --force`
   3. 添加gitee开放仓库链接
   `git remote add gitee https://gitee.com/apulis/apulis_platform.git`
   4. 强制push到apulis_platform
   `git push gitee master --force`

* 同步发布指定的分支

1. switch到新分支如：release-0.1.6
   ```
   git pull
   git branch -a 
   git checkout -b release-0.1.3 remotes/origin/release-0.1.6
   ```
2. 上传到远端代码库
   ```bash
   git remote -v

   git remote add githubopen https://github.com/apulis/apulis_platform.git 

   git push githubopen release-0.1.6 # --force --allow-unrelated-histories
   
* 使用github action 同步代码库
  *Action要创建在对应的分支上，才能监控到该分支的push ,pull request触发执行*
  参考：`ci/github_action_sync_public.yml`

---
**FAQ:**

1. 代码开发的时候，有时需要把某分支（比如develop分支）的某一次提交合并到另一分支（比如master分支），这就需要用到git cherry-pick命令。

   ```bash
   git checkout develop # 切换到develop分支
   git log # 查找需要合并的commit记录，比如commitID：7fcb3defff；
   git checkout master # 切换到master分支<br>
   git cherry-pick 7fcb3defff  # 就把该条commit记录合并到了master分支，这只是在本地合并到了master分支；
   git push # 提交到master远程 
   # 至此，就把develop分支的这条commit所涉及的更改合并到了master分支
   ```
2. 撤销合并

   `sudo git merge --abort`

3. 合并冲突分支
   ```
   git branch -a 

   git merge dlws/public –no-ff 
   # 如果冲突太多，计划以新代码为主
   git  cherry-pick ..<master>  # 预先查看冲突文件并处理
   # git reset HEAD               # 撤销add
   # 如果远端冲突太多，策略必须要用远端代码
   git merge origin/master -X theirs --no-ff
   ```
**参考**

1. [git 合并策略](https://blog.walterlv.com/post/git-merge-strategy.html#recursive)

FAQ:

1. merge 失败有冲突，回退到merge 之前
   `git reset --merge`

2. 不同分支merge：fatal: refusing to merge unrelated histories

   `--allow-unrelated-histories` --no-off

3. 撤销失败的merge

   `git merge --abort `