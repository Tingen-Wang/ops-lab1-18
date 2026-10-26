# 实验 1 记录：第 18 组

现场检查时将打开本文件，按其中的链接核对。证据一律以链接或文本形式给出，不要只贴截图。将 `<填写>` 替换为本组的内容。main 受保护之后，修改本文件也须通过 PR。

## 1. 基本信息

| 项 | 内容 |
|---|---|
| 小组 | 第 18 组 |
| 仓库地址 | https://github.com/Tingen-Wang/ops-lab1-18 |
| 代码载体 | 统一示例（`enroll-conflict`，Go） |
| runner 与语言版本 | `ubuntu-24.04`，Go `1.26.9`（工作流固定到补丁号；本地同为 1.26.9） |
| 合并方式 | squash（PR 标题即 main 上的提交标题） |

## 2. 任务 0：本地基线

| 问题 | 观察结果 |
|---|---|
| 第一次 `make package` 得到的 BUILD_INFO（粘贴） | name=enroll-conflict<br>version=1.0.0<br>build_no=1<br>run_id=local<br>commit=7b63b41c13f061439997b0e6e4eaa2af922a7aa1<br>go=go1.26.9<br><br>`sha256sum`：<br>`f23447d406c42af7dd35e23c2d1b2f15484cfb709cc9b4bb1262643694b8528e  dist/enroll-conflict_1.0.0_linux_amd64/enroll-conflict`<br><br>`go version -m` 的 vcs 行：<br>`build   vcs=git`<br>`build   vcs.revision=7b63b41c13f061439997b0e6e4eaa2af922a7aa1`<br>`build   vcs.time=2026-10-12T16:26:07Z`<br>`build   vcs.modified=false` |
| 修改一行、不提交，再次打包：BUILD_INFO 是否变化？二进制文件的 sha256 是否变化？（贴出两次的 sha256） | **BUILD_INFO 没有变化**（`diff` 无输出，因为里面没有构建时间，`commit` 取的是 `git rev-parse HEAD`，改文件不改 HEAD）。<br>**二进制 sha256 变了**：<br>第一次 `f23447d406c42af7dd35e23c2d1b2f15484cfb709cc9b4bb1262643694b8528e`<br>第二次 `10b61e37d95efe4ecf9ebb8c2e3dd921048b7009724182ae5e1b2dbf29f0fac2`<br>两次的 vcs 行：<br>`build   vcs=git`<br>`build   vcs.revision=7b63b41c13f061439997b0e6e4eaa2af922a7aa1`<br>`build   vcs.time=2026-10-12T16:26:07Z`<br>`build   vcs.modified=true` |
| 两次 `go version -m` 输出中的 vcs 行有何不同？ | `vcs`、`vcs.revision`、`vcs.time` 三行完全相同，只有 `vcs.modified` 由 `false` 变成 `true`——Go 把"构建时工作区是脏的"这个事实编进了二进制，因此同一份 `BUILD_INFO` 对应了两个字节不同的二进制。 |
| 仅凭 BUILD_INFO，能否回答该包是什么、在哪台机器上、何时、用哪份代码构建？哪一项信息会出错？ | 只能回答"是什么"（`name`/`version`）。**答不出"在哪台机器上"和"何时"**：`BUILD_INFO` 里没有机器名、也没有构建时间（本组 Makefile 有意不写时间，见问题 3）。**"用哪份代码构建"这一项会出错**：`commit` 只记录 HEAD，与工作区是否干净无关，所以当构建时存在未提交改动（上面第二次就是 `vcs.modified=true`），`BUILD_INFO` 里仍然写着 `commit=7b63b41…`，但实际参与编译的源码已经不是那个提交了。换言之，同一组 `version`+`build_no` 可以对应两份不同的源码。 |

## 3. 证据

