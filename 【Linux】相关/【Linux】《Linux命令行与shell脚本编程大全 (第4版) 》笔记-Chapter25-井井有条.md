###### 二十五、井井有条

1. 版本控制（也称为源代码控制或修订控制）是一种组织各种项目文件并跟踪其更新的方法（或系统）。
- 版本控制系统（ version control system，VCS）提供了一个公共中央位置来存储和合并 bash 脚本文件， 以便轻松访问脚本的最新版本。 VCS 能够保护文件，使其不会被另一个脚本编写者意外覆盖。同时还消除了谁当前正在修改什么内容这类额外通信。
- 分布式 VCS 使脚本项目开发变得更加容易。脚本编写者可以在自己的 Linux 系统中进行开发或修改工作。 一旦达到修改目标， 就将修改后的文件副本和 VCS 元数据发送到远程中央系统，其他团队成员可以下载这个最新的项目版本并进行测试，或是继续他们自己的修改任务。
- Git 是一种分布式VCS，多部署于敏捷和持续软件开发环境中。不过它也可用于管理bash shell 脚本。

2. 工作目录
- 工作目录是所有脚本的创建、修改和审查之地。它通常是脚本编写者的主目录中的某个子目录， 类似于/home/christine/scripts。最好为每个项目都创建一个新的子目录，因为 Git 会在其中放置文件，以便进行跟踪。

3. 暂存区
- 暂存区也称为索引。该区域和工作目录位于同一系统。 bash 脚本通过 Git 命令（git add） 在暂存区内注册。通过 git init 命令，暂存区在工作目录中设置了一个名为.git 的隐藏子目录。
- 当脚本被编入暂存区时， Git 会在索引文件.git/index 中创建或更新脚本信息。记录的数据包括校验和、时间戳和相关的脚本文件名。
- 除了更新索引文件， Git 还会压缩脚本文件并将这些压缩文件作为对象（也称为 blob）存储 在.git/objects/目录中。如果脚本已被修改，则将其作为一个新对象压缩并存储在.git/objects/目录中。 Git 不只存储脚本的改动，还会保留每个已修改的脚本的压缩副本。

4. 本地仓库
- 本地仓库包括每个脚本文件的历史记录。它也会用到工作目录的.git 子目录。脚本文件的各 个版本（称为项目树）和提交信息之间的关系通过 Git 命令（ git commit），以对象的方式存储在.git/objects/ 目录中。
- 项目树和提交数据合起来称为快照。每次提交数据都会创建一个新快照。不过，旧快照并不 会被删除，依然可以查看。如果需要，还可以返回到之前的快照，这是 Git 另一个不错的特性。

5. 远程仓库
- 在 Git 配置中，远程仓库通常位于云端，提供代码托管服务。然而，也可以在本地网络中的 另一台服务器上建立代码托管站点作为远程仓库。
- 著名的远程仓库有 GitHub 、GitLab 、BitBucket 和 Launchpad。

6. 分支
- Git 还提供了一个名为分支的特性， 该特性可以在各种脚本项目中发挥作用。
- 支是本地仓库中属于特定项目的一个区域。举例来说，你可以将脚本项目的主分支命名为 main，当你打算对 main 分支中的脚本进行改动时，最好的做法是创建一个新分支（比如命名为 modification），在该分支中修改脚本。一旦脚本的改动通过了测试， modification 分支中的脚本通常就会被合并回主分支。
- 使用这种方法的好处在于，存放在 main 分支中的脚本仍具有生产价值，因为正在被修改和 测试的 bash shell 脚本位于另一个分支。只有当修改过的脚本通过测试之后，才会被并入 main主分支。

7. 克隆
- Git 的另一个特性是复制项目。这个过程称为克隆。如果你的团队有新人加入，那么他可以 从远程仓库克隆脚本和跟踪文件，获得开展工作所需的一切资源。
- 在 Git 中，克隆（cloning）和分叉（forking）是两种紧密相关的操作。使用 git clone
命令将文件从远程仓库下载到本地系统，这一过程是克隆。将文件从一个远程仓库复制到另一个远程仓库，这一过程是分叉。

8. 使用 Git 作为 VCS
- 将 Git 作为 VCS 能带来如下好处：
    - 性能：Git 只操作本地文件，这提高了其部署速度。同远程仓库之间收发文件属于例外情况。
    - 历史文件：从文件被注册那一刻起， Git 就开始使用索引来记录文件的内容。当对本地存储库的提交完成时， Git 会及时创建并存储对该快照的引用。
    - 准确性：Git 使用校验和来保护文件完整性。
    - 去中心化：脚本编写者可以在同一个项目中工作，但不必位于同一网络或系统。

