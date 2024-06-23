1. 设置Git的user name和email
```
$ git config --global user.name "bread"
$ git config --global user.email "1234567890@qq.com"
```
- 因为Git是分布式版本控制系统，所以需要填写用户名和邮箱作为一个标识。
- git config  --global 参数，有了这个参数，表示你这台机器上所有的Git仓库都会使用这个配置，当然你也可以对某个仓库指定的不同的用户名和邮箱。

2. 创建版本库
```
$ cd myproject
$ git init
```

2. clone 仓库到本地
```
$ git clone https://github.com/bread/myproject.git
```

3. 修改代码并上传代码到远端
```
$ cd myproject
$ vim README.md
$ git add README.md                 // 添加新文件或已经被修改过的文件到暂存区
$ git commit -m "项目启动"          // 把文件提交到仓库
$ git status                        // 查看是否还有文件未提交
    # 位于分支 main
    # 您的分支领先 'origin/main' 共 1 个提交。
    #   （使用 "git push" 来发布您的本地提交）
    #
    无文件要提交，干净的工作区
$ git push -u origin main           // 将本地仓库分支main推送到远端仓库
    # 位于分支 main
    无文件要提交，干净的工作区
```

4. git add 之后发现 add 错了一个文件 tags
```
$ git reset HEAD tags
```

5. 本地update远端的最新代码
```
$ git pull origin main
```

6. 文件/文件夹重命名
```
$ git mv oldname newname
```

7. 比较更改内容
```
// 如果在 $ git status 发现了未被提交的更改，则可以通过 $ git diff a.txt 来查看到底修改了什么。

$ git diff a.txt		    // 比较工作区与暂存区(已add但未commit文件)的差异
$ git diff HEAD a.txt		// 比较工作区与本地仓库的差异
$ git diff --cached a.txt	// 比较暂存区与本地仓库的差异
```

8. 查看历史
```
$ git log                   // git log命令显示从最近到最远的显示日志
$ git log --pretty=oneline  // 精简显示
```

9. 版本回退
```
$ git reset --hard HEAD^        // 把当前的版本回退到上一个版本
$ git reset --hard HEAD^^       // 把当前的版本回退到上上一个版本
$ git reset --hard HEAD~100     // 把当前的版本回退到前100个版本

// 回退版本到版本A后，可以通过命令cat readme.txt查看回退版本后的文件的内容
$ cat readme.txt
$ git log   // 将看不到版本A之后的提交记录

// 如果恢复版本A之后的某次提交，则
$ git reflog                    // 查找对应的版本号
$ git reset --hard 6fcfc89      // 回退到版本6fcfc89
```

10. 工作区和暂存区的区别
- 工作区：就是你在电脑上看到的目录，比如目录下testgit里的文件(.git隐藏目录版本库除外)。
- 版本库(Repository)：工作区有一个隐藏目录.git，这个不属于工作区，这是版本库。其中版本库里面存了很多东西，其中最重要的就是stage(暂存区)，还有Git为我们自动创建了第一个分支master，以及指向master的一个指针HEAD。
- 通过 git status 可以看到我们当前在哪个分支。
- 使用Git提交文件到版本库有两步：
    - 第一步：是使用 git add 把文件添加进去，实际上就是把文件添加到暂存区。
    - 第二步：使用git commit提交更改，实际上就是把暂存区的所有内容提交到当前分支上。

11. 撤销修改
```
$ git checkout -- readme.txt
```
- 命令 git checkout -- readme.txt 意思就是，把readme.txt文件在工作区做的修改全部撤销。
- 但这里有2种情况，如下：
    - readme.txt自动修改后，还没有放到暂存区，使用撤销修改就回到和版本库一模一样的状态。
    - 另外一种是readme.txt已经放入暂存区了，接着又作了修改，撤销修改就回到添加暂存区后的状态。
- 注意：命令git checkout -- readme.txt 中的 -- 很重要，如果没有 -- 的话，那么命令变成创建分支了。

12. 删除文件
- 一般情况下，可以直接在文件目录中把文件删了，或者使用rm命令：rm b.txt，如果想彻底从版本库中删掉此文件的话，可以再执行commit命令提交。
- 只要没有commit之前，如果想在版本库中恢复此文件可以使用如下命令：
    ```
    git checkout  -- b.txt
    ```