| 项 | 链接或文本 |
|---|---|
| 规则集里启用的规则 | Ruleset `main-protection`，Enforcement **Active**，目标分支为默认分支 `main`，**Bypass list 为空**（管理员同样被拦）。启用：**Restrict deletions**、**Block force pushes**、**Require a pull request before merging**（**Required approvals = 1**）、**Require status checks to pass**（必需检查名 = job 名 `ci`） |
| 组长直接 `git push origin main` 被拒的终端输出 | root@devops-1:~/lab1# git push origin main<br>Enumerating objects: 1, done.<br>Counting objects: 100% (1/1), done.<br>Writing objects: 100% (1/1), 188 bytes \| 188.00 KiB/s, done.<br>Total 1 (delta 0), reused 0 (delta 0), pack-reused 0<br>remote: error: GH013: Repository rule violations found for refs/heads/main.<br>remote: Review all repository rules at https://github.com/Tingen-Wang/ops-lab1-18/rules?ref=refs%2Fheads%2Fmain<br>remote: <br>remote: - Changes must be made through a pull request.<br>remote: <br>remote: - Required status check "ci" is expected.<br>remote: <br>To github.com:Tingen-Wang/ops-lab1-18.git<br> ! [remote rejected] main -> main (push declined due to repository rule violations)<br>error: failed to push some refs to 'github.com:Tingen-Wang/ops-lab1-18.git' |
| 第一轮：失败的 PR 运行 | https://github.com/Tingen-Wang/ops-lab1-18/actions/runs/40217342030 （`ci` #2，01:37:25 结束，结论 failure；`test` 未过，合并框同时列出"检查未通过"与"未审批"两项） |
| 第一轮：修复后通过的 PR 运行 | https://github.com/Tingen-Wang/ops-lab1-18/actions/runs/40217687100 （`ci` #3，01:41:54 结束，结论 success） |
| 第一轮：合入 main 后的运行（含制品） | https://github.com/Tingen-Wang/ops-lab1-18/actions/runs/40229182684 （`ci` #10，push，04:06:18–04:06:28，本次运行执行 `package` 并上传 artifact） |
| 第二轮：找到的缺陷，以及现有测试为何无法发现它 | 缺陷：在 `weeksOverlap` 里、`AllWeeks` 判断之后插入 `if a == EvenWeeks \|\| b == EvenWeeks { return false }`，使**两个都在双周的班**（同星期、节次重叠）被判为不冲突，违背注释"只有单周对单周、双周对双周才有交集"。现有 8 个用例只出现过 `odd/even`、`odd/odd`、`even/all` 与 `all/all`，**从未出现 `even/even` 组合**，`main_test.go` 的 HTTP 用例也只有 `odd/even` 与 `all/all`，所以 `make test` 全绿；而且这个错误是**对称**的，用例里逐条校验的"冲突关系应对称"断言同样发现不了它。 |
| 第二轮：仅含缺陷时仍然通过的运行 | https://github.com/Tingen-Wang/ops-lab1-18/actions/runs/40219325749 （`ci` #4，02:02:30 结束，success——自动检查放行了语义缺陷，合并框只剩"待审批"） |
| 第二轮：补测试后失败的运行 | https://github.com/Tingen-Wang/ops-lab1-18/actions/runs/40219579933 （`ci` #5，02:05:43 结束，failure；新增用例 `{"双周对双周冲突", mon(1,2,EvenWeeks), mon(1,2,EvenWeeks), true}` 抓到该缺陷） |
| 第二轮：撤销缺陷后通过的运行 | https://github.com/Tingen-Wang/ops-lab1-18/actions/runs/40219821389 （`ci` #6，02:08:47 结束，success；`git revert` 掉缺陷提交后 squash 合入，main 上只多出这个新用例） |
| 未合入分支上的 tag 被 release gate 拒绝的运行 | https://github.com/Tingen-Wang/ops-lab1-18/actions/runs/40231237859 （`release` #2，tag `v0.0.1` 打在 `scratch/gate-test` 上，04:32 在 `release gate` 一步失败，未产生 Release；测试用的 tag 与分支已删除） |
| Release v1.0.0 | https://github.com/Tingen-Wang/ops-lab1-18/releases/tag/v1.0.0 （04:13 发布，附件 `enroll-conflict_1.0.0_linux_amd64.tar.gz` + `.sha256`） |
| Release v1.0.1 | https://github.com/Tingen-Wang/ops-lab1-18/releases/tag/v1.0.1 （15:53 发布，附件 `enroll-conflict_1.0.1_linux_amd64.tar.gz` + `.sha256`） |

任务 5 记录（v1.0.0 发布当时，从 Releases 页面下载的两个附件）：

