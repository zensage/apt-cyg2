<a id="top"></a>

apt-cyg2 / apt-cyg
==================

[English](#en) · [简体中文](#zh-cn)

---

<a id="en"></a>

## English

**[apt-cyg2](#apt-cyg2)** (version 2.0) · **[apt-cyg](#apt-cyg)** (version 1.0)

Both are command-line package managers for Cygwin. They cooperate with Cygwin
Setup (setup.exe), read the same `setup.rc`, share its download cache and keep
`/etc/setup/installed.db` in the same format, so packages installed by one can
be managed by the other.

---

<a id="apt-cyg2"></a>

### apt-cyg2

apt-cyg2 is a rewrite of apt-cyg 1.0. It keeps the single-script design and
the original operations, and adds what is needed to keep a Cygwin
installation up to date and consistent from the command line: upgrade,
reinstall, local package install, dependency-aware removal, autoremove and an
integrity check. It also fixes the defects of version 1 that could leave a
system half-installed or broken.

#### What's new in apt-cyg2

New operations

- **upgrade** – upgrade all installed packages, or only the named ones, to the
  current version in setup.ini. New dependencies are installed, and files the
  new version no longer ships are removed.
- **reinstall** – reinstall packages, keeping their manual/auto flag.
- **localinstall** – install package archives from the local disk. The name
  is taken from `NAME-VERSION-RELEASE.tar.*` (or `--name=`); dependencies are
  read from a `.hint` file next to the archive and installed from the mirror.
- **check** – verify installed.db, file manifests, dependencies and pending
  postinstall scripts; `--files` also checks every packaged file; `--fix`
  installs missing dependencies and reinstalls broken packages.
- **autoremove** – remove dependencies that nothing needs any more.

Safer installs and removals

- The full plan (new, upgraded, reinstalled packages) is shown first;
  `-s` / `--simulate` stops there.
- Every package is downloaded and checksum-verified before the first one is
  unpacked, so a failed download changes nothing.
- installed.db records whether a package was requested (manual) or pulled
  in as a dependency (auto), as setup.exe does.
- `remove` refuses to break packages that still depend on the target
  (override with `--force`); essential packages are protected.
- installed.db is written atomically, and a lock prevents two runs from
  changing the system at the same time.
- Postinstall scripts run in setup.exe order (`0p_*`, one-shot, `zp_*`),
  including `.dash`, `.bat` and `.cmd` scripts; perpetual scripts such as
  autorebase run after every install.
- Every change is logged to `/var/log/apt-cyg.log`.

Faster and more compatible

- Dependencies are read from `depends2:` (newer setup.ini) as well as
  `requires:`.
- The package list is fetched as `setup.zst`, `setup.xz` or `setup.bz2`, and
  an unchanged list is neither downloaded nor unpacked again. Update messages
  show the full path of the setup.ini in use.
- setup.ini is indexed once, so queries no longer rescan the whole file.
- `search` matches exact paths or file names (`ls` no longer matches
  `lsattr`).
- wget, curl or lynx can download; `file://` mirrors are accepted.

The full list of new features and bug fixes is in
[CHANGES-apt-cyg2.txt](CHANGES-apt-cyg2.txt).

#### Operations

~~~
install       Install package(s) and their dependencies.
localinstall  Install package archive file(s) from the local disk.
reinstall     Reinstall installed package(s).
upgrade       Upgrade all (or the named) installed packages.
remove        Remove package(s); refuses if other packages depend on them.
autoremove    Remove dependency packages that nothing needs any more.
check         Verify the installation (database, dependencies, manifests).
update        Download a fresh copy of the package list (setup.ini).
download      Retrieve package(s) into the cache, do not install.
show          Display information on given package(s).
depends       Produce a dependency tree for a package.
rdepends      Produce a tree of packages that depend on the named package.
list          List installed packages, optionally matching regexp(s).
listall       List available packages matching regexp(s).
category      Display all packages that are members of a named category.
listfiles     List all files owned by given package(s).
search        Find installed packages that own the specified file(s).
searchall     Search cygwin.com for packages that contain the file(s).
mirror        Set the mirror URL, or display it if none is given.
cache         Set the package cache directory, or display it.
~~~

#### Options

~~~
--nodeps      Skip all dependency handling.
--noscripts   Do not run postinstall / preremove scripts.
-s, --simulate, --dry-run
              Show what would be done, change nothing.
-y, --yes     Do not ask for confirmation.
--force       Remove even if other packages depend on the target;
              allow replacing the running cygwin package.
--fix         With check: install missing dependencies, reinstall
              broken packages.
--files       With check: also verify that every packaged file exists.
--name=PKG    With localinstall: package name, when the file name does not
              follow the NAME-VERSION-RELEASE.tar.* convention.
--version     Display version and exit.
-h, --help    Display this help and exit.
~~~

Environment variables, mainly for testing:

~~~
APT_CYG_ROOT        Operate on an alternate root directory.
APT_CYG_ARCH        Override the architecture (x86_64 or x86).
APT_CYG_DOWNLOADER  Force the downloader: wget, curl or lynx.
~~~

#### Requirements

bash 4.2+, gawk, tar, gzip, xz and bzip2, plus wget, curl or lynx. zstd is
used when present. All of these are in a standard Cygwin installation except
possibly wget/curl and zstd. Cygwin Setup must have been run once, so that
`/etc/setup/setup.rc` names a mirror and a cache directory (or set them with
`apt-cyg2 mirror URL` and `apt-cyg2 cache DIR`).

#### Installation

apt-cyg2 is a single script. Download it into `/usr/bin` and make it
link2: https://cdn.jsdelivr.net/gh/zensage/apt-cyg2@master/apt-cyg2
executable:

    curl https://raw.githubusercontent.com/zensage/apt-cyg2/refs/heads/master/apt-cyg2 -o /usr/bin/apt-cyg2
    curl https://cdn.jsdelivr.net/gh/zensage/apt-cyg2@master/apt-cyg2 -o /usr/bin/apt-cyg2
    chmod +x /usr/bin/apt-cyg2

Without curl, use wget instead:

    wget https://raw.githubusercontent.com/zensage/apt-cyg2/refs/heads/master/apt-cyg2 -O /usr/bin/apt-cyg2
    chmod +x /usr/bin/apt-cyg2

To get the test suite as well:

    curl https://raw.githubusercontent.com/zensage/apt-cyg2/refs/heads/master/apt-cyg2-test.sh -o apt-cyg2-test.sh

Check the installation:

    apt-cyg2 --version

#### Quick start

Example use of apt-cyg2:

    apt-cyg2 update
    apt-cyg2 install nano
    apt-cyg2 upgrade -s          # preview the upgrade
    apt-cyg2 upgrade
    apt-cyg2 localinstall ./mytool-1.0-1.tar.xz
    apt-cyg2 remove nano
    apt-cyg2 autoremove
    apt-cyg2 check --files

#### Tests

`apt-cyg2-test.sh` builds a fake mirror and a fake root in a temporary
directory and runs every operation against them. It never touches the live
system, so it is safe to run on Cygwin:

    bash apt-cyg2-test.sh ./apt-cyg2

The HTTP mirror tests run when python3 is installed; the searchall test runs
when cygwin.com is reachable (set `APT_CYG_TEST_OFFLINE=1` to skip it).

---

<a id="apt-cyg"></a>

### apt-cyg

Version 1.0 · [back to top](#top)

apt-cyg is a Cygwin package manager. It includes a command-line installer for
Cygwin which cooperates with Cygwin Setup and uses the same repository.

[github.com/transcode-open/apt-cyg][1]

[1]:https://github.com/transcode-open/apt-cyg

#### Operations

~~~
install
  Install package(s).

remove
  Remove package(s) from the system.

update
  Download a fresh copy of the master package list (setup.ini) from the
  server defined in setup.rc.

download
  Retrieve package(s) from the server, but do not install/upgrade anything.

show
  Display information on given package(s).

depends
  Produce a dependency tree for a package.

rdepends
  Produce a tree of packages that depend on the named package.

list
  Search each locally-installed package for names that match regexp. If no
  package names are provided in the command line, all installed packages will
  be queried.

listall
  This will search each package in the master package list (setup.ini) for
  names that match regexp.

category
  Display all packages that are members of a named category.

listfiles
  List all files owned by a given package. Multiple packages can be specified
  on the command line.

search
  Search for downloaded packages that own the specified file(s). The path can
  be relative or absolute, and one or more files can be specified.

searchall
  Search cygwin.com to retrieve file information about packages. The provided
  target is considered to be a filename and searchall will return the
  package(s) which contain this file.
~~~

#### Quick start

apt-cyg is a simple script. To install:

    lynx -source rawgit.com/transcode-open/apt-cyg/master/apt-cyg > apt-cyg
    install apt-cyg /bin

Example use of apt-cyg:

    apt-cyg install nano

---

<a id="zh-cn"></a>

## 简体中文

[English](#en) · [返回页顶](#top)

**[apt-cyg2](#apt-cyg2-zh)**（2.0 版） · **[apt-cyg](#apt-cyg-zh)**（1.0 版）

两者都是 Cygwin 的命令行包管理器。它们与 Cygwin 安装程序（setup.exe）协同工作：
读取同一个 `setup.rc`，共用它的下载缓存，并以相同格式维护
`/etc/setup/installed.db`，因此用其中一个安装的包，可以用另一个来管理。

---

<a id="apt-cyg2-zh"></a>

### apt-cyg2

apt-cyg2 是 apt-cyg 1.0 的重写版本。它保留了单脚本的设计和原有的全部命令，
并补齐了在命令行下让 Cygwin 保持最新、保持一致所需的功能：升级、重装、
本地包安装、带依赖检查的卸载、自动清理和完整性检查。同时修复了 1.0 版中
可能导致系统装到一半或被装坏的缺陷。

#### apt-cyg2 的新功能

新命令

- **upgrade**：把全部已安装的包（或指定的包）升级到 setup.ini 中的当前版本。
  会补装新版本新增的依赖，并删除新版本已不再包含的文件。
- **reinstall**：重装包，保留其手动/自动安装标记。
- **localinstall**：从本地磁盘安装包归档。包名取自 `NAME-VERSION-RELEASE.tar.*`
  （或用 `--name=` 指定）；依赖从归档旁的 `.hint` 文件读取，并从镜像安装。
- **check**：检查 installed.db、文件清单、依赖以及尚未执行的 postinstall 脚本；
  `--files` 还会逐一检查包内文件是否存在；`--fix` 会补装缺失的依赖并重装损坏的包。
- **autoremove**：卸载已不再被任何包需要的依赖包。

更安全的安装与卸载

- 先显示完整的执行计划（新装、升级、重装的包）；
  `-s` / `--simulate` 只显示计划，不做任何改动。
- 所有包先全部下载并校验通过，才开始解包第一个包，下载失败时系统保持原样。
- installed.db 与 setup.exe 一样，记录包是用户指定安装（手动）还是作为依赖
  装入（自动）。
- `remove` 拒绝卸载仍被其他包依赖的包（可用 `--force` 强制）；核心包受保护。
- installed.db 以原子方式写入，并用锁防止两个进程同时修改系统。
- postinstall 脚本按 setup.exe 的顺序执行（`0p_*`、一次性脚本、`zp_*`），
  包括 `.dash`、`.bat` 和 `.cmd` 脚本；autorebase 这类永久脚本在每次安装后都会执行。
- 每次变更都记录到 `/var/log/apt-cyg.log`。

更快、更兼容

- 依赖既读取 `requires:`，也读取新版 setup.ini 使用的 `depends2:`。
- 包列表依次尝试 `setup.zst`、`setup.xz`、`setup.bz2`；列表没有变化时，
  既不重新下载也不重新解压。更新信息会显示所用 setup.ini 的完整路径。
- setup.ini 只建一次索引，查询不再反复扫描整个文件。
- `search` 精确匹配路径或文件名（`ls` 不再误匹配 `lsattr`）。
- 可用 wget、curl 或 lynx 下载；支持 `file://` 镜像。

新功能与修复的完整清单见 [CHANGES-apt-cyg2.txt](CHANGES-apt-cyg2.txt)（英文）。

#### 命令

~~~
install       安装包及其依赖。
localinstall  从本地磁盘安装包归档文件。
reinstall     重装已安装的包。
upgrade       升级全部（或指定的）已安装包。
remove        卸载包；若有其他包依赖它则拒绝。
autoremove    卸载已不再被需要的依赖包。
check         检查安装状态（数据库、依赖、文件清单）。
update        重新下载包列表（setup.ini）。
download      把包下载到缓存，不安装。
show          显示指定包的信息。
depends       显示一个包的依赖树。
rdepends      显示依赖指定包的包树。
list          列出已安装的包，可按正则表达式筛选。
listall       列出可用包中名称匹配正则表达式的包。
category      显示指定分类下的全部包。
listfiles     列出指定包所包含的全部文件。
search        查找哪些已安装的包包含指定文件。
searchall     在 cygwin.com 上查找包含指定文件的包。
mirror        设置镜像地址；不给参数时显示当前镜像。
cache         设置包缓存目录；不给参数时显示当前目录。
~~~

#### 选项

~~~
--nodeps      不处理任何依赖。
--noscripts   不执行 postinstall / preremove 脚本。
-s, --simulate, --dry-run
              只显示将要执行的操作，不做任何改动。
-y, --yes     不询问确认。
--force       即使有其他包依赖也照样卸载；
              允许替换正在运行的 cygwin 包。
--fix         配合 check：补装缺失的依赖，重装损坏的包。
--files       配合 check：同时检查每个包内文件是否存在。
--name=PKG    配合 localinstall：文件名不符合 NAME-VERSION-RELEASE.tar.*
              规则时，指定包名。
--version     显示版本并退出。
-h, --help    显示帮助并退出。
~~~

环境变量（主要用于测试）：

~~~
APT_CYG_ROOT        在另一个根目录下操作。
APT_CYG_ARCH        指定架构（x86_64 或 x86）。
APT_CYG_DOWNLOADER  指定下载工具：wget、curl 或 lynx。
~~~

#### 运行要求

bash 4.2 以上、gawk、tar、gzip、xz 和 bzip2，以及 wget、curl、lynx 三者之一。
装有 zstd 时会优先使用。除 wget/curl 和 zstd 可能需要另装外，其余都在 Cygwin
的标准安装中。需要先运行过一次 Cygwin 安装程序，使 `/etc/setup/setup.rc`
中有镜像和缓存目录的设置（也可以用 `apt-cyg2 mirror URL` 和
`apt-cyg2 cache DIR` 设置）。

#### 安装

apt-cyg2 只有一个脚本文件。下载到 `/usr/bin` 并加上执行权限：

    curl https://raw.githubusercontent.com/zensage/apt-cyg2/refs/heads/master/apt-cyg2 -o /usr/bin/apt-cyg2
    chmod +x /usr/bin/apt-cyg2

没有 curl 时，可以改用 wget：

    wget https://raw.githubusercontent.com/zensage/apt-cyg2/refs/heads/master/apt-cyg2 -O /usr/bin/apt-cyg2
    chmod +x /usr/bin/apt-cyg2

如需同时获取测试集：

    curl https://raw.githubusercontent.com/zensage/apt-cyg2/refs/heads/master/apt-cyg2-test.sh -o apt-cyg2-test.sh

检查是否安装成功：

    apt-cyg2 --version

#### 快速上手

apt-cyg2 的用法示例：

    apt-cyg2 update
    apt-cyg2 install nano
    apt-cyg2 upgrade -s          # 预览升级
    apt-cyg2 upgrade
    apt-cyg2 localinstall ./mytool-1.0-1.tar.xz
    apt-cyg2 remove nano
    apt-cyg2 autoremove
    apt-cyg2 check --files

#### 测试

`apt-cyg2-test.sh` 会在临时目录里搭建模拟镜像和模拟根目录，并对其执行全部命令。
它不会触及正在使用的系统，因此可以放心在 Cygwin 上运行：

    bash apt-cyg2-test.sh ./apt-cyg2

装有 python3 时会运行 HTTP 镜像测试；能访问 cygwin.com 时会运行 searchall 测试
（设置 `APT_CYG_TEST_OFFLINE=1` 可跳过）。

---

<a id="apt-cyg-zh"></a>

### apt-cyg

1.0 版 · [返回页顶](#top)

apt-cyg 是一个 Cygwin 包管理器。它提供了一个命令行安装工具，与 Cygwin
安装程序协同工作，并使用同一个软件仓库。

[github.com/transcode-open/apt-cyg][1]

[1]:https://github.com/transcode-open/apt-cyg

#### 命令

~~~
install
  安装包。

remove
  从系统中卸载包。

update
  从 setup.rc 中指定的服务器重新下载主包列表（setup.ini）。

download
  从服务器获取包，但不安装或升级任何内容。

show
  显示指定包的信息。

depends
  显示一个包的依赖树。

rdepends
  显示依赖指定包的包树。

list
  在本地已安装的包中查找名称匹配正则表达式的包。若命令行未提供包名，
  则列出全部已安装的包。

listall
  在主包列表（setup.ini）中查找名称匹配正则表达式的包。

category
  显示指定分类下的全部包。

listfiles
  列出指定包所包含的全部文件。命令行中可以同时指定多个包。

search
  查找哪些已下载的包包含指定文件。路径可以是相对路径或绝对路径，
  可以同时指定一个或多个文件。

searchall
  在 cygwin.com 上查询包的文件信息。给出的目标视为文件名，searchall
  会返回包含该文件的包。
~~~

#### 快速上手

apt-cyg 只是一个简单的脚本。安装方法：

    lynx -source rawgit.com/transcode-open/apt-cyg/master/apt-cyg > apt-cyg
    install apt-cyg /bin

apt-cyg 的用法示例：

    apt-cyg install nano
