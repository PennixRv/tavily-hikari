# History

## 2026-11-09：submodule 工作树定位修复

根协调任务 `paseo-based-parallelism-integration-design` 的 W2 发现：以 Git common directory
的父目录推算 primary worktree，会在 submodule 中误入 superproject 的 Git 元数据目录。
现读取 `git worktree list --porcelain -z` 的首条工作树记录；对 absorbed submodule，
再按 common directory 的原生 `core.worktree` 解析实际 checkout，并核对 common directory 身份。
primary checkout 不可用时，自动准备只告警，显式 strict 准备失败并提示 `git worktree repair`。
隔离 smoke 增加真实 submodule、含空格路径、独立 env 来源、目标不覆盖及失效来源两种失败语义。
本轮不读取实际用户 env，不启动服务，也不改变原有依赖准备范围。

## Decision Trace

- 2026-07-15: 决定把 linked worktree bootstrap 正式收口为 shared `post-checkout` + explicit `worktree:setup` 双入口，而不是继续散落在启动脚本里做隐式补救。
- 2026-07-15: 决定 env 只从 primary worktree `copy-missing`，不引入 `.env.example` fallback，也不允许覆盖目标已有文件。
- 2026-07-15: 决定自动路径固定为 `best-effort + warning`，显式 repair 才承担 `force + strict` 语义。
- 2026-07-15: 决定恢复面只覆盖 hooks、root / `web` / `docs-site` Bun 依赖与 `cargo fetch --locked`，明确排除 DB / dist / runtime artifacts。
- 2026-07-15: 决定用真实 linked worktree smoke + CI lightweight job 锁住合同，并要求历史 revision 缺脚本时安全 no-op。

## Key Reasons

- 这个仓原先只有“启动某个子系统时顺手补依赖”的局部策略，没有 worktree 级 current truth，因此新 linked checkout 很容易出现 hooks 缺失、root 工具链缺失、docs-site 缺失、env 断层。
- primary worktree 是当前仓唯一真实且用户已确认过的本地配置来源；`copy-missing` 能保留 linked worktree 自己的私有修改，避免 secret 被意外覆盖。
- shared common hooks 比“每个 linked worktree 再手工装一次 hooks”更符合 git worktree 的共享事实，也让未来 linked checkout 能自动进入 bootstrap 路径。
- 明确排除 DB / dist / runtime artifacts 可以避免 bootstrap 退化成不透明的全运行态修复器，减少误恢复与大体积副作用。

## References

- `./SPEC.md`
- `./IMPLEMENTATION.md`

## 变更记录（Change log）

- 2026-07-15: 创建 worktree bootstrap topic spec，冻结 shared `post-checkout`、primary `copy-missing` env、auto `best-effort`、manual strict repair，以及 DB/dist/runtime exclusion contract。
- 2026-07-15: 实现 `scripts/install-hooks.sh`、`scripts/worktree-bootstrap.sh`、`scripts/worktree-setup.sh`、`scripts/test-worktree-bootstrap.sh`、root package scripts、CI smoke、README/AGENTS current truth。

## Legacy Identity

- Legacy compatibility identity: `#w7tbk`.