```text
enroll-conflict_1.0.0_linux_amd64.tar.gz: OK
73b126d9cd82f960376dbf008a80a3eb8da1075ff53ed21702c66462de16f427  enroll-conflict_1.0.0_linux_amd64.tar.gz
```

任务 6 记录（发布 v1.0.1 之后重新下载两个版本的附件）：

```text
73b126d9cd82f960376dbf008a80a3eb8da1075ff53ed21702c66462de16f427  dist/release/enroll-conflict_1.0.0_linux_amd64.tar.gz
de55cae42619ed58ef213d33fa539fd6fd73af702b9ddde64a19bb6002c05bdd  dist/release/enroll-conflict_1.0.1_linux_amd64.tar.gz
```

## 4. 追溯与运行

对 v1.0.0 和 v1.0.1 各运行一次 `scripts/trace.sh`，粘贴输出，并填写下表。

| 项 | v1.0.0 | v1.0.1 |
|---|---|---|
| 版本号 | 1.0.0 | 1.0.1 |
| 构建号 | 1 | 3 |
| 提交号（SHA） | 68b25a200e7843bd0422e302823ca4900ccdb9f3 | d8fb207235d87cd6e5fb207423c3d1b909cd9a0d |
| 引入该提交的 PR | #3 https://github.com/Tingen-Wang/ops-lab1-18/pull/3 | #4 https://github.com/Tingen-Wang/ops-lab1-18/pull/4 |
| 构建它的 Actions 运行（`trace.sh` 给出的链接） | https://github.com/Tingen-Wang/ops-lab1-18/actions/runs/40229753471 | https://github.com/Tingen-Wang/ops-lab1-18/actions/runs/40288579260 |
| 制品 sha256（发布当时，取自 Release 里的 .sha256） | 73b126d9cd82f960376dbf008a80a3eb8da1075ff53ed21702c66462de16f427 | de55cae42619ed58ef213d33fa539fd6fd73af702b9ddde64a19bb6002c05bdd |

**仅凭构建号能否唯一确定是哪次运行？** 不能。`build_no` 取自 `github.run_number`，而它在**每个工作流里各自从 1 开始计数**：本组 `ci` 的第 1 次运行是 40214456907（01:01 push `ci.yml` 到 main），`release` 的第 1 次运行是 40229753471（04:13 发布 v1.0.0），两者**都是构建号 1**。只看 v1.0.0 的 `build_no=1` 无法判断它来自哪个工作流的哪一次运行；能唯一定位的是 `BUILD_INFO` 里的 `run_id`（`github.run_id`，全 GitHub 唯一），`trace.sh` 正是用它直接拼出运行链接。v1.0.1 同理：`build_no=3` 对应 `release` 的第 3 次运行（第 1 次 v1.0.0、第 2 次是被 gate 拒绝的 `v0.0.1`）。

**发布 v1.0.1 之后重新下载 v1.0.0 的附件，计算得到的 sha256**：`73b126d9cd82f960376dbf008a80a3eb8da1075ff53ed21702c66462de16f427`。与上表中发布当时的值**完全一致**，说明制品没有被替换，Release 附件是不可变的。

`trace.sh` 输出（v1.0.0）：

```text
== 1. 校验和
enroll-conflict_1.0.0_linux_amd64.tar.gz: OK
== 2. 制品里的 BUILD_INFO
name=enroll-conflict
version=1.0.0
build_no=1
run_id=40229753471
commit=68b25a200e7843bd0422e302823ca4900ccdb9f3
go=go1.26.9
== 3. 提交 68b25a200e7843bd0422e302823ca4900ccdb9f3
作者: Tingen-Wang <2174024810@qq.com>
时间: 2026-10-26T04:07:12+08:00
标题: ci: 添加 tag 触发的发布工作流（含 release gate） (#3)
主干: 该提交在 origin/main 的历史上
指向它的 tag: v1.0.0
版本号: 1.0.0    构建号: 1
提交页面（含关联的 PR）: https://github.com/Tingen-Wang/ops-lab1-18/commit/68b25a200e7843bd0422e302823ca4900ccdb9f3
构建它的运行: https://github.com/Tingen-Wang/ops-lab1-18/actions/runs/40229753471
```

`trace.sh` 输出（v1.0.1）：