13. 创建与分并分支
```
$ git checkout -b dev       // 创建并切换分支
$ git branch                // 查看当前的分支

$ git checkout 命令加上 –b参数表示创建并切换，相当于如下2条命令：
git branch dev
git checkout dev    
```
- HEAD严格来说不是指向提交，而是指向master，master才是指向提交的，所以，HEAD指向的就是当前分支。
- git branch查看分支，会列出所有的分支，当前分支前面会添加一个星号。
- 在分支 dev 上的修改不会自动同步到 master 分支，因此需要把dev分支上的内容合并到分支master上，**在master分支上**使用如下命令 ：
    ```
    $ git merge dev
    ```
    - git merge命令用于合并指定分支到当前分支上。
- merge命令如果出现Fast-forward信息，这是Git告诉我们，这次合并是“快进模式”，也就是直接把master指向dev的当前提交，所以合并速度非常快。
- 合并完成后，可以接着删除dev分支了：
    ```
    $ git branch -d dev
    ```
- 总结即：
    ```
    查看分支：git branch

    创建分支：git branch name

    切换分支：git checkout name

    创建+切换分支：git checkout –b name
    
    合并某分支到当前分支：git merge name
    
    删除分支：git branch –d name
    ```

14. 解决冲突
- 如果在 dev 分支和 master 分支上的相同位置改了内容，则 master 分支在合并 dev 分支时，会报冲突。
    ```
    $ git merge dev
    ...
    Automatic merge failed: fix conflicts and then commit the result.
    ...
    $ git status        // 查看状态
    ...
        both modified:  readme.txt
    ...
    $ cat readme.txt    // 查看 readme.txt 内容
    <<<<<<<< HEAD
    999999999999
    ========
    888888888888
    >>>>>>>> dev
    
    // 修改后(保留master修改的内容，删除dev分支修改的内容)：
    $ cat readme.txt
    999999999999
    
    $ git add readme.txt
    $ git commit -m "conflict fixed"
    ```
    - Git用<<<<<<<，=======，>>>>>>>标记出不同分支的内容，其中 <<<<<<<< < HEAD是指主分支修改的内容，>>>>>>>> dev 是指dev上修改的内容，我们可以修改后保存。
- 如果我想查看分支合并的情况的话，需要使用命令 git log 命令行演示：
    ```
    $ git log
    ```

15. 分支管理策略
- 通常合并分支时，git一般使用”Fast forward”模式，在这种模式下，删除分支后，会丢掉分支信息，我们可以用带参数 -–no-ff来禁用”Fast forward”模式。
    ```
    // 合并 dev 分支，--no-ff 表示禁用 Fast forward
    $ git merge --no-ff -m "merge with no-ff" dev
    ```
- 分支策略：首先master主分支应该是非常稳定的，也就是用来发布新版本，一般情况下不允许在上面干活，干活一般情况下在新建的dev分支上干活，干完后，比如要发布，或者说dev分支代码稳定后可以合并到主分支master上来。

16. bug分支
- Git提供了一个stash功能，可以把当前工作现场 ”隐藏起来”，等以后恢复现场后继续工作：
    ```
    $ git stash         // 将当前的工作现场隐藏起来
    $ git status        // 此时查看状态，是干净的
    
    // 假设要修复一个404的bug，又假设要在主分支master上来修复，则需要在master分支上创建一个临时分支：
    $ git checkout -b issue-404
    $ cat readme.txt    // 修改前看看文件内容
    ...
    $ cat readme.txt    // 修改后再看看文件内容
    $ git add readme.txt
    $ git commit -m "fix bug 404"
    $ git checkout master   // 切换回master分支
    $ git merge --no-ff -m "merge bug fix 404" issue-404
    $ cat readme.txt    //  合并分支后查看文件内容，与issue-404内容一致
    $ git branch -d issue-404   // 在master分支上删除临时分支issue-404
    $ git checkout dev      // 最后从master分支切换回dev分支上干活
    $ git status        // 此时查看状态，是干净的（因为还没恢复现场）
    $ git stash list    // 查看工作现场
    $ git stash pop     // 恢复现场
    ```
- Git把stash内容存在某个地方了，但是需要恢复一下，可以使用如下2个方法：
    - git stash apply恢复，恢复后，stash内容并不删除，你需要使用命令git stash drop来删除。
    - 另一种方式是使用git stash pop，恢复的同时把stash内容也删除了。

17. 远端仓库
- 当你从远程库克隆时候，实际上Git自动把本地的master分支和远程的master分支对应起来了，并且远程库的默认名称是origin。
    - 要查看远程库的信息 使用 git remote
    - 要查看远程库的详细信息 使用 git remote –v
    ```
    $ git remote
    origin
    $ git remote -v
    origin https://........(fetch)      // 抓取
    origin https://........(push)       // 推送
    ```