9. 设置 Git 环境
- Git 通常并非默认安装项，在设置 Git 环境之前，需要自行安装 git 软件包。
- 在 CentOS Linux 发行版中安装 Git 的过程如下：
    ```
    $ sudo dnf install git
    ...
    Complete!
    $ which git
    /usr/bin/git
    ```
- 在 Ubuntu Linux 发行版中安装 Git 的过程如下：
    ```
    $ sudo apt install git
    ...
    Processing triggers for man-db (2.9.1-1) ...
    $ which git
    /usr/bin/git
    ```
- 安装好 git 软件包之后，为新的脚本项目设置 Git 环境涉及以下 4 个基本步骤：
    - (1) 创建工作目录。
    - (2) 初始化.git/子目录。
    - (3) 设置本地仓库选项。
    - (4) 确定远程仓库位置
- 具体步骤如下：
    ```
    1、首先，创建工作目录。在本地主目录下创建一个子目录即可：
    $ mkdir MWGuard
    $
    $ cd MWGuard/
    $
    $ pwd
    /home/christine/MWGuard
    $

    2、然后，在工作目录中初始化.git/子目录。这要用到 git init 命令：
    $ git init
    Initialized empty Git repository in /home/christine/MWGuard/.git/
    $
    $ ls -ld .git
    drwxrwxr-x 7 christine christine 4096 Aug 24 14:49 .git
    $

    3、如果是首次使用，则设置name和email：
    $ git config --global user.name "Christine Bresnahan"
    $
    $ git config --global user.email "cbresn1723@gmail.com"
    $
    $ git config --get user.name
    Christine Bresnahan
    $
    $ git config --get user.email
    cbresn1723@gmail.com
    $
    
    4、配置好本地 Git 环境之后，就可以建立项目的远程仓库了。建立好项目的远程仓库之后，需要把仓库地址记下来。随后向远程仓库发送项目文件时， 要 用到这个地址。
    ```
    - 创建的子目录 MWGrard 用于脚本项目。然后， 使用 cd 命令进入工作目录。
    - git init 命令创建了.git/子目录。因为目录名之前有点号（.），所以普通的 ls 命令无法将其显示出来。使用 ls -la 命令或将该目录名作为 ls -ld 命令的参数就可以看到相关信息。
    - 可以同时拥有多个项目目录。为每个项目创建单独的工作目录即可。
    - 如果你是首次在系统中构建.git/子目录， 则需将姓名和 email 地址添加到 Git 的全局仓库配置文件中。这些标识信息有助于跟踪文件变更，尤其是多人参与项目的时候。
    - 在 git config 命令中加入--global 选项，就能把 user.name 和 user.email 保存在 Git 全局配置文件中。注意，要想查看此信息，可以使用--get 选项，并将数据名称作为参数。
    - Git 全局配置信息表示这些数据会应用于系统中的所有Git 项目。 Git 本地配置信息仅应用于工作目录中的特定 Git 项目。
    - Git 全局配置信息保存在主目录的.gitconfig 文件中，本地配置信息保存在 working-directory/ .git/config 文件中。注意，有些系统还有系统级的配置文件/etc/gitconfig。

- 要查看这些文件中的各种配置信息， 可以使用 git config --list 命令：
    ```
    $ git config --list
    user.name=Christine Bresnahan
    user.email=cbresn1723@gmail.com
    core.repositoryformatversion=0
    core.filemode=true
    core.bare=false
    core.logallrefupdates=true
    $ cat /home/christine/.gitconfig
    [user]
        name = Christine Bresnahan
        email = cbresn1723@gmail.com
    $
    $ cat /home/christine/MWGuard/.git/config
    [core]
        repositoryformatversion = 0
        filemode = true
        bare = false
        logallrefupdates = true
    $
    ```
    - 通过--list 选项显示的设置信息采用 file-section.name 格式。注意，当使用 cat 命令将两个 Git 配置文件（全局和项目的本地仓库）输出至 STDOUT 时， 会显示节名（section name）及其包含的数据。
    - 
- 尽管 Git 能够处理任意文件类型，但其相关工具主要针对的是纯文本文件，比如 bash shell 脚本。因此要注意，不是所有的 git 工具都能用于非文本文件。

10. 使用 Git 提交文件
- 建立好 Git 环境之后，就可以使用它的各种组织功能了。这也有 4 个基本步骤：
    - (1) 创建或修改脚本。
    - (2) 将脚本添加到暂存区（索引）。
    - (3) 将脚本提交至本地仓库。
    - (4) 将脚本推送至远程仓库。
