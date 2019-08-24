---
priority: 0.6
title: 如何開始打包第一個 Debian 套件
excerpt: How to start your first Debian package
categories: topics
background-image: works-sample.png
tags:
  - featured
---

下面提供了第一次打包 Debian 套件的流程，新手可以參考這些步驟，來打包出你的第一個 Debian 套件！

指引中所使用的套件並不是唯一選擇，你可以自己參考其他資源，選擇最適合自己的工具。

## 前置作業

### 了解套件狀態

可以到 [WNPP](https://www.debian.org/devel/wnpp/)(Work-Needing and Prospective Packages) 的列表查看套件的狀態是什麼。

也可以使用 wnpp-check 這個套件來查看：

```sh
wnpp-check [套件名]
```

## 認領套件

認領套件的方法是把套件狀態改為 ITP(Intent to Package)，可以使用指令的方式 `reportbug wnpp`，也可以使用寄信的方式，下面會介紹寄信的方式。

認領套件有兩種情況：

### 套件並不存在 WNPP 上面

- 信件主旨格式：`ITP: [套件名] - 簡述`
- 收件者：`submit@bugs.debian.org`
- 內容格式：要符合規範，可以參考下面範例

```sh
# 這是註解：owner填上自己的名字與 email
Package: wnpp
Severity: wishlist
Owner: Chen-Ying Kuo <evshary@gmail.com>

* Package name    : tzdiff
  Version         : 0.9
  Upstream Author : minmin <belgianbeer@aj.admwt.jp>
* URL             : https://github.com/belgianbeer/tzdiff
* License         : BSD-2-Clause
  Programming Lang: shell script
  Description     : Show Timezone differences with local time in CLI (shell script).
```

### 認領既有套件

如果要認領既有套件其實很簡單，首先先找到你要認領套件的 WNPP 頁面，確定 bug number。

- 信件主旨：沒有限制，可以一樣是 `ITP: [套件名] - 簡述`
- 收件者：`[bug number]@bugs.debian.org`
- 內容格式：主要是要使用 Control 指令改變 title 和 owner(Debian 會自己爬蟲內容)，可參考下面範例

```sh
# 這是註解：-1 的地方代表延續 bug number
# 這是註解：retitle 最主要是改變狀態，變成 ITP
# 這是註解：owner 填上自己代表要打包
Control: retitle -1 ITP: checksec -- Bash script to test executable properties
Control: owner -1  Chen-Ying Kuo <evshary@gmail.com>

# 這是註解：可以表明自己要打包這個套件
I want to package this.
```

## 安裝打包的必要套件

```sh
sudo apt-get install build-essential fakeroot devscripts git git-buildpackage debhelper debmake lintian pbuilder
```

如果是要打包給 SID 版本(unstable Debian)使用的套件，可能結果會有些版本上的問題，所以建議可以安裝 SID 版本的打包套件。

`debhelper`, `debmake` 用來協助打包套件，`lintian` 則可以檢查套件內是否有不符合打包規範的錯誤或警告。

1. 修改 sources.list

    ```sh
    vim /etc/apt/sources.list

    # 加上以下這行
    deb http://ftp.de.debian.org/debian sid main
    ```

1. 執行下面指令來安裝 SID 版本套件

    ```sh
    sudo apt-get update

    # 確認可用版本
    sudo apt-cache policy debhelper
    sudo apt-cache policy debmake
    sudo apt-cache policy lintian

    # 更新版本
    sudo apt-get -t sid install debhelper debmake lintian
    ```

## 打包套件

### 建立 package repository

Debian 套件都是透過 [Salsa](https://salsa.debian.org/) GitLab 做管理，如果沒有帳號要申請一個。

進入後就建立一個名稱跟套件名相同的 repository。若已有前人的 repository，直接 fork 出來。

通常我們會建立一個名為 `debian/master` 的 branch 來當我們打包的主要分支。

### 打包前置作業

1. 先在上層目錄(與套件目錄平行)抓下 upstream(github 上該套件的 release)的 tar 檔

   - 通常我們選擇 upstream 最新的 release 版本，名稱應該會是 [版本].tar.gz

1. 利用 tar 檔產生初始 branch，然後 gbp 會詢問你package 的名稱

```sh
gbp import-orig --pristine-tar ../[版本].tar.gz --debian-branch=debian/master -u [版本]
```

1. 完成後我們就有套件初步的 repository 了

   - repository 裡面會有三個 branch(debian/master, pristine-tar, upstream)
   - 這三個 branch 分別代表的意義如下
      1. upstream: 原作者的 source
      1. pristine-tar: 利用原作者的 source 產生出來的tar檔
      1. debian/master: 除了 upstream 的 source 外，還包括 Debian 打包文件的資料夾，這也是我們主要修改上傳資訊的地方
   - 上層資料夾會有 [package_name]_[version].orig.tar.gz 的檔案

### 產生 Debian 套件所需內容

接著我們會執行 `debmake`，產生 Debian 的資料夾，裡面的相關檔案就是打包套件所需的資訊

```sh
# 把剛剛創的資料夾加上版號
mv [repository] [repository]-[版號]
cd [repository]-[版號]
debmake
```

### 修改 Debian 相關檔案

修改的規則可參考 [Debian Policy Manual](https://www.debian.org/doc/debian-policy/)，下面記錄的內容僅供參考，因為 policy 會變，還是要以 manual 為主，以下是需要修改的檔案。

#### debian/copyright

描述套件的著作權相關內容。

- copyright 內容要小心，必須按照原作者的 copyright，通常 `demake` 會幫忙判斷版權，不過如果有錯誤還是得自己修正。

- 如果有原作者自己定義的 License 的話，可以參考 [zsh](https://salsa.debian.org/debian/zsh) 的改法

- 幾個可參考的連結

  - <https://salsa.debian.org/debian/nnn/blob/debian/master/debian/copyright>
  - <https://salsa.debian.org/debian/skypat/blob/debian/master/debian/copyright>
  - <https://gist.github.com/kasramp/3ef3e1e4b323d8b3a74551e688c2988f>

#### debian/control

描述套件細節與相依性。

1. Section 選擇應有分類，通常選 misc，可以參考[官網分類](https://www.debian.org/doc/debian-policy/ch-archive.html#s-subsections)

1. 其餘如 `Maintainer` 的欄位也要修改，`Description` 寫上官網和敘述

1. 如果有套件相依性的話，需要在 `Depends` 上面加上去，讓 debuild 和 pubuilder 執行時自己裝

   - 不過有些套件不用寫上去，除了沒用到的以外還有 essential，可用這個指令 `dpkg-query -Wf '${Package;-40}${Essential}\n' | grep yes` 確定哪些是 essential，可參考 [Finding all “essential” packages with apt](https://unix.stackexchange.com/questions/73242/finding-all-essential-packages-with-apt)
   - 如果不清楚相依套件或版本的話，可能就需要詢問上游了。不過如果沒有特別需求，版本一般可以不用填
   - 有些 shell 裡面會寫有目前使用的指令，如果要反查是什麼套件的話可用如下指令(參考自 [Command to find the source package of a binary?](https://superuser.com/questions/146875/command-to-find-the-source-package-of-a-binary))

    ```sh
    dpkg -S `which COMMANDHERE`
    ```

1. 記得要補上 VCS 路徑

#### debian/changelog

套件修改歷史。

1. 這邊會用到 [dch](https://www.commandlinux.com/man-page/man1/dch.1.html)，可使用 `dch -i` 來修改 changelog 檔案，基本上一次上傳只進一版(第一次的上傳所有修改都只會是 x.x.x-1)

2. 有幾個地方要修改一下

- unknown 要改成 unstable
- bug number 要改為自己的 ITP number
- 修改自己的名字與 email

#### debian/rules

套件自動編譯相關的設定。

1. 通常原始程式碼的 Makefile 不會是我們要的，例如 upstream 通常會將 binary 安裝到 `/usr/local/bin` (我們沒有權限可以做這件事)，此時就必須要修改這個檔案。

1. 記住要把 debmake 產生的註解拿掉。

#### debian/patch

因應 Debian 套件所做的修正。

1. 若有更動原始碼的需求，可以使用 `gbp` 打 patch，使用時記得在參數加上 `--no-patch-num` ，來去除 patch number

    ```sh
    gbp clone --pristine-tar [Salsa的git位置] delay_package_gbm

    gbp pq import --no-patch-num

    # 此時會切換到patch branch，開始做必要的更動並commit
    git add ${modified files}
    git ci -m "Fix xxx for debian package"
    
    gbp pq export --no-patch-num
    ```

#### debian/[pkg_name].manpages

套件的 manpage。

1. 若無manpage，請在 github 上開 issue 請原作者加入，或參考別的 package 加入後回饋給上游

1. 在 `debian/[pkg_name].manpages` 內新增 manpage 的位置，例如`[package].1`

1. 如果想要看 manpage 長什麼樣子，可用 `nroff -man [package].1` 測試

#### debian/tests

套件的自動測試設定，如何修改可以參考 [autopkgtest: Automatic testing for packages](http://packaging.ubuntu.com/html/auto-pkg-test.html) 和 [Tutorial: Functional Testing of Debian Packages](https://annex.debconf.org/debconf-share/debconf15/slides/173-tutorial-functional-testing-of-debian-packages.pdf)

1. 首先新增 `debian/tests` 資料夾

    ```sh
    mkdir debian/tests
    ```

1. 在 `debian/tests` 底下增加 control 這個檔案，內容如下

    - Tests 右邊的 build 代表我們要測試的 script file
    - Depends 右邊的 @ 代表相依檔案只有自己，如果有其他就加在這行
    - 可參考[別人的範例](https://github.com/HypoPG/hypopg/blob/master/debian/tests/control)

    ```sh
    Tests: build
    Depends: @
    ```

1. 接著新增 debina/tests/build

    ```sh
    #!/bin/sh

    set -e
    echo "run start"
    # 要做的測試
    echo "run: OK"
    ```

#### debian/watch

監控上游是否有新的 release 的設定，如何修改 watch 檔案，可參考[官網介紹](https://wiki.debian.org/debian/watch)或[這邊的範例](https://salsa.debian.org/debian/buku/blob/debian/master/debian/watch)

1. 如範例新增 `debian/watch` 檔案

    ```sh
    version=4
    opts=\
    repacksuffix=+ds,\
    repack,compression=xz,\
    dversionmangle=s/\+(debian|dfsg|ds|deb)(\.?\d+)?$//,\
    filenamemangle=s%(?:.*?)?v?(\d[\d.]*)\.tar\.gz%<project>-$1.tar.gz% \
    https://github.com/jarun/Buku/releases \
    (?:.*?/)?v?(\d[\d.]*)\.tar\.gz debian uupdate
    ```

1. 新增完畢可用 uscan 測試

    ```sh
    # 測試方式
    uscan --no-download --verbose --debug
    ```

#### debian/upstream

保留上游資訊的檔案。

1. 新增 `debian/upstream` 資料夾

    ```sh
    mkdir debian/upstream
    ```

1. 在 upstream 資料夾中新增 metadata，相關資訊，可參考[別人的範例](https://salsa.debian.org/debian/skypat/blob/debian/master/debian/upstream/metadata)

    ```sh
    Bug-Database: https://github.com/skymizer/SkyPat/issues
    Bug-Submit: https://github.com/skymizer/SkyPat/issues/new
    Name: skypat
    Repository: https://github.com/skymizer/SkyPat.git
    Repository-Browse: https://github.com/skymizer/SkyPat
    ```

#### debian/install

設定套件要安裝在哪裡的設定

1. 新增 `debian/install`

    ```sh
    vi debian/install
    ```

1. 增加要搬移的位置，通常為 `usr/bin`

    ```sh
    [執行檔] usr/bin
    ```

#### debian/[package_name].docs

要記得創立 debian/[package_name].docs，主要是要把 `README.md` 包入 package 內。

### 整理 git log

最後修改完相關 Debian 檔案後，git log 要整理一下。若是第一次上套件，git log 只留一筆，內容可以寫 `Initialize packaging`。

### 執行 debuild 打包套件

改完 `debian/` 相關檔案後，就可以開始用 debuild 打包，不過我們不是 Debian developer，沒有資格上傳套件，所以可以跳過 sign key 這個動作。

1. 執行前我們會先 debuild 的設定檔，檔案位於 `~/.devscripts` 或 `/etc/devscripts.conf`

    ```sh
    DEBUILD_LINTIAN=yes

    # -E include experimental tags
    DEBUILD_LINTIAN_OPTS="-i -EvIL +pedantic --show-overrides"

    #-uc -us代表不要sign key
    DEBUILD_DPKG_BUILDPACKAGE_OPTS="-uc -us --changes-option=-sa"

    # DEBSIGN_KEYID可填可不填
    DEBSIGN_KEYID="FIXME - YOUR KEYID"

    DEB_BUILD_OPTIONS="parallel=10"
    ```

1. 接下來在套件目錄下，執行 `debuild`

    ```sh
    debuild
    ```

    執行完成後會在自己資料夾的上一層產生許多檔案，包括[套件名稱].dsc，這就是所謂的 Debian source control files，之後如果要跑 pubuilder 來驗證套件正確性時需要用到。

1. 由於`debuild` 會產生許多檔案，這些都是不用上 Git的。如果想要清除的話，可以用下面指令

    ```sh
    debuild -- clean
    ```

### 使用 Lintian 檢查套件的問題

我們在 `debuild` 中有加入 `Lintian` 的檢查，跑完後可能會出現 Lintian 相關的錯誤，最好把這些錯誤與警告解決再上傳套件。

其中一些常見錯誤的解法可參考 [Solutions to some lintian warnings](https://hackmd.io/tsRaPMWTQACNbFZ5v05O-w)

## 驗證套件

為了確認打包結果是否可以在一個乾淨的環境上執行，以及是否能通用在 32/64 bit 的系統上，所以要經過下面的測試。

### 使用 pbuilder 建立 sid 環境

使用 `pubuilder` 最主要的原因是 `debuild` 是在自己的開發環境下執行，可能已經安裝了某些相依性套件，`pbuilder` 可以模擬一個乾淨的環境供我們測試是否有相依性或其他問題。

1. 安裝 `pbuilder` 後，首先編輯 `/etc/pbuilderrc` 修改設定

    ```sh
    # this is your configuration file for pbuilder.
    # the file in /usr/share/pbuilder/pbuilderrc is the default template.
    # /etc/pbuilderrc is the one meant for overwriting defaults in
    # the default template
    #
    # read pbuilderrc.5 document for notes on specific options.
    MIRRORSITE=http://deb.debian.org/debian
    # the hook dir may already be set/populated!
    HOOKDIR="/var/cache/pbuilder/hooks/"
    if [ -n "$DEPS" ] ; then
            export DEPSBASE=/var/cache/pbuilder/deps
            BINDMOUNTS=$DEPSBASE
    fi
    ```

1. `pbuilder` 執行時，會執行 hook 資料夾下設定的hook script，通常我們會加入一個 Lintian hook 來做檢查。

    - 在 `/var/cache/pbuilder/hooks/B90lintian` 中輸入下面內容，相關範例可在 `/usr/share/doc/pbuilder/examples` 找到。每個不同功能的hook都有制式的名字，檔名不能隨意輸入

    ```sh
    #!/bin/dash
    set -e

    install_packages() {
            apt-get -y --allow-downgrades install "$@"
    }

    install_packages lintian

    echo "+++ lintian output +++"
    su -c "lintian -i -EvIL +pedantic --show-overrides /tmp/buildd/*.changes" - pbuilder
    # use this version if you don't want lintian to fail the build
    #su -c "lintian -i -I --show-overrides /tmp/buildd/*.changes; :" - pbuilder
    echo "+++ end of lintian output +++"
    ```

    - 打開 `/var/cache/pbuilder/hooks/B90lintian` 執行權限打開

    ```sh
    chmod a+x /var/cache/pbuilder/hooks/B90lintian
    ```

1. 由於我們要在 SID 的環境執行(也就是不穩定的 Debian 版本)，可參考[這裡的設定](https://wiki.debian.org/PbuilderTricks)，先創立 `~/.pbuilderrc` (通常應該是 `/root/.pbuilderrc` )，然後貼上下面的內容

    ```sh
        # Codenames for Debian suites according to their alias. Update these when
        # needed.
        UNSTABLE_CODENAME="sid"
        TESTING_CODENAME="bullseye"
        STABLE_CODENAME="buster"
        OLDSTABLE_CODENAME="stretch"
        OLDOLDSTABLE_CODENAME="jessie"
        STABLE_BACKPORTS_SUITE="$STABLE_CODENAME-backports"

        # List of Debian suites.
        DEBIAN_SUITES=($UNSTABLE_CODENAME $TESTING_CODENAME $STABLE_CODENAME $STABLE_BACKPORTS_SUITE
            "experimental" "unstable" "testing" "stable")

        # List of Ubuntu suites. Update these when needed.
        UBUNTU_SUITES=("xenial" "wily" "vivid" "utopic" "trusty")

        # Mirrors to use. Update these to your preferred mirror.
        DEBIAN_MIRROR="ftp.debian.org"
        UBUNTU_MIRROR="mirrors.kernel.org"

        # Optionally use the changelog of a package to determine the suite to use if
        # none set.
        if [ -z "${DIST}" ] && [ -r "debian/changelog" ]; then
            DIST=$(dpkg-parsechangelog --show-field=Distribution)
        fi

        # Optionally set a default distribution if none is used. Note that you can set
        # your own default (i.e. ${DIST:="unstable"}).
        : ${DIST:="$(lsb_release --short --codename)"}

        # Optionally change Debian codenames in $DIST to their aliases.
        case "$DIST" in
            $UNSTABLE_CODENAME)
                DIST="unstable"
                ;;
            $TESTING_CODENAME)
                DIST="testing"
                ;;
            $STABLE_CODENAME)
                DIST="stable"
                ;;
        esac

        # Optionally set the architecture to the host architecture if none set. Note
        # that you can set your own default (i.e. ${ARCH:="i386"}).
        : ${ARCH:="$(dpkg --print-architecture)"}
        NAME="$DIST"
        if [ -n "${ARCH}" ]; then
            NAME="$NAME-$ARCH"
            DEBOOTSTRAPOPTS=("--arch" "$ARCH" "${DEBOOTSTRAPOPTS[@]}")
        fi

        BASETGZ="/var/cache/pbuilder/$NAME-base.tgz"
        DISTRIBUTION="$DIST"
        BUILDRESULT="/var/cache/pbuilder/$NAME/result/"
        APTCACHE="/var/cache/pbuilder/$NAME/aptcache/"
        BUILDPLACE="/var/cache/pbuilder/build/"
        HOOKDIR="/var/cache/pbuilder/hooks/"

        if $(echo ${DEBIAN_SUITES[@]} | grep -q $DIST); then
            # Debian configuration
            MIRRORSITE="http://$DEBIAN_MIRROR/debian/"
            COMPONENTS="main contrib non-free"
            if $(echo "$STABLE_CODENAME stable" | grep -q $DIST); then
                OTHERMIRROR="$OTHERMIRROR | deb $MIRRORSITE $STABLE_BACKPORTS_SUITE $COMPONENTS"
            fi
        elif $(echo ${UBUNTU_SUITES[@]} | grep -q $DIST); then
            # Ubuntu configuration
            MIRRORSITE="http://$UBUNTU_MIRROR/ubuntu/"
            COMPONENTS="main restricted universe multiverse"
        else
            echo "Unknown distribution: $DIST"
            exit 1
        fi

        PBUILDERSATISFYDEPENDSCMD="/usr/lib/pbuilder/pbuilder-satisfydepends-apt"
    ```

1. 透過 `pbuilder` 建立 i386/amd64 兩個環境

    ```sh
    sudo OS=debian DIST=sid ARCH=i386 pbuilder create
    sudo OS=debian DIST=sid ARCH=amd64 pbuilder create
    ```

    P.S. 若有更動 `/root/.pbuilderrc`，記得更新 `pbuilder` 環境

    ```sh
    sudo OS=debian DIST=sid ARCH=i386 pbuilder update --override-config --configfile ~/.pbuilderrc

    sudo OS=debian DIST=sid ARCH=amd64 pbuilder update --override-config --configfile ~/.pbuilderrc
    ```

1. 執行 `pbuilder` 來檢查套件

    ```sh
    # 注意若changelog版本號有變，xxx.dsc的檔案名也要更動
    sudo OS=debian DIST=sid ARCH=i386 pbuilder build [剛剛產生的的dsc檔].dsc
    sudo OS=debian DIST=sid ARCH=amd64 pbuilder build [剛剛產生的的dsc檔].dsc

    # 若有錯誤，修正後先跑debuild然後再重複上面pbuilder build的步驟
    debuild
    ```

## 上傳套件

一切檢查完成後，我們可以透過下面步驟上傳套件到 Debian 系統上。

1. commit 並推上自己在 Salsa 的 repository 上

1. 請認識的 Debian Developer 來 clone repository，或是發 MR 到已經存在的 repository，再請他們幫忙重新打包上傳

1. 當打包過程中，發現原始套件有誤，有作出修正時，記得將 patch 送回上游進行反饋

## 參考資源

- [Debian 新維護人員手冊](https://www.debian.org/doc/manuals/maint-guide/index.zh-tw.html): 本文只能簡單帶過，如果遇到其他問題還是建議參考這份文件。

- [配合Git为Debian系发行版打包的正确方式](https://hosiet.me/blog/2016/09/15/make-debian-package-with-git-the-canonical-way/): 非常推薦看這份文件，寫得很詳細。

## 感謝

本文由 `Chen-ying Kuo` 與 `Kun-Hung Tsai` 共同編寫。感謝獲得 `SZ lin` 的許多回饋與教導，與 `Kaiden Yu` 校正其中錯誤內容
