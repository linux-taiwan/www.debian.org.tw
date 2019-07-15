---
priority: 0.6
title: 如何更新 Debian 套件版本
excerpt: How to update Debian package version
categories: topics
background-image: works-sample.png
tags:
  - featured
---

當你打包了第一個 Debian 之後，上游作者可能會持續更新版本，此時你可以按照下面步驟來更新套件的版本。

## 由上游更新套件內容

先注意自己本地端的 repository 和目前在 Salsa 上的 repository 有沒有同步，如果沒有要先做好同步。

如果我們有設定 watch 的話，可以直接透過 `gbp` 指令來更新，可參考 [New upstream release](https://wiki.debian.org/Python/GitPackaging?action=show&redirect=Python%2FGitPackagingPQ#New_upstream_release)

- 在執行前確認自己 local 端都已經有相對應的 branch，包括 `debian/master`, `pristine-tar`, `upstream`

- `gbp pq` 相關操作可參考: <http://honk.sigxcpu.org/projects/git-buildpackage/manual-html/man.gbp.pq.html>

- `gbp import-orig` 相關操作可參考: <http://honk.sigxcpu.org/projects/git-buildpackage/manual-html/man.gbp.import.orig.html>

    ```sh
    # 從 upstream 更新程式碼到 pristine-tar 和 debian branch，並且指定 local 的 debian branch 為 debian/master
    gbp import-orig --pristine-tar --uscan --debian-branch=debian/master

    # 如果跟 upstream 有衝突的話，可以用 --merge-mode=replace 強制使用 upstream commit
    gbp import-orig --pristine-tar --uscan --debian-branch=debian/master --merge-mode=replace
    ```

完成後就可以切回 debian/master branch 來測試，並作後續相關的修改了。

## 更新 Debian 相關檔案

### `debian/changelog`

更新 `debian/` 資料夾的相關內容後，我們一樣可以用 `gbp` 來更新 changelog(記得要把 UNRELEASED 狀態改為 unstable)

```sh
gbp dch --debian-branch=debian/master
```

要特別注意的是，changelog 上面記錄的是你對 debian file 的所有修改(不是 upstream 的部份)。

### `debian/control` 和 `debian/compat`

更新套件的時候要注意 `debian/control` 裡面的資訊有沒有過時，若沒有更新，在跑 `pbuilder` 應該就會跳出警告了。

- Build-Depends：這個跟 `debhelper` 版本有關，記得更新完也要更新 `debian/compat`

- Standards-Version：這個跟 `debmake` 有關

如果發現自己 local 端的 `debhelper` 和 `debmake` 過時，那可以用前面安裝 SID 版本套件的方式更新。

### 發Merge Request

當套件都更新完後，一樣要用 `debuild` 和 `pbuilder` 再跑一次，確認沒問題後，就可以把`debian/master`、`pristine-tar` 和 `upstream` 都推上 Salsa。

接下來準備送 MR 給你所認識的 Debian developer，請他幫忙打包新版，記得三個 branch 都要發 MR。

## 感謝

本文由 `Chen-ying Kuo` 與 `Kun-Hung Tsai` 共同編寫。感謝獲得 `SZ lin` 的許多回饋與教導，與 `Kaiden Yu` 校正其中錯誤內容