```text
== 1. 校验和
enroll-conflict_1.0.1_linux_amd64.tar.gz: OK
== 2. 制品里的 BUILD_INFO
name=enroll-conflict
version=1.0.1
build_no=3
run_id=40288579260
commit=d8fb207235d87cd6e5fb207423c3d1b909cd9a0d
go=go1.26.9
== 3. 提交 d8fb207235d87cd6e5fb207423c3d1b909cd9a0d
作者: Tingen-Wang <2174024810@qq.com>
时间: 2026-10-26T09:32:33+08:00
标题: 修复： /api/conflict 的 400 响应指出哪一方时段不合法 (#4)
主干: 该提交在 origin/main 的历史上
指向它的 tag: v1.0.1
版本号: 1.0.1    构建号: 3
提交页面（含关联的 PR）: https://github.com/Tingen-Wang/ops-lab1-18/commit/d8fb207235d87cd6e5fb207423c3d1b909cd9a0d
构建它的运行: https://github.com/Tingen-Wang/ops-lab1-18/actions/runs/40288579260
```

任务 6 第 4 步，两次 `/version` 的输出：

```text
<运行 v1.0.1 时>
[1] 241190
2026/10/26 16:18:40 enroll-conflict 1.0.1 (d8fb207235d87cd6e5fb207423c3d1b909cd9a0d) 监听 127.0.0.1:18080
{"version":"1.0.1","commit":"d8fb207235d87cd6e5fb207423c3d1b909cd9a0d","build_no":"3","go":"go1.26.9"}
<换回 v1.0.0 后>
[1] 241442
2026/10/26 16:19:41 enroll-conflict 1.0.0 (68b25a200e7843bd0422e302823ca4900ccdb9f3) 监听 127.0.0.1:18080
{"version":"1.0.0","commit":"68b25a200e7843bd0422e302823ca4900ccdb9f3","build_no":"1","go":"go1.26.9"}
```

## 5. 数据

时间均从 GitHub 页面读取，**时区为 UTC+8**（2026-10-26）。时长取自各次运行的 job 起止时间，与 Actions 页面的 Duration 一致（个别差 1 秒）。

第一轮 gate 演练的时间线：

| 事件 | 时间 | 链接 |
|---|---|---|
| 推送分支并发起 PR | 01:37 | https://github.com/Tingen-Wang/ops-lab1-18/pull/1 |
| 第一次 `ci` 失败（运行结束） | 01:37:25 | https://github.com/Tingen-Wang/ops-lab1-18/actions/runs/40217342030 |
| 推送修复提交（`103ac3e`） | 01:41:12 | https://github.com/Tingen-Wang/ops-lab1-18/pull/1/commits |
| `ci` 通过（运行结束） | 01:41:54 | https://github.com/Tingen-Wang/ops-lab1-18/actions/runs/40217687100 |
| 组员审批（APPROVED） | 04:05:57 | https://github.com/Tingen-Wang/ops-lab1-18/pull/1 |
| 合入 main（squash，`123ee56`） | 04:06 | https://github.com/Tingen-Wang/ops-lab1-18/pull/1 |
| main 上的 push 运行（含制品） | 04:06:18–04:06:28 | https://github.com/Tingen-Wang/ops-lab1-18/actions/runs/40229182684 |

第二轮 gate 演练的时间线（补充，供问题 1 使用）：

| 事件 | 时间 | 链接 |
|---|---|---|
| 推送"仅含缺陷"提交 | 02:01:55 | https://github.com/Tingen-Wang/ops-lab1-18/pull/2 |
| `ci` **通过**（自动检查没拦住） | 02:02:30 | https://github.com/Tingen-Wang/ops-lab1-18/actions/runs/40219325749 |
| 推送"补充测试"提交 | 约 02:05:20 | https://github.com/Tingen-Wang/ops-lab1-18/pull/2/commits |
| `ci` **失败**（新用例抓到缺陷） | 02:05:43 | https://github.com/Tingen-Wang/ops-lab1-18/actions/runs/40219579933 |
| `git revert` 缺陷提交后 `ci` 通过 | 02:08:47 | https://github.com/Tingen-Wang/ops-lab1-18/actions/runs/40219821389 |
| 组员审批 / 合入 main（`0b76faa`） | 04:06:37 / 04:06 | https://github.com/Tingen-Wang/ops-lab1-18/pull/2 |

