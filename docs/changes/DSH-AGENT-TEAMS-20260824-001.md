# AgentTeams Manager 参数页与 Settings 接入

## 问题

AgentTeams 原先只有 Cordis entry 配置，没有供 Manager 使用的参数页。如果只增加声明文件，Manager 保存的值不会进入插件真实运行路径。

## 修改

- 注册官方 DSH Settings namespace：
  - `agent-teams`（live）：`memberProvider`、`memberDefaultRoute`、`memberMaxDepth`、`maxMembers`；
  - `agent-teams-startup`（restart）：`stateDir`、`slashCommand`、`promptSectionOrder`；
- 新增 `dsh-management/panel.yaml`，使用 Contract v2 与 DSH 模型目录选择器；
- `promptSectionOrder` 放入默认折叠的“开发者选项”；
- 工具配置改成动态读取 live Settings，后续新建成员与容量检查立即使用新值；
- 启动配置在进程启动时取快照，保存后必须重启才改变命令、状态目录和系统提示段顺序；
- 保留旧 `memberModel` 兼容层。

## 模型选择优先级

1. `agent_teams_add_member` 显式 `provider/model/reasoning_effort`；
2. Manager/Settings 的 `memberDefaultRoute`；
3. 旧 Cordis `memberModel`；
4. 队长当前会话模型与思考强度。

未保存过 `memberDefaultRoute` 时仍允许旧 `memberModel` 生效；用户在面板显式选择“跟随队长”后，保存的 `null` 会覆盖旧值。

## 关键位置

- `src/index.ts`：两个 Settings namespace、live getter 与 restart 快照；
- `src/members.ts`：默认模型优先级和最终 `ctx.llm.resolveCallConfig()` 校验；
- `src/tools.ts`：真实工具消费者；
- `dsh-management/panel.yaml`：Manager 参数页声明；
- `scripts/verify.mjs`：显式参数、固定默认、旧值和跟随队长的优先级验证。

## 发布排错

第一次通过 Maintenance 从干净 Git checkout 打包时，registry 未登记构建命令；由于 `lib/` 是生成物且不提交 Git，得到的 tgz 缺少 `lib/index.js`，冷启动报 `ERR_MODULE_NOT_FOUND`。修复方式是在 Maintenance registry 为本插件固定 `pnpm build && pnpm verify` 配方，再从同一提交重新构建内容寻址 artifact。以后不能把开发工作树中残留的 `lib/` 当成发布成功证据。

## 回退

回退本提交并重新安装 0.1.13。DSH profile 中新增的两个 Settings namespace 可以保留；旧版本不会消费它们，也不会影响原 `memberModel` 配置。
