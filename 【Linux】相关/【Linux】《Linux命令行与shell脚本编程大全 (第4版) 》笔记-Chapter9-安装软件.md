###### 九、安装软件

1. 软件包管理系统使用数据库来记录下列内容：
    - Linux 系统中已安装的软件包。
    - 每个软件包安装了哪些文件。
    - 每个已安装的软件包的版本。

2. 软件包存储在称为仓库（repository）的服务器上， 可以利用本地 Linux 系统中的软件包管理 器通过 Internet访问，在其中搜索新的软件包，或是更新系统中已安装的软件包。

3. 软件包通常存在依赖关系，为了能够正常运行，被依赖的包必须提前安装。软件包管理器会检测这些依赖关系并提前安装好所有额外的软件包。

4. Linux 中广泛使用的两种主要的软件包管理系统基础工具是 dpkg 和 rpm。
    - 基于 Debian 的发行版（比如 Ubuntu 和 Linux Mint）使用的是 dpkg 命令，该命令是其软件 包管理系统的底层基础。 dpkg 会直接和 Linux 系统中的软件包管理系统交互，用于安装、管理 和删除软件包。
    - 基于 Red Hat 的发行版（比如 Fedora、CentOS 和 openSUSE）使用的是 rpm 命令， 该命令是 其软件包管理系统的底层基础。与 dpkg 命令类似， rpm 命令也可以安装、管理和删除软件包。
- 注意， 这两个命令是它们各自软件包管理系统的核心， 并不代表整个软件包管理系统。许多使用 dpkg 命令或 rpm 命令的 Linux 发行版在这些基本命令的基础上构建了另一些专业的软件包管理器。

5. dpkg 命令是基于 Debian 的软件包管理器的核心，用于在 Linux 系统中安装、更新、删除 DEB 包文件。
- 从所用的 Linux 发行版仓库中安装软件包，需要使用 APT（advanced package tool）工具集。
    - apt-cache
    - apt-get
    - apt
- apt 命令本质上是 apt-cache 命令和 apt-get 命令的前端。 APT 的好处是不需要记住什 么时候该使用哪个工具——它涵盖了软件包管理所需的方方面面。
- apt 命令的基本格式如下：
    ```
    apt [options] command
    ```
    - command 定义了 apt 要执行的操作。如果需要，可以指定一个或多个 options 进行微调。

6. 使用 apt 管理软件包
- apt list 命 令会显示仓库中所有可用的软件包， 如果再加入--installed 选项，就可以限制仅输出那些已 安装在系统中的软件包。
    ```
    $ apt --installed list
    ```
- 如果已经知道系统中的某个软件包， 希望显示其详细信息，可以使用 show 命令来操作： 
    ```
    apt show package_name
    
    $ apt show zsh
    ```
- apt show命令并不会指明软件包是否已经安装。它只根据软件仓库显示该软件包的详细信息。
- 有一个细节无法通过 apt 获得，即与特定软件包相关的所有文件。为此，需要使用 dpkg 命令：
    ```
    dpkg -L package_name
    
    $ dpkg -L acl
    ```
- 也可以执行相反的操作，即找出特定的文件属于哪个软件包（注意，文件要使用绝对路径）：
    ```
    dpkg --search absolute_file_name
    
    $ dpkg --search /bin/getfacl
    acl: /bin/getfacl               // 表明文件 getfacl属于 acl 软件包
    ```

7. 使用 apt 安装软件包
- 同时使用 apt 命令与 search 命令查找特定的软件包：
    ```
    apt search package_name
    
    $ apt --names-only search zsh
    ``` 
    - search 命令的妙处在于不需要在 package_name 周围添加通配符，直接就有通配符的效果。在默认情况下， search 命令显示的是在名称或描述中包含搜索关键字的那些软件包，这有时候会产生误导。如果只想搜索软件包名称，可以加入--name-only 选项。
- 一旦找到待安装的软件包， 就可以使用 apt 安装了：
    ```
    apt install package_name
    
    $ sudo apt install zsh
    ```