- 例子：
    ```
    1、创建好脚本之后， 使用 git add 命令将其添加到暂存区（索引）。由于该脚本目前不在工作 目录/home/christine/MWGuard  中，因此需要先把它复制过来。然后切换到工作目录（通过 pwd 命令确认），执行 git add 命令：
    $ pwd
    /home/christine/scripts
    $
    $ cp MyGitExampleScript.sh /home/christine/MWGuard/
    $
    $ cd /home/christine/MWGuard/
    $
    $ pwd
    /home/christine/MWGuard
    $
    $ ls *.sh
    MyGitExampleScript.sh
    $
    $ git add MyGitExampleScript.sh
    $
    $ git status
    [&hellip;]
    No commits yet
    Changes to be committed:
    (use "git rm --cached <file>..." to unstage)
    new file:   MyGitExampleScript.sh
    $
    
    2、下一步是使用 git commit 命令将项目提交至本地仓库。可以使用-m 选项来添加注释，这有助于记录（ documenting ）提交。
    $ git commit -m "Initial Commit"
    [&hellip;] Initial Commit
        1 file changed, 5 insertions(+)
        create mode 100644 MyGitExampleScript.sh
    $
    $ cat .git/COMMIT_EDITMSG
    Initial Commit
    $
    $ git status
    [&hellip;]
    nothing to commit, working tree clean
    $
    
    3、创建 README.md 文件，将其加入暂存区并提交到本地仓库。
    $ pwd
    /home/christine/MWGuard
    $
    $ ls
    MyGitExampleScript.sh
    $
    $ echo "# Milky Way Guardian" > README.md
    $ echo "## Script Project" >> README.md
    $
    $ cat README.md
    # Milky Way Guardian
    ## Script Project
    $
    $ git add README.md
    $
    $ git status
    [...]
    Changes to be committed:
        (use "git restore --staged <file>..." to unstage)
            new file: README.md
    $
    $ git commit -m "README.md commit"
     [...] README.md commit
        1 file changed, 2 insertions(+)
        create mode 100644 README.md
    $
    $ git status
     [...]
    nothing to commit, working tree clean
    $
    ```
    - git add 命令不会产生任何输出。因此， 要想知道脚本是否成功， 需要使用git status 命令。 该命令显示，一个新文件 MyGitExampleScript.sh 已经被加入索引。
    - 注释被保存在 COMMIT_EDITMSG 文件中，它能够帮助我们记录为什么要修改脚本。
    - 执行过 git commit 之后， git status 命令会显示以下消息：nothing to commit, working directory clean。这说明 Git 现在认为，工作目录中的所有文件都已经提交至本地仓库了。
    - 如果使用 git commit 命令时没有加上-m 选项，那么你会被引至 vim 编辑器，要求手动编辑.git/COMMIT_EDITMSG 文件。
   
- 如果需要，可以将当前工作目录中的所有脚本同时添加到暂存区（索引）。只需使用 git
add .命令即可。注意，该命令结尾有个点号（.）！ 这个点号相当于一个通配符，告诉 Git 把工作目录中的所有文件都加入暂存区。但是，如果不想把某些文件添加到暂存区，则可以在工作目录中创建一个.gitignore  文件，将不希望加入暂存区的文件或目录名写入该文件。这样， git add .命令就会忽略这些文件或目录，只把其他的文件或目录加入暂存区。
- 暂存区的索引文件是.git/index。如果对该文件使用 file 命令，则其类型会显示为 Git index。Git 会使用此文件跟踪变更：
    ```
    $ file .git/index
    .git/index: Git index, version 2, 1 entries
    $
    ```
- 如果这是一个新的脚本项目，那么在注册过远程仓库账户后，需要创建一个称为 Markdown file 的特殊文件，其内容会显示在远程仓库的 Web 页面上，描述该仓库的相关信息。该文件使用 Markdown 语言编写。你需要将文件命名为 README.md。
- 可以随时查看 Git 日志，但最好在将脚本项目推送到远程存储库之前做这件事。每次提交都有一个对应的哈希值作为标识，这个值也会出现在日志中。此外，请注意各种注释以及日期和作者信息。
    ```
    $ git log
    commit 898330bd0b01e0b6eee507c5eeb3c72f9544f506[...]
    Author: Christine Bresnahan <cbresn1723@gmail.com>
    Date:   Mon Aug 24 15:58:52 2020 -0400
        
        README.md commit
    
    commit 3b484638bc6e391d0a1b816946cba8c5f4bbc8e6
    Author: Christine Bresnahan <cbresn1723@gmail.com>
    Date:   Mon Aug 24 15:46:56 2020 -0400
    
        Initial Commit
    $
    ```