提交到发布的间隔，至少记录三个合入的 PR：

| PR | 第一个提交的时间 | 合入时间 | 对应 Release 的发布时间 | 提交到发布的间隔 |
|---|---|---|---|---|
| #1 | 01:34:06 | 04:06 | 04:13（v1.0.0） | 2 小时 39 分 |
| #2 | 02:01:55 | 04:06 | 04:13（v1.0.0） | 2 小时 11 分 |
| #3 | 02:19:30 | 04:07 | 04:13（v1.0.0） | 1 小时 53 分 |
| #4 | 03:35:26 | 09:32 | 15:53（v1.0.1） | 12 小时 18 分 |

`ci` 工作流的运行时长（从 Actions 页面读取），全部 13 次，不取平均值：

| 运行 | 事件（PR / push） | 时长 | 备注 |
|---|---|---|---|
| #1 `40214456907` | push main | 43 s | 首次运行，`ci.yml` 直推 main；冷缓存 + 首次 `package`/上传制品 |
| #2 `40217342030` | PR #1 | 10 s | `test` 失败（第一轮缺陷被抓） |
| #3 `40217687100` | PR #1 | 15 s | 修复后通过 |
| #4 `40219325749` | PR #2 | 9 s | 仅含缺陷，**通过**（自动检查没拦住） |
| #5 `40219579933` | PR #2 | 15 s | 补充测试后失败 |
| #6 `40219821389` | PR #2 | 14 s | revert 缺陷后通过 |
| #7 `40220725679` | PR #3 | 11 s | `release.yml` 的 PR |
| #8 `40226796832` | PR #4 | 7 s | `fmt-check` 失败（`gofmt` 未通过） |
| #9 `40227426902` | PR #4 | 14 s | `gofmt` 修正后通过 |
| #10 `40229182684` | push main | 10 s | PR #1 合入（含制品） |
| #11 `40229222311` | push main | 12 s | PR #2 合入（含制品） |
| #12 `40229260131` | push main | 12 s | PR #3 合入（含制品） |
| #13 `40254726330` | push main | 14 s | PR #4 合入（含制品） |

同一时段 `release` 工作流的 3 次运行（供问题 4/5 对照，不计入上表）：`40229753471` v1.0.0 17 s 成功、`40231237859` v0.0.1 5 s 失败（release gate 拒绝）、`40288579260` v1.0.1 17 s 成功。

## 6. 分析问题

**问题 1：gate 的收益与成本。**

用到的数据：第 5 节两轮时间线、13 次 `ci` 时长（7～43 秒，中位数 12 秒）。

**拦住了什么、没拦住什么。** 第一轮 `ci` #2 用 10 秒失败在"单周/双周误判"上，第二轮补测试后 `ci` #5 用 15 秒抓到缺陷，`release` 的 gate 用 5 秒拒绝 `v0.0.1`，`gofmt` 也被 `ci` #8 用 7 秒拦下——四次拦截最贵的一次只花 15 秒。但第二轮的第一步 `ci` #4 是**绿灯**：缺陷落在 `weeksOverlap`，现有 8 个用例没有 `even/even` 组合，只要有人批准就能合入。自动门禁只能拦住"被测试表达过的规则"，拦不住"测试没想到的组合"。

**成本。** 第一轮因失败多花 4 分 39 秒（失败运行 10 秒 + 人工修改 3 分 47 秒 + 重跑 42 秒），其中等 `ci` 占 19%、人工占 81%；一次 `ci` 约 12 秒，占最短的"提交到发布间隔"（1 小时 53 分）不到 0.2%，可以忽略。真正的成本是人：PR #1 绿灯后等审批 2 小时 24 分（占其提交到发布间隔的 91%），PR #4 等了 5 小时 47 分。补充测试之后，测试仍覆盖不到的还有：未被断言的组合、非功能属性（并发、性能、日志与错误文案）、环境与部署相关的错误，以及"代码与测试同源误解"的系统性偏差。

**成立条件。** 样本仅 4 个 PR、13 次 `ci`，且都在深夜、审批人长时间离线，把"等人"成本放大了两个数量级；若审批人实时在线，成本结构会反转、`ci` 时长占比上升。`ci` 时长受共享 runner 负载影响，43 秒那次是冷启动，不作典型值。