18. 远端仓库之推送分支
- 推送分支就是把该分支上所有本地提交到远程库中，推送时，要指定本地分支，这样，Git就会把该分支推送到远程库对应的远程分支上，使用命令：
    ```
    $ git push origin master
    // $ git push origin dev    // 也可以推送其他分支，但是一般不这么做
    ```
- 一般情况下，那些分支要推送呢？
    - master分支是主分支，因此要时刻与远程同步。
    - 一些修复bug分支不需要推送到远程去，可以先合并到主分支上，然后把主分支master推送到远程去。
    
19. 远端仓库之抓取分支
```
1. 同事A把dev分支也要推送到远程去
$ git push origin dev

2. 同事B clone仓库
$ git clone https://xxxx.git

3. 同事B 要在 dev 分支上开发，那就必须把远程的origin的dev分支到本地来
$ git checkout -b dev origin/dev    // 创建远程origin的dev分支到本地来

4. 同事B开发完，add且commit之后，把现在的dev分支推送到远程去
$ git push origin dev

5. 假设同事A也在相同的地方作了修改，也试图推送到远程库时，会报错：
$ git checkout dev
...修改了文件，add且commit之后
$ git push origin dev
error: failed to push some refs to ....

6. 此时，需先用 git pull 把最新的提交从origin/dev抓下来，然后在本地合并，解决冲突，再推送：
$ git pull
失败，因为没有指定本地dev分支与远程origin/dev分支的链接，根据提示，设置dev和origin/dev的链接：如下：
$ git branch --set-upstream dev origin/dev
$ git pull
pull成功了，但是有冲突，需要解决，再push
解决冲突的方式和上面的一致，解决完，add且commit之后
$ git push origin dev
此时成功了！
```
- 因此，多人协作工作模式一般是这样的：
    - 首先，可以试图用git push origin branch-name推送自己的修改.
    - 如果推送失败，则因为远程分支比你的本地更新早，需要先用git pull试图合并。
    - 如果合并有冲突，则需要解决冲突，并在本地提交。再用git push origin branch-name推送。

20. Git基本常用命令如下：
```
mkdir XX                // 创建一个空目录 XX指目录名

pwd                     // 显示当前目录的路径。

git init                // 把当前的目录变成可以管理的git仓库，生成隐藏.git文件。

git add XX              // 把xx文件添加到暂存区去。

git commit –m “XX”      // 提交文件, –m 后面的是注释。

git status              // 查看仓库状态

git diff  XX            // 查看XX文件修改了那些内容

git log                 // 查看历史记录

git reset --hard HEAD^ 
或者 git reset --hard HEAD~ 回退到上一个版本
                        // 如果想回退到100个版本，使用git reset –hard HEAD~100

cat XX                  // 查看XX文件内容

git reflog              // 查看历史记录的版本号id

git checkout -- XX      // 把XX文件在工作区的修改全部撤销。

git rm XX               // 删除XX文件

git remote add origin https://github.com/bread/xxx.git  // 关联一个远程库

git push –u(第一次要用-u 以后不需要) origin master      // 把当前master分支推送到远程库

git clone https://github.com/bread/xxxxx.git            // 从远程库中克隆

git checkout –b dev     // 创建dev分支并切换到dev分支上

git branch              // 查看当前所有的分支

git checkout master     // 切换回master分支

git merge dev           // 在当前的分支上合并dev分支

git branch –d dev       // 删除dev分支

git branch name         // 创建分支

git stash               // 把当前的工作隐藏起来，等以后恢复现场后继续工作

git stash list          // 查看所有被隐藏的文件列表

git stash apply         // 恢复被隐藏的文件，但是内容不删除

git stash drop          // 删除文件

git stash pop           // 恢复文件的同时，也删除文件

git remote              // 查看远程库的信息

git remote –v           // 查看远程库的详细信息

git push origin master  // Git会把master分支推送到远程库对应的远程分支上
```