- sudo 命令允许以root 用户身份执行命令。 这类管理任务都需要使用 sudo 命令帮助完成。
- 可以使用 list 命令的--installed 选项检查安装是否正确。如果在输出中看到了软件包， 那么说明已经安装好了。
- 注意，在安装指定的软件包时， apt 也会要求安装其他软件包。这是因为 apt 会自动解析 必要的依赖关系，根据需要安装额外的库和软件包。

8. 使用 apt 升级软件
- upgrade 命令可以使用仓库中的任何新版本安全地升级系统中所有的软件包：
    ```
    apt upgrade
    ```
    - 注意，该命令无须使用任何软件包名称作为参数。原因在于 upgrade 会将所有已安装的软 件包升级为仓库中可用的最新版本。
- upgrade 命令在升级过程中不会删除任何软件包。如果必须删除某个软件包才能完成升级，可以使用以下命令：
    ```
    apt full-upgrade
    ```
- 应该定期执行 apt upgrade，保持系统处于最新状态。

9. 使用 apt 卸载软件包
- apt 的 remove 命令可以删除软件包， 同时保留数据和配置文件。如果要将软件包以及相关 的数据和配置文件全部删除，那么需要使用 purge 选项：
    ```
    $ sudo apt purge zsh
    
    $ sudo apt autoremove
    ```
    - 注意， 在 purge 的输出中， apt 警告我们 zsh-common 软件包存在依赖， 不能自动删除， 以 免其他软件包还有需要。如果确定有依赖关系的软件包不会再有他用，可以使用 autoremove 命令将其删除。
    - autoremove 命令会检查所有被标记为存在依赖关系且不再被需要的软件包。

10. apt 仓库
- apt 默认的软件存储库位置是在安装 Linux 发行版时设置的。仓库位置保存在文件/etc/apt/ sources.list 中。
- apt 只会从这些仓库中拉取软件。此外，在搜索要安装或更新的软件时， apt 也只检查这些仓库。如果 你想为你的软件包管理系统加入一些额外的软件仓库，那就得修改文件了。
- Linux 发行版开发人员努力确保加入仓库的软件包版本不会彼此之间发生冲突。通过仓库升级或安装软件包通常是最安全的方法。
- sources.list 文件使用下列结构来指定仓库源：
    ```
    deb (or deb-src) address  distribution_name  package_type_list
    ```
    - deb 或deb-src 指定了软件包的类型。 deb 表明这是一个已编译程序的仓库源， 而deb-src 表明这是一个源代码的仓库源。
    - address 是软件仓库的网址。 distribution_name 是该软件仓库的发行版的版本名称（并不是说用了这个发行版，只是说用了这个发行版的仓库。例如， 在 Linux Mint 的 sources.list 文件中， 你能看到混用了 Linux Mint 和 Ubuntu 的软件仓库）。
    - package_type_list 可能并不止一个单词，它还表明仓库里面有什么类型的软件包。 你可能会在其中看到如 main 、restricted 、universe 或 partner这样的词。

11. 和基于 Debian 的发行版类似，基于 Red Hat 的系统有以下几种前端工具：
    - yum：用于 Red Hat、CentOS 和 Fedora。
    - zypper：用于 openSUSE。
    - dnf：yum 的升级版，有一些新增的特性。

12. dnf 命令的语法为：
    ```
    dnf [options]  [...]
    
    // 要找出系统中已安装的软件包，可以使用下列命令：
    dnf list installed
    
    dnf list installed > installed_software
    ```
    - 输出的信息可能会在屏幕上一闪而过，所以最好是将已安装软件的列表重定向到一个文件 中，然后使用 more 命令或 less 命令（或GUI 编辑器）来仔细查看。

13. 要想查找特定软件包的详细信息，dnf 除了会给出软件包的详细描述，它还可以告诉你软件包是否已经安装：
    ```
    $ dnf list xterm
    $ dnf list installed xterm
    ```

