# Sync Gitee
一个用于在hub间（例如Github，Gitee）账户代码仓库同步的action

## 用法
```yaml
steps:
- name: Mirror the Github organization repos to Gitee.
  uses: Yikun/hub-mirror-action@master
  with:
    src: github/kunpengcompute
    dst: gitee/kunpengcompute
    dst_key: ${{ secrets.GITEE_PRIVATE_KEY }}
    dst_token: ${{ secrets.GITEE_TOKEN }}
    account_type: org
```
上面的配置完成了kunpencompute组织从github到gitee的同步，你可以在[测试和demo](https://github.com/Yikun/hub-mirror-action/tree/master/.github/workflows)找到完整用法。

有疑问、想法、问题、建议，可以通过[![Gitter](https://badges.gitter.im/hub-mirror-action/community.svg)](https://gitter.im/hub-mirror-action/community?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge)找到我们。

## 参数详解
#### 必选参数
- `src` 需要被同步的源端账户名，如github/kunpengcompute，表示Github的kunpengcompute账户。
- `dst` 需要同步到的目的端账户名，如gitee/kunpengcompute，表示Gitee的kunpengcompute账户。
- `dst_key` 用于在目的端上传代码的私钥(默认可以从~/.ssh/id_rsa获取），可参考[生成/添加SSH公钥](https://gitee.com/help/articles/4181)或[generating SSH keys](https://docs.github.com/articles/generating-an-ssh-key/)生成，并确认对应公钥已经被正确配置在目的端。对应公钥，Github可以在[这里](https://github.com/settings/keys)配置，Gitee可以[这里](https://gitee.com/profile/sshkeys)配置。
- `dst_token` 创建仓库的API tokens， 用于自动创建不存在的仓库，Github可以在[这里](https://github.com/settings/tokens)找到，Gitee可以在[这里](https://gitee.com/profile/personal_access_tokens)找到。

#### 可选参数
- `account_type` 默认为user，源和目的的账户类型，可以设置为org（组织）或者user（用户），目前仅支持**同类型账户**（即组织到组织，或用户到用户）的同步。
- `clone_style` 默认为https，可以设置为ssh或者https。
- `cache_path` 默认为'', 将代码缓存在指定目录，用于与actions/cache配合以加速镜像过程。
- `black_list` 默认为'', 配置后，黑名单中的repos将不会被同步，如“repo1,repo2,repo3”。
- `white_list` 默认为'', 配置后，仅同步白名单中的repos，如“repo1,repo2,repo3”。
- `static_list` 默认为'', 配置后，仅同步静态列表，不会再动态获取需同步列表（黑白名单机制依旧生效），如“repo1,repo2,repo3”。
- `force_update` 默认为false, 配置后，启用git push -f强制同步，**注意：开启后，会强制覆盖目的端仓库**。
- `debug` 默认为false, 配置后，启用debug开关，会显示所有执行命令。
- `timeout` 默认为'30m', 用于设置每个git命令的超时时间，'600'=>600s, '30m'=>30 mins, '1h'=>1 hours
- `mappings` 源仓库映射规则，比如'A=>B, C=>CC', A会被映射为B，C会映射为CC，映射不具有传递性。主要用于源和目的仓库名不同的镜像。

## 举些例子

#### 组织同步

同步Github的kunpengcompute组织到Gitee

```yaml
- name: Organization mirror
  uses: Yikun/hub-mirror-action@master
  with:
    src: github/kunpengcompute
    dst: gitee/kunpengcompute
    dst_key: ${{ secrets.GITEE_PRIVATE_KEY }}
    dst_token: ${{ secrets.GITEE_TOKEN }}
    account_type: org
```

#### 黑/白名单

动态获取源端github/Yikun的repos，但仅同步名为hub-mirror-action，不同步hashes这个repo到Gittee

```yaml
- name: Single repo mirror
  uses: Yikun/hub-mirror-action@master
  with:
    src: github/Yikun
    dst: gitee/yikunkero
    dst_key: ${{ secrets.GITEE_PRIVATE_KEY }}
    dst_token: ${{ secrets.GITEE_TOKEN }}
    white_list: "hub-mirror-action"
    black_list: "hashes"
```

#### 静态名单（可用于单一仓库同步）

不会动态获取源端github/Yikun的repos，仅同步hub-mirror-action这个repo

```yaml
- name: Black list
  uses: Yikun/hub-mirror-action@master
  with:
    src: github/Yikun
    dst: gitee/yikunkero
    dst_key: ${{ secrets.GITEE_PRIVATE_KEY }}
    dst_token: ${{ secrets.GITEE_TOKEN }}
    static_list: "hub-mirror-action"
```

#### clone方式

使用ssh方式进行clone

```yaml
- name: ssh clone style
  uses: Yikun/hub-mirror-action@master
  with:
    src: github/Yikun
    dst: gitee/yikunkero
    dst_key: ${{ secrets.GITEE_PRIVATE_KEY }}
    dst_token: ${{ secrets.GITEE_TOKEN }}
    clone_style: "ssh"
```

#### 指定目录cache
```yaml
- name: Mirror with specific cache
  uses: Yikun/hub-mirror-action@master
  with:
    src: github/Yikun
    dst: gitee/yikunkero
    dst_key: ${{ secrets.GITEE_PRIVATE_KEY }}
    dst_token: ${{ secrets.GITEE_TOKEN }}
    cache_path: /github/workspace/hub-mirror-cache
```

#### 强制更新，并打开debug日志开关
```yaml
- name: Mirror with force push (git push -f)
  uses: Yikun/hub-mirror-action@master
  with:
    src: github/Yikun
    dst: gitee/yikunkero
    dst_key: ${{ secrets.GITEE_PRIVATE_KEY }}
    dst_token: ${{ secrets.GITEE_TOKEN }}
    force_update: true
    debug: true
```

#### 设置命令行超时时间为1小时
```yaml
- name: Mirror with force push (git push -f)
  uses: Yikun/hub-mirror-action@master
  with:
    src: github/Yikun
    dst: gitee/yikunkero
    dst_key: ${{ secrets.GITEE_PRIVATE_KEY }}
    dst_token: ${{ secrets.GITEE_TOKEN }}
    force_update: true
    timeout: '1h'
```

#### 仓库名不同时同步（github/yikun/yikun.github.com to gitee/yikunkero/blog）
```
- name: mirror with mappings
  uses: Yikun/hub-mirror-action@mappings
  with:
    src: github/yikun
    dst: gitee/yikunkero
    dst_key: ${{ secrets.GITEE_PRIVATE_KEY }}
    dst_token: ${{ secrets.GITEE_TOKEN }}
    mappings: "yikun.github.com=>blog"
    static_list: "yikun.github.com"
```

## FAQ

- 如何在secrets添加dst_token和dst_key？
  下面是添加secrets的方法，也可以参考[secrets官方文档](https://help.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets)了解更多：
  1. **获取Token和Key**，分别获取[ssh key](https://gitee.com/profile/sshkeys)和[token](https://gitee.com/profile/personal_access_tokens)。
  2. **增加Secrets配置**，在配置仓库的Setting-Secrets中新增Secrets，例如GITEE_PRIVATE_KEY、GITEE_TOKEN
  3. **在Workflow中引用**， 可以用过类似`${{ secrets.GITEE_PRIVATE_KEY }}`来访问

## 参考
- [Hub mirror template](https://github.com/yi-Xu-0100/hub-mirror): 一个用于展示如何使用这个action的模板仓库. from @yi-Xu-0100
- [自动镜像 GitHub 仓库到 Gitee](https://github.com/ShixiangWang/sync2gitee): 一个关于如何使用这个action的介绍. from @ShixiangWang
- [巧用Github Action同步代码到Gitee](http://yikun.github.io/2020/01/17/%E5%B7%A7%E7%94%A8Github-Action%E5%90%8C%E6%AD%A5%E4%BB%A3%E7%A0%81%E5%88%B0Gitee/): Github Action第一篇软文