**问题 2：PR 中的构建与 main 上的构建。**

用到的数据：PR #3 的 `ci` #7（`pull_request` 事件，head_sha=`53a2115…`）与合并后 main 上的 `68b25a2…`。

**不同。** PR 触发时 `github.sha` 是 GitHub 为"把分支合进 main"生成的**合成合并提交**（`refs/pull/3/merge`，只存在于临时引用里），`pull_request.head.sha` 是分支头的真实提交（`53a2115…`）；本组用 squash，合并后 main 上是**第三个**提交 `68b25a2…`（PR #4 同理为 `d8fb207…`），分支上的提交只保留在 PR 页面。

**为什么只能从 main 或 tag 构建。** 两者都不在 main 的历史上，`trace.sh` 第 3 项检查（`commit` 必须是 `origin/main` 的祖先）必然失败；而且 squash 之后 main 又生成了新提交，从 PR 构建的制品永远答不上"线上跑的是哪次提交"。这也是 `release` 必须在 tag 上重新编译、而不复用 PR 上制品的原因。成立条件：以上结论与合并方式强相关——若改用 merge commit，`head.sha` 会成为 main 的祖先；若用 rebase，提交号会被全部重写。

**问题 3：可重复构建。**

在本地 Linux 环境（WSL2 Ubuntu 22.04，Go 1.26.9、GNU tar 1.34、gzip 1.10）克隆小组仓库（HEAD = `d8fb207…`，工作区干净），四参数固定为 `VERSION=1.0.0`、`BUILD_NO=7`、`RUN_ID=123456789`、`COMMIT=<HEAD>`，按下表构建：

| 构建 | 条件 | 二进制 sha256 | tar.gz sha256 |
|---|---|---|---|
| ① | `make package`，`VERSION=1.0.0`、`BUILD_NO=7`、`RUN_ID=123456789`、`COMMIT=<HEAD>` 均指定固定值 | `e76231971705af67a20b6f79209d9a8a14466aabc972020adff6bebb9f5f4dde` | `4c1df7327077f65604c4be015951dd7ba4371c72afc6ecb9b7ab65d6cfc73380` |
| ② | 与 ① 完全相同，再构建一次 | `e76231971705af67a20b6f79209d9a8a14466aabc972020adff6bebb9f5f4dde` | `4c1df7327077f65604c4be015951dd7ba4371c72afc6ecb9b7ab65d6cfc73380` |
| ③ | 仅将 `BUILD_NO` 换成另一个值（8），其余同 ① | `8ff7357360c468b457fb6fceeb3d67b21a264275d71076219e6647466b7201f0` | `57f2c583d78be70bb1ee4959d03ce5f1c5863fde557d45a0222b1fa4262d4f17` |
| ④ | 去掉 `--mtime=@0`，其余同 ① | `62e62956e863df8418efe5aa22266c6c2c06c5df6608a7fc7b8cb9499618f2d7` | `5357ac110b536f0816a77419a89688cbe6793ed036ca745b92655f9100b41f8d` |
| ⑤ | 同 ④，间隔几秒后再构建一次 | `62e62956e863df8418efe5aa22266c6c2c06c5df6608a7fc7b8cb9499618f2d7` | `18a3862008b2aefc95f8c70e17abf349b0c8a50cc884347087b91d36d9ad5a60` |
| Actions | 对 `ci` #13（`40254726330`）执行 Re-run all jobs，比较两次尝试中 `package` 步骤打印的四个值 | 两次相同：四个值 `0.0.0-main`、`13`、`40254726330`、`d8fb207…` 不变，二进制字节也不变（`run_attempt` 不进制品；本次未在远端点 Re-run，见下） | 两次相同：`BUILD_INFO` 与 tar 归档内容都不变（同上） |

> 上表为本组实测：④⑤ 临时去掉 `--mtime=@0` 改的是 `Makefile`，实测后 `git checkout -- Makefile` 还原、`git status` 为空。本地哈希与 Release 附件、runner 产物不必相同（本地 tag、tar/gzip 版本、文件系统权限位等环境因素不同），表内只做互相对比。