21. 切换tag
```
$ git status
位于分支 master
您的分支与上游分支 'origin/master' 一致。
$ git checkout v0.4.0                   // 注意：不用-b
注意：正在切换到 'v0.4.0'。

您正处于分离头指针状态。您可以查看、做试验性的修改及提交，并且您可以在切换
回一个分支时，丢弃在此状态下所做的提交而不对分支造成影响。

如果您想要通过创建分支来保留在此状态下所做的提交，您可以通过在 switch 命令
中添加参数 -c 来实现（现在或稍后）。例如：

  git switch -c <新分支名>

或者撤销此操作：

  git switch -

通过将配置变量 advice.detachedHead 设置为 false 来关闭此建议

HEAD 目前位于 f9130bb Merge branch 'bin-xxx-xxx' into 'master'
$ git branch
* (头指针分离于 v0.4.0)
  master

// 删除 tag 后再重新拉取 tag：
$ git tag -d v0.7.1
$ git pull
$ git checkout v0.7.1
```

22. 修改分支名字
```
$ git branch -m oldBranchName newBranchName
$ git push --delete origin oldBranchName
$ git push origin newBranchName 
$ git branch --set-upstream-to=origin/newBranchName 
```

23. 创建分支
```
$ git clone https://xxx.com/xxxxx.git
$ git checkout -b myBranchName
$ git push --set-upstream origin myBranchName
```

24. 拉取远端分支
```
$ git branch
* master
$ git pull
$ git checkout -b dev origin/dev
$ git branch
* dev
  master
```

25. 回退版本
```
$ git log
commit d3ad0937cb40f1214fa5adda3a859fce2221675b
Author: xiaohuamao <xiaohuamao@qq.com>
Date:   Thu Feb 23 15:54:48 2023 +0800

    1.删除无用json字段

commit 1e3f401289b7b6dbcb89059e6658ad094a91f5fa
Author: xiaohuamao <xiaohuamao@qq.com>
Date:   Thu Feb 23 15:54:04 2023 +0800

    1.[修改]合并代码的问题。
    
$ git reset --hard 1e3f401289b7b6dbcb89059e6658ad094a91f5fa
```

26. 查看某个文件的历史修改记录和内容
```
git log -- test.cpp                 // 仅查看历史修改记录
git -P log --follow -p test.cpp     // 不仅有历史修改记录，还有对应的文件内容

git show 123444fee8873121d9ada3e67308bfasaa150a4c           // 查看某次提交的内容
git show 123444fee8873121d9ada3e67308bfasaa150a4c --stat    // 查看某次提交的文件列表
```

27. 记录远程仓库的用户名和密码
```
$ git config --global credential.helper store
// 设置完成后，再 git pull 一次，然后输入用户名和密码，后面就不用再输入用户名和密码了。
```

28. dev 分支合并 master 分支上的某个提交 
```
git checkout master
git pull
git log     // 找到需要合并的提交的 commitid f92829dd57b8bfa84b77d6672474b95d9583beaa
git checkout dev
git pull
git cherry-pick f92829dd57b8bfa84b77d6672474b95d9583beaa
error: 提交 f92829dd57b8bfa84b77d6672474b95d9583beaa 是一个合并提交但未提供 -m 选项。
fatal: 拣选失败
git cherry-pick -m 1 f92829dd57b8bfa84b77d6672474b95d9583beaa
自动合并 src/my_manager.hpp
自动合并 projectx.pro
冲突（内容）：合并冲突于 projectx.pro
error: 不能应用 f92829d... Merge branch 'dev' into 'master'
提示：冲突解决完毕后，用 'git add <路径>' 或 'git rm <路径>'
提示：对修正后的文件做标记，然后用 'git commit' 提交
git add .
git commit -m "xxxx"
```

29. git中 rebase 和 merge 的区别是什么？

区别：
1. rebase 把当前的 commit 放到公共分支的最后面，merge 把当前的 commit 和公共分支合并在一起；
2. 用 merge 命令解决完冲突后会产生一个 commit，而用 rebase 命令解决完冲突后不会产生额外的 commit。

rebase 会把当前分支的 commit 放到公共分支的最后面，所以叫变基。就好像从公共分支又重新拉出来这个分支一样。

举例：如果从 master 拉个 feature 分支出来，然后提交了几个 commit，这个时候刚好有人把他开发的东西合并到 master 了，这个时候 master 就比你拉分支的时候多了几个 commit，如果这个时候你 rebase master 的话，就会把你当前的几个 commit，放到那个人 commit 的后面。而 merge 会把公共分支和你当前的 commit 合并在一起，形成一个新的 commit 提交。

30. 查看用户信息
```
$ git config --list
```

31. 推送本地已有仓库到远端
```
$ git remote add origin http://bread@xxx.xxx.xxx.xxx:8080/bread/glog-0.6.0.git
$ git remote -v
$ git push -u origin master
```

32. 合并远程分支
```
$ git status
$ git pull
$ git merge origin/master
```
