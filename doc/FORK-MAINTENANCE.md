# 我的 Paperclip fork 维护手册

本手册记录 2026-10-12 对 `LiuGuoJiang/paperclip` 的实际整理，并给出后续维护步骤。命令默认在本仓库根目录执行。

## 分支和远程仓库

| 名称 | 地址或用途 |
| --- | --- |
| `upstream` | 官方仓库 `https://github.com/paperclipai/paperclip.git` |
| `origin` | 我的 fork `https://github.com/LiuGuoJiang/paperclip.git` |
| `master` | 只同步官方代码，尽量保持与 `upstream/master` 一致 |
| `custom` | 我的长期维护分支；自己的改动和这份手册放在这里 |
| 功能分支 | 从 `custom` 建立，用于单项修改和自仓库 PR |

## 这次实际做了什么

1. 检查工作区：没有未提交修改；本地在 `master`，原 `origin` 指向官方仓库。
2. 核对远端：fork 的 `master` 在 `1944c861532dcfb2cf953619523d3639b4a7b079`；官方的 `master` 在 `ce08872dfb5490bd69b83a5db697c373a4f77b18`。fork 没有独立提交，只比官方落后 1875 个提交，因此可安全快进。
3. 把原 `origin` 改名为 `upstream`，将我的 fork 添加为新 `origin`；分别 `fetch` 两个仓库。
4. 使用 `git merge --ff-only upstream/master` 将本地 `master` 更新到 `ce08872df`，并设置本地 `master` 跟踪 `origin/master`。
5. 终端执行 `git push origin master` 时，GitHub HTTPS 凭据不可用，报错 `could not read Username`。使用已连接的 GitHub 账户，按原分支 SHA 校验后对 fork 的 `master` 执行无强制快进更新。随后重新获取并确认 fork 的 `master` 也是 `ce08872df`。
6. 从该提交建立 `custom` 分支，并让本地 `custom` 跟踪 `origin/custom`。此手册存放在 `custom` 中。

没有重置、强制推送或改写原有提交。

## 日常获取官方更新

先确认工作区干净：

```sh
git status --short --branch
```

若有未提交改动，先完成提交，或临时保存：`git stash push -u -m "同步前暂存"`。同步后可用 `git stash pop` 恢复，并处理可能出现的冲突。

```sh
git switch master
git fetch upstream
git merge --ff-only upstream/master
git push origin master
```

`--ff-only` 只允许沿官方历史向前移动。如果失败，先用 `git log --oneline --left-right master...upstream/master` 查明分叉原因，不要强制推送。若终端提示没有 GitHub 凭据，先配置 GitHub CLI／Git Credential Manager 或 SSH，再重试推送；也可以在 GitHub fork 页面使用 **Sync fork**，随后执行 `git fetch origin` 校验。首次本机配置好认证后，上述 `git push` 就可直接使用。

再将官方更新合入自己的维护分支：

```sh
git switch custom
git merge master
pnpm -r typecheck
pnpm test:run
pnpm build
git push origin custom
```

当 `custom` 没有独立提交时，合并是快进；有独立提交后，Git 通常会生成合并提交。若发生冲突，打开冲突文件，保留正确的代码，执行 `git add <文件>` 和 `git commit` 完成合并；想撤销这次合并可执行 `git merge --abort`。按改动范围补充必要的专项测试。仓库要求 PR 就绪时通过完整的 typecheck、测试和构建。

如果暂时不想把官方更新并入 `custom`，可以只同步 `master`，等准备验证自己的修改时再合并。

## 做自己的修改

从 `custom` 建立单项功能分支，便于评审与回退：

```sh
git switch custom
git pull --ff-only origin custom
git switch -c codex/my-feature
# 编辑文件，并运行与改动相关的检查
git add <修改的文件>
git commit -m "简述修改"
git push -u origin codex/my-feature
```

在 GitHub 创建 PR 时，目标仓库选 `LiuGuoJiang/paperclip`，目标分支选 `custom`。填写仓库的 `.github/PULL_REQUEST_TEMPLATE.md` 全部章节，并按 `AGENTS.md` 完成检查。合并后更新本地分支：

```sh
git switch custom
git pull --ff-only origin custom
```

只有希望向官方贡献通用改动时，才另开面向 `paperclipai/paperclip:master` 的 PR。

## 每次同步后的核对

```sh
git remote -v
git branch -vv
git status --short --branch
git rev-list --left-right --count origin/master...upstream/master
```

最后一条命令输出 `0  0` 表示两个远端跟踪分支在本地已一致。它们是 `fetch` 时记录的快照；要确认 GitHub 上的实时状态，先再次 `git fetch --all --prune`。如果想看 `custom` 比官方多出哪些提交，运行 `git log --oneline master..custom`。