**哪些相同、哪些不同。** ①=② 逐字节相同：`-trimpath` 去掉绝对路径、`BUILD_INFO` 不写构建时间、tar 的 `--sort/--owner/--mtime=@0` 与 `gzip -n` 固定了其余可变因素。③ 两列都变：`BUILD_NO` 同时进 `BUILD_INFO` 与 `-ldflags`。④⑤ 的二进制相同（脏状态与参数未变），但两版 tar.gz 彼此不同、也不同于 ①：去掉 `--mtime=@0` 后 tar 记录真实 mtime。

**④ 的二进制与 ① 不同。** ④ 是在"改过 Makefile 未提交"的脏工作区构建的，`go version -m` 只变两行：`mod … v1.0.1` → `v1.0.1+dirty`、`vcs.modified=false` → `true`（正是任务 0 的现象）；还原 Makefile 重建即回到 ①。tag 也会进二进制：删掉本地 tag 后用同组参数重建，`mod` 行变成伪版本 `v0.0.0-20261026013233-d8fb207235d8`，哈希由 `e7623197…` 变为 `4b0a9233…`。可见"源码相同"不足以保证字节相同，`-trimpath` 只能去掉路径。故制品应含 `version`/`build_no`/`run_id`/`commit`/`go` 这类可追溯标识，不应含构建时间、机器名、绝对路径、`run_attempt` 等无追溯价值且每次都变的内容。

**关于 `ci` #13 的 Re-run。** 它的 `package` 步骤写入 `VERSION=0.0.0-main`、`BUILD_NO=13`、`RUN_ID=40254726330`、`COMMIT=d8fb207…`；Re-run 只把 `run_attempt` 从 1 加到 2，而它既不进 `BUILD_INFO` 也不进 `-ldflags`，所以两次 attempt 的四个值与两个哈希相同。本组未对远端做写操作（没点 Re-run），因此只按机制断言"两次相同"，不虚构 runner 上的字节。

**`release` 为什么要重新编译。** 版本号必须在编译期用 `-ldflags` 注入，`ci` 在 main 上的制品是 `0.0.0-main` 加 `ci` 的构建号，改不成 `1.0.0`。这与"一次构建、多处部署"冲突：同一份源码编译两次，而 `build_no`/`run_id`/工作区状态/本地 tag 每次都不同，两次字节并不可比，真正被部署的只有 Release 附件；代价是多一次构建、`ci` 的主干制品未参与发布。

**问题 4：发布与回退。**

用到的数据：第 4 节两次 `trace.sh` 与两次 `/version` 输出、v1.0.0 在发布当时与发布 v1.0.1 之后一致（`73b126d9…`）。

**发现缺陷后**应再发 `v1.0.2`、并在 v1.0.1 的说明里写明问题，而不删 tag、不复用版本号：否则同一版本号对应两份字节，下游所有校验和、缓存与回退判断失效，已下载 v1.0.1 的人也无法再用 `trace.sh` 自证。

**回退**直接用 Release 里的 v1.0.0 附件运行，不需要重新构建：两次 `/version` 的 `commit` 分别是 `d8fb207…` 与 `68b25a2…`，与各自 `BUILD_INFO` 一致，说明跑的就是那两个不可变制品；若改为从源码重建，`run_id` 等参数无法复现，字节一般不同（参见问题 3 ③）。

**换掉附件 + 新 `.sha256`**：`trace.sh` 发现不了（它只校验包自洽与 `commit` 在 main 上，替换者原样保留 `BUILD_INFO` 即可通过）；第 4 节记录的 `73b126d9…` 能发现；开启 GitHub 的 **Immutable releases** 可从平台侧禁止替换附件与移动 tag。

## 7. 选做记录

| 选做 | 完成情况与证据 |
|---|---|
| O1 容器镜像 | 未做 |
| O2 缓存对反馈时间的影响 | 未做 |
| O3 迁移到小组项目 | 未做 |

## 8. 成员贡献

| GitHub 用户名 | 发起并合入的 PR 编号 | 审批过的 PR 编号 |
|---|---|---|
| Tingen-Wang | #1、#2、#3、#4、#5 | ——（目前尚未审批过任何 PR） |
| funnyctrl | 【待其本人 PR 合入后填写】 | #1、#2、#3、#4、#5 |
