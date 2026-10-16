apt-cyg2 / apt-cyg
==================

**[apt-cyg2](#apt-cyg2)** (version 2.0) · **[apt-cyg](#apt-cyg)** (version 1.0)

Both are command-line package managers for Cygwin. They cooperate with Cygwin
Setup (setup.exe), read the same `setup.rc`, share its download cache and keep
`/etc/setup/installed.db` in the same format, so packages installed by one can
be managed by the other.

---

<a id="apt-cyg2"></a>

apt-cyg2
--------

apt-cyg2 is a rewrite of apt-cyg 1.0. It keeps the single-script design and
the original operations, and adds what is needed to keep a Cygwin
installation up to date and consistent from the command line: upgrade,
reinstall, local package install, dependency-aware removal, autoremove and an
integrity check. It also fixes the defects of version 1 that could leave a
system half-installed or broken.

### What's new in apt-cyg2

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

See the commit history or `CHANGES-apt-cyg2.txt` for the full list of new
features and bug fixes.

### Operations

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

### Options

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

### Requirements

bash 4.2+, gawk, tar, gzip, xz and bzip2, plus wget, curl or lynx. zstd is
used when present. All of these are in a standard Cygwin installation except
possibly wget/curl and zstd. Cygwin Setup must have been run once, so that
`/etc/setup/setup.rc` names a mirror and a cache directory (or set them with
`apt-cyg2 mirror URL` and `apt-cyg2 cache DIR`).

### Quick start

apt-cyg2 is a single script. To install:

    install apt-cyg2 /bin

Example use of apt-cyg2:

    apt-cyg2 update
    apt-cyg2 install nano
    apt-cyg2 upgrade -s          # preview the upgrade
    apt-cyg2 upgrade
    apt-cyg2 localinstall ./mytool-1.0-1.tar.xz
    apt-cyg2 remove nano
    apt-cyg2 autoremove
    apt-cyg2 check --files

### Tests

`apt-cyg2-test.sh` builds a fake mirror and a fake root in a temporary
directory and runs every operation against them. It never touches the live
system, so it is safe to run on Cygwin:

    bash apt-cyg2-test.sh ./apt-cyg2

The HTTP mirror tests run when python3 is installed; the searchall test runs
when cygwin.com is reachable (set `APT_CYG_TEST_OFFLINE=1` to skip it).

---

<a id="apt-cyg"></a>

apt-cyg
-------

Version 1.0 · [back to top](#apt-cyg2--apt-cyg)

apt-cyg is a Cygwin package manager. It includes a command-line installer for
Cygwin which cooperates with Cygwin Setup and uses the same repository.

[github.com/transcode-open/apt-cyg][1]

[1]:https://github.com/transcode-open/apt-cyg

### Operations

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

### Quick start

apt-cyg is a simple script. To install:

    lynx -source rawgit.com/transcode-open/apt-cyg/master/apt-cyg > apt-cyg
    install apt-cyg /bin

Example use of apt-cyg:

    apt-cyg install nano