14. 如果需要找出文件系统中的某个文件是由哪个软件包安装的，只需输入以下命令：
    ```
    dnf provides file_name
    
    $ dnf provides /usr/bin/gzip
    Last metadata expiration check: 0:12:06 ago on Sat 16 May 2020 12:10:24 PM EDT.
    gzip-1.10-1.fc31.x86_64 : The GNU data compression program
    Repo        : @System
    Matched from:
    Filename    : /usr/bin/gzip
    gzip-1.10-1.fc31.x86_64 : The GNU data compression program
    Repo        : fedora
    Matched from:
    Filename    : /usr/bin/gzip
    ```
    - 上面例子中，dnf 分别检查了两个仓库： 本地系统和默认的 fedora 仓库。
    
15. 使用 dnf 安装软件
- 下面是安装软件包的基础命令，包括其所需的全部库 以及依赖：
    ```
    dnf install package_name
    
    $ sudo dnf install zsh
    ```
    - 该命令允许以root用户身份执行命令。应该仅在执行安装和更新软件这类管理任务的时候才临时切换为 root 用户。

16. 使用 dnf 升级软件
- 要查看已安装软件包的所有可用更新，可以输入下列命令：
    ```
    dnf list upgrades
    ```
- 如果你发现有需要升级的软件包， 可以输入下列命令：
    ```
    dnf upgrade package_name
    ```
- 如果想升级更新列表中的所有软件包，那么只需输入下列命令即可：
    ```
    dnf upgrade
    ```
- dnf 还有一个很不错的特性： upgrade-minimal 命令。它会将软件包升级至最新的 bug修复版或安全补丁版， 而不是最新的最高版本。

17. 使用 dnf 卸载软件
- dnf 还提供了一种简单的方法来卸载系统中不需要的软件（但尚没有选项或命令可以在卸载软件的同时保留配置文件或数据文件）：
    ```
    dnf remove package_name
    ```

18. 处理损坏的依赖关系
- 有时在安装多个软件包时， 一个软件包的依赖关系可能会被另一个软件包搞乱。这称为依赖 关系损坏（broken dependency）。
- 如果你的系统出现了这种情况，可以先试试下列命令：
    ```
    dnf clean all
    ```
- 然后尝试使用 dnf 命令的upgrade 选项。有时， 只要清理了放错位置的文件就可以了。
- 如果还解决不了问题，再试试下列命令：
    ```
    dnf repoquery --deplist package_name
    
    # dnf repoquery --deplist xterm
    ```
    - 该命令会显示指定软件包的所有依赖关系以及哪些软件包提供了这种依赖。只要知道软件包 需要哪个库， 就可以自行安装了。
- yum 工具的 upgrade 命令支持--skip-broken 选项，可以跳过依赖关系损坏的软件包，同时仍尝试继续升级其他包。 dnf 工具则自动执行该操作。

19. RPM 仓库
- dnf 在安装时也设置了软件仓库。
- 要想查看当前拉取软件的仓库，可以使用下列命令：
    ```
    dnf repolist
    ```
- 如果发现其中没有所需的仓库，那就需要编辑配置文件了。有两个地方可以找到 dnf 仓库的定义。
    - 配置文件/etc/dnf/dnf.conf。
    - /etc/yum.repos.d 目录中的单独文件。

20. 使用容器管理软件
- 云计算带来了应用程序打包方式的一种新范式： 应用程序容器（application container）。应用 程序容器创建了一个环境， 其中包含了应用程序运行所需的全部文件， 包括运行时库文件。开发 人员随后可以将应用程序容器作为单个软件包分发，保证能够在任何 Linux 系统中正常运行。

21. 使用 snap 容器
- Ubuntu Linux 发行版的创建者 Canonical 开发了一种称为 snap 的应用程序容器格式（于 Ubuntu 16.04 LTS 发布时引入）。
- snap  打包系统会将应用程序所需的所有文件集中到单个 snap 分发文件中。snapd 应用程序运行在后台，可以使用 snap 命令行工具查询 snap 数据库， 显示已安装的 snap 包，以及安装、升级和删除 snap 包。
- 使用 snap version 命令检查 snap 是否正在运行：
    ```
    $ snap version
    ```
