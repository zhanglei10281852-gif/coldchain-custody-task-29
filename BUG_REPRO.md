# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

后台扫描日志显示已过期一条交接待办，但接口再次查询仍是 pending，下一轮扫描又重复处理同一条记录。请先不要修改代码，定位内存返回状态与数据库持久化状态为何不一致，并给出证据。 调查全程不要修改目标仓库中的生产代码、测试代码或配置。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/coldchain-custody-task-29
- 仓库地址：https://github.com/zhanglei10281852-gif/coldchain-custody-task-29.git
- parent SHA：525143b8f3d6eb27f72101c702be8bc151ea8734

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/coldchain-custody-task-29.git bug-repro
cd bug-repro
git checkout --detach 525143b8f3d6eb27f72101c702be8bc151ea8734
go test ./internal/worker -run "^TestRunOnceExpiresHandoffsAndCompletesJobs$" -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/worker -run "^TestRunOnceExpiresHandoffsAndCompletesJobs$" -count=1
--- FAIL: TestRunOnceExpiresHandoffsAndCompletesJobs (0.07s)
    worker_test.go:89: handoff = {ID:handoff_worker ShipmentID:ship_worker FromCustodian:from ToCustodian:to Location:dock Status:pending ExpiresAt:2026-08-18 07:59:00 +0000 UTC ResolvedAt:2026-08-18 08:00:00 +0000 UTC ResolutionNote:handoff expired before acceptance CreatedAt:2026-08-18 07:00:00 +0000 UTC UpdatedAt:2026-08-18 08:00:00 +0000 UTC Version:2}
FAIL
FAIL	github.com/zhanglei10281852-gif/coldchain-custody-base/internal/worker	0.069s
FAIL

```

stderr：

```text
warning: internal/worker/worker_test.go has type 100755, expected 100644
warning: internal/worker/worker_test.go has type 100755, expected 100644

```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/worker -run "^TestRunOnceExpiresHandoffsAndCompletesJobs$" -count=1
--- FAIL: TestRunOnceExpiresHandoffsAndCompletesJobs (0.33s)
    worker_test.go:89: handoff = {ID:handoff_worker ShipmentID:ship_worker FromCustodian:from ToCustodian:to Location:dock Status:pending ExpiresAt:2026-08-18 07:59:00 +0000 UTC ResolvedAt:2026-08-18 08:00:00 +0000 UTC ResolutionNote:handoff expired before acceptance CreatedAt:2026-08-18 07:00:00 +0000 UTC UpdatedAt:2026-08-18 08:00:00 +0000 UTC Version:2}
FAIL
FAIL	github.com/zhanglei10281852-gif/coldchain-custody-base/internal/worker	0.509s
FAIL

```

stderr：

```text
warning: internal/worker/worker_test.go has type 100755, expected 100644
warning: internal/worker/worker_test.go has type 100755, expected 100644

```

## 通过条件

准确定位根因，指出具体 Go 文件和符号，解释错误行为如何导致题面症状，并给出实际复现、调用链或持久化证据。 完成时目标仓库代码、测试和配置零改动。