- 在向远程仓库推送项目之前，需要先在系统中配置远程仓库地址。在使用 Git 服务提供商 （比如 GitHub）设置远程仓库时，它会向你提供此地址。
- 可以使用 git remote add origin URL 命令来添加地址， 其中 URL 就是远程仓库地址：
    ```
    $ git remote add origin https://github.com/C-Bresnahan/MWGuard.git
    $
    $ git remote -v
    origin https://github.com/C-Bresnahan/MWGuard.git (fetch)
    origin https://github.com/C-Bresnahan/MWGuard.git (push)
    $
    ```
    - 注意，我们通过 git remote -v 命令检查了远程仓库地址的状态。在推送项目之前，最好先检查一下地址。如果地址有误或有输入错误，那么推送操作就会失败，因此一定要仔细检查！
    - 如果地址不对（比如有输入错误）， 则可以通过 git remote rm origin 命令删除远程仓库地址， 然后使用正确的地址重新设置。
- 配置好远程仓库地址之后，就可以向其推送脚本项目了。但在此之前，为简单起见，我们打算使用 git branch 命令把主分支重命名为main：
    ```
    $ git branch -m main
    $
    $ git branch --show-current
    main
    $
    ```
    - 注意，可以使用 git branch --show-current 命令查看分支的当前名称。在推送之前， 最好先确认分支名无误，在 push 命令中也要用到此分支名。
- 将脚本复制到远程仓库。这需要在 push 命令中加入-u origin 选项来指定仓库位置和当前使用的分支名 main：
    ```
    $ git remote add origin https://github.com/C-Bresnahan/MWGuard.git
    $
    $ git push -u origin main
    Username for 'https://github.com ': C-Bresnahan
    Password for 'https://C-Bresnahan@github.com ':
    Enumerating objects: 6, done.
    Counting objects: 100% (6/6), done.
    Compressing objects: 100% (4/4), done.
    Writing objects: 100% (6/6), 604 bytes | 60.00 KiB/s, done.
    Total 6 (delta 0), reused 0 (delta 0)
    To https://github.com/C-Bresnahan/MWGuard.git
        * [new branch]      main -> main
    Branch 'main' set up to track remote branch 'main' from 'origin'.
    $
    ```
    - 远程仓库通常要求输入用户名和密码。项目被推送至远程仓库之后，你应该能通过 Web 浏览器看到。
- 远程仓库真正的美妙之处在于， Linux 管理团队中参与此项目的任何人都可以使用 git pull 命令从中拉取最新版本的脚本。
    ```
    $ git remote add origin https://github.com/C-Bresnahan/MWGuard.git
    $
    $ git pull origin main
    remote: Enumerating objects: 6, done.
    remote: Counting objects: 100% (6/6), done.
    remote: Compressing objects: 100% (4/4), done.
    remote: Total 6 (delta 0), reused 6 (delta 0), pack-reused 0
    Unpacking objects: 100% (6/6), 584 bytes | 58.00 KiB/s, done.
    From https://github.com/C-Bresnahan/MWGuard
        * branch
        * [new branch] 
    $
    ```
    - 如果拉取项目文件的个人在其本地存储库中有尚未上传到远程仓库的特定脚本的修改版 本， 则 git pull 命令会失败并会保护该脚本。
- 如果未参与该项目的人想得到脚本的最新版本， 那么当他们尝试使用 git remote add origin 命令时， 会收到类似于 fatal: not a git repository 的错误消息。对这些人而言， 最好先克隆该项目。
- 开发团队的新成员可以使用 git clone 命令将整个脚本项目从远程仓库复制到自己的本地 系统：
    ```
    $ git clone https://github.com/C-Bresnahan/MWGuard.git
    Cloning into 'MWGuard'...
    remote: Enumerating objects: 6, done.
    remote: Counting objects: 100% (6/6), done.
    remote: Compressing objects: 100% (4/4), done.
    remote: Total 6 (delta 0), reused 6 (delta 0), pack-reused 0
    Unpacking objects: 100% (6/6), 584 bytes | 58.00 KiB/s, done.
    $
    $ ls
    MWGuard
    $
    $ cd MWGuard/
    $
    $ ls -a
    .  ..  .git MyGitExampleScript.sh README.md
    $
    $ git log
    commit [...](HEAD -> main, origin/main, origin/HEAD)
    Author: Christine Bresnahan <cbresn1723@gmail.com>
    Date:   Mon Aug 24 15:58:52 2020 -0400
    
        README.md commit
    
    commit 3b484638bc6e391d0a1b816946cba8c5f4bbc8e6
    Author: Christine Bresnahan <cbresn1723@gmail.com>
    Date:   Mon Aug 24 15:46:56 2020 -0400
    
        Initial Commit
    $
    ```
    - 从远程仓库克隆项目时会自动创建工作目录以及.git/目录、 Git 暂存区（索引）和本地仓库。 git log 命令可以显示项目历史。