- 如果 snap 正在运行， 可以使用 snap list 命令查看当前已安装的 snap 应用程序列表：
    ```
    $ snap list
    ```
- snap find 命令可以在 snap 仓库中搜索指定程序：
    ```
    $ snap find solitaire
    ```
- snap info 命令可以查看 snap 应用程序（简称为 snap ）的详细信息：
    ```
    $ snap info solitaire
    ```
- snap install 命令可以安装新的 snap：
    ```
    $ sudo snap install solitaire
    ```
    - 注意，安装 snap 的时候必须有 root 权限，这意味着要用到 sudo 命令。
    - 在安装 snap 的时候， snapd 程序会将其作为驱动器挂载。可以使用 mount 命令查看新的snap 挂载。
- 如果需要删除某个 snap ，使用 snap remove 命令即可：
    ```
    $ sudo snap remove solitaire
    ```
- 也可以通过禁用来代替删除。使用 snap disable 命令即可。要想重新恢复 snap，可以使用 snap enable 命令。

22. 使用 flatpak 容器
- flatpak 应用程序容器格式是作为一个独立的开源项目创建的， 与任何特定的 Linux 发行版都 没有直接联系。
- 如果你的 Linux 发行版支持 flatpak，可以使用 flatpak list 命令列出已安装的应用程序容器：
    ```
    $ flatpak list
    ```
- flatpak search 命令可以在 flatpak 仓库中搜索指定应用程序：
    ```
    $ flatpak search solitaire
    Name         Description    Application ID      Version Branch  Remotes
    Aisleriot Solitaire     org.gnome.Aisleriot     stable  fedora
    ```
    - 使用容器时，必须使用其 Application ID 值，而不是名称。
- 可以使用 flatpak install 命令安装应用程序：
    ```
    $ sudo flatpak install org.gnome.Aisleriot
    ```
- 要想检查安装是否正确，可以再次使用 flatpak list 命令：
    ```
    $ flatpak list
    Name                    Application ID      Version Branch  Installation 
    Aisleriot Solitaire     org.gnome.Aisleriot         stable  system
    ```
- 可以使用 flatpak uninstall 命令删除应用程序容器：
    ```
    $ sudo flatpak uninstall org.gnome.Aisleriot
    ```

23. 从源代码安装
- 源代码形式的软件包通常以 tarball 的形式发布（经由 tar 命令创建出的归档文件通常称为 tarball）。
- 使用软件包 sysstat 作为示例。 sysstat 提供了各种系统监测工具，非常好用。
    ```
    // 首先，需要将 sysstat 的 tarball 下载到你的 Linux 系统中。尽管通常能在各种 Linux 网站 上找到 sysstat，但最好直接到程序的官方站点下载。
    $ tar -Jxvf sysstat-12.3.3.tar.xz
    $ cd sysstat-12.3.3
    $ ls
    // 在目录的列表中，应该能看到 README 文件或 INSTALL 文件。务必阅读这些文件，其中写明了软件安装所需的操作步骤。
    // 运行 configure 工具， 检查你的 Linux，确保拥有 合适的能够编译源代码的编译器，以及正确的库依赖关系
    // 如果有问题， 则 configure 会显示错误消息，说明缺失了哪些东西。
    $ ./configure
    // 用 make 命令来构建各种二进制文件。 make 命令会编译源代码，然后由链接器生 成最终的可执行文件。和 configure 命令一样， make 命令会在编译和链接所有源代码文件的 过程中产生大量的输出：
    $ make
    // make 命令结束后， 可运行的sysstat 程序就出现在目录中了。但是从这个目录中运行程序有 点儿不方便。你希望将其安装在 Linux 系统的常用位置。为此， 必须以 root 用户身份登录（或者 使用 sudo 命令）， 然后使用 make 命令的 install 选项：
    # make instal
    ```
- 大多数 Linux 程序是用 C 或 C++编程语言编写的。要在系统中编译这些源代码，需要安装 gcc 软件包和make 软件包。大多数Linux 桌面发行版默认没有安装。
