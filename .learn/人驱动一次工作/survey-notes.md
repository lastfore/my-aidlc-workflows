# 调查笔记：人驱动一次工作

证据索引。不是讲义。每条标【事实】【推断】或【未知】。

## 1. 规模

**中。** 一个入口，一条主路径，必读源码 11 个。没有把目录里能单独讲清的功能算进必读。测试、`dist/`、编译出的 `stage-graph.json` 不计入。

入口：Cursor 的 conductor skill `harness/cursor/skills/aidlc/SKILL.md`。人用它开始或继续一次工作；conductor 不自己排路线。

主路径数量：1。成功路径是「已有工作记录，当前站是 `intent-capture` 且尚未完成」时的一轮：`orchestrate next` 给出 `run-stage`，conductor 执行这一站，再 `orchestrate report` 把结果交回。`next` 与 `report` 是同一轮的读和写，不是两块功能。同一入口上的失败分支（解析错误、人的原话被丢掉后从消息记录补回、跳过、拒绝）不另算一条路径。

必读文件：

1. `harness/cursor/skills/aidlc/SKILL.md`
2. `harness/cursor/skills/aidlc/question-rendering.md`
3. `core/tools/aidlc.ts`
4. `core/tools/aidlc-orchestrate.ts`
5. `core/tools/aidlc-directive.ts`
6. `core/tools/aidlc-graph.ts`
7. `core/tools/aidlc-lib.ts`
8. `core/tools/aidlc-state.ts`
9. `core/aidlc-common/protocols/stage-protocol.md`
10. `core/aidlc-common/conductor.md`
11. `core/aidlc-common/stages/ideation/intent-capture.md`

## 2. 边界六问

1. 边界：人在 Cursor 里驱动一次已经在进行的工作。conductor 向引擎要下一步，执行引擎点名的那一站，再用引擎的 `report` 回报。这一轮落到的站是 `intent-capture`。打包、插件、其他 harness、其他 stage 正文不在边界内。
2. 位置：skill 是人对 conductor 说的话；`aidlc.ts` 把 `engine orchestrate` 派到 `aidlc-orchestrate.ts`；阶段协议和这一站的正文在 `core/aidlc-common`；批准落账在 `aidlc-state.ts`。
3. 要解决的问题：下一站由记录和阶段图决定，不由 conductor 重算。人在这一站里澄清意图，并在门上批准或要求修改。
4. 内部：转发循环、directive 合同、`routeNext` 的在途分支、`report` 到 state 子命令的派发、`intent-capture` 正文、Cursor 上结构化问题的渲染。外部：编译好的 `stage-graph.json`、项目里的工作记录、`Bun.spawnSync` 拉起的 state 子进程、宿主用来跑命令的 shell。
5. 依据文件：上一节的 11 个，加上优先级里点名的测试与协议。结论以这些源码为准。
6. 仍然不可得：见文末未知项。没有打开安装后的 `dist/`，没有在 Cursor 里真跑一轮，没有运行测试。

## 3. 本次主路径

起点：工作记录里已有 `Current Stage`，复选框仍在途（`pending` / `in-progress` / `awaiting-approval` / `revising`，或没有这一行）。本路径把这一站固定为 `intent-capture`。初始化三站不在这条正文里；它们用同一条循环，但 `gate` 为 false。

1. 人的入口是 Cursor skill。转发循环：先跑 `engine orchestrate next`，按 `directive.kind` 做一件事，阶段做完后跑 `engine orchestrate report`。【事实】`harness/cursor/skills/aidlc/SKILL.md:33-44`
2. `engine orchestrate` 的动词 `next` 和 `report` 派到 `aidlc-orchestrate.ts`。【事实】`core/tools/aidlc.ts:1122-1128` `engine-orchestrate`
3. `main` 在子命令 `next` 时调用 `handleNext`，`report` 时调用 `handleReport`。【事实】`core/tools/aidlc-orchestrate.ts:24009` `main`；`24209-24216`
4. `handleNext` 只调用 `routeNext`。【事实】`core/tools/aidlc-orchestrate.ts:11349-11357` `handleNext`
5. 有工作记录时，Branch 10 用 `getField(..., "Current Stage")` 和 `parseCheckboxes` 读当前位置。在途则不再往下走。【事实】`core/tools/aidlc-orchestrate.ts:14209-14221`；`14272-14282` `currentIsInFlight`；`14313`
6. 在途且不是 swarm 时，`emitForSlug`。非 per-unit 站走到 `emitRunStageForSlug`。【事实】`14436-14438`；`18781-18786` `emitForSlug`；`18835`；`15125-15141` `emitRunStageForSlug`
7. `nodeForSlug` 从编译图取节点。【事实】`11263-11265` `nodeForSlug`，内部 `loadGraph`
8. `buildRunStageDirective` 填 `kind: "run-stage"`，`stage` 来自节点 slug，`stage_file` 来自 `stageFileFor`，`gate` 来自 `computeGate`。【事实】`10140` `buildRunStageDirective`；`10196-10230`；`9077-9078` `stageFileFor`
9. `intent-capture` 的 phase 是 `ideation`，不是 initialization，也不是 construction，所以 `computeGate` 得到 `true`。【事实】`core/aidlc-common/stages/ideation/intent-capture.md:2-4`；`core/tools/aidlc-orchestrate.ts:9946-9960` `computeGate`；`9455-9458` `isSkeletonGateStage` 只认 construction
10. 工作流的第一份 `run-stage`，或这个聊天还没有拿到 persona 时，`readConductorPersona` 把 `conductor.md` 放进 `conductor_persona`。【事实】`9094-9101` `readConductorPersona`；`10389-10412`
11. conductor 对 `run-stage` 先按该步的阅读说明读文件，再按 `gate` 分支执行站正文。`gate: true` 时走提问、产物、审阅（directive 列出 `reviewer` 时）、然后 `report --result awaiting-approval`，再向人展示批准门。批准用 `report --result approved`。禁止直接调 `aidlc-state.ts` 的生命周期动词。【事实】`harness/cursor/skills/aidlc/SKILL.md:88`；`104-115`
12. 这一站的正文：Step 1 用固定命令读项目描述；Step 6 写明 `report --stage intent-capture` 拥有生命周期转移；Step 7 用协议模板展示批准门（Approve / Request Changes）。【事实】`core/aidlc-common/stages/ideation/intent-capture.md:39-45`；`229-240`
13. 给人看的结构化问题按 Cursor 附录渲染成编号散文，不把 `question` 围栏原文贴进对话。【事实】`harness/cursor/skills/aidlc/question-rendering.md:1-17`；`30-32`。skill 把批准门指到这个文件。【事实】`SKILL.md:89`；`115`
14. `handleReport` 要求 `--result`。有记录时用 `--stage`，否则用 `Current Stage`。`intent-capture` 在图里才能提交。【事实】`22016` `handleReport`；`22127-22136`；`22165-22195`
15. 非 initialization 即 `isGated`。向前的批准结果在 checkbox 仍是 `in-progress` 时先 `gate-start`，再 `approve`。子命令经 `spawnState` 拉起 `aidlc-state.ts`，不在本文件里重写转移。【事实】`22438` `isGated`；`23116-23143`；`23204-23205`；`20294-20322` `spawnState`；`19959-19986`
16. `aidlc-state.ts` 的 `approve` 进入 `handleApprove`。它不是 export。调用方还要带上 slug，并经过 `validateSlugInState(..., "awaiting-approval")`（团队门除外）。【事实】`core/tools/aidlc-state.ts:8265` `handleApprove`；`1136` 的 `"approve"` 分支；`8285`
17. `awaiting-approval` 成功后，引擎返回的 print 带上 `next_stage`（`nextInScopeStage`）和产物位置，conductor 据此展示门，而不是再发明下一站。【事实】`core/tools/aidlc-orchestrate.ts:22944-22950`；`core/tools/aidlc-lib.ts:38144` `nextInScopeStage`
18. 当前站已经完成时，Branch 10 才用 `nextInScopeStage` 找下一站；没有下一站则 `kind: "done"`。【事实】`core/tools/aidlc-orchestrate.ts:14442-14458`
19. 发出前 directive 要过 `validateDirective`。【事实】`core/tools/aidlc-orchestrate.ts:2371`；`core/tools/aidlc-directive.ts:1959` `validateDirective`；`270` `RunStageDirective`；`325` `gate`

【推断】把「一次工作」收成上面这一轮，而不是从空文件夹创建 intent 再自动跑完初始化。依据：用户要求只走一条主路径，并且只把真正执行的那一个 stage 文件算必读；初始化三站 `gate` 为 false，正文可以单独理解。创建 intent 的 `print` 在文件头被点名，但不在这 11 个文件的成功路径上展开。【事实】文件头把新鲜工作区的创建说成单独的 `intent-create` 动作：`core/tools/aidlc-orchestrate.ts:17-19`

【推断】在包含 `intent-capture` 的计划里，它是初始化之后人要批准的第一站。依据：阶段表把它排在三站初始化之后（`harness/cursor/skills/aidlc/tables.md:37-40`），协议写明除初始化三站外都要人批准（`stage-protocol.md:191`），`computeGate` 与此一致。`classic` 不在这一站的 `scopes` 里，所以不是每个计划都会走到它。【事实】`intent-capture.md:26-30`

## 4. 暂不展开

- 其余 stage 正文。初始化三站（`workspace-scaffold`、`workspace-detection`、`state-init`）走同一条循环且 `gate` 为 false，正文不读。Construction、swarm、per-unit、code-generation 不读。
- 打包 `scripts/package.ts`，以及 `{{INVOKE}}` 在安装树里被替换成什么。
- 插件、传感器实现、规则学习环（`stage-protocol-learnings.md`）。`intent-capture` 的 Sensors / Learn 节因此不展开。
- 审阅协议正文。这一站 frontmatter 有 `reviewer`，skill 在完成序列第 1 步会加载它；协议本身能单独理解。
- composer、`next compose`、改 scope、`--skip` / `--add`、park、jump、config、status、doctor、team-board、wait、`--single`。
- 其他 harness 的 aidlc skill。Cursor 差异已写在本 skill 的 Harness notes 里，不另读副本。
- 人的原话没被放进 argv 时，从消息记录补回的分支（`takePersonsOwnLine`）。那是同一入口的失败补救，不是这条成功路径。

无独立功能被漏记成「无」。上面这些都在同一批目录里，但不属于这一次的必读。

## 5. 文件优先级

| 优先级 | 文件 | 关键符号与行号 | 原因 |
|---|---|---|---|
| P0 | `harness/cursor/skills/aidlc/SKILL.md` | 转发循环 `33-44`；`run-stage` / `ask` 行 `88-89`；`gate` 分支 `104-115`；Cursor 备注 `117-122` | 人驱动时 conductor 唯一的控制结构 |
| P0 | `harness/cursor/skills/aidlc/question-rendering.md` | 渲染合同 `1-17`；批准门也算结构化问题 `30-32` | 人不看围栏，看编号选项 |
| P0 | `core/tools/aidlc.ts` | `engine-orchestrate` `1122-1128` | `next` / `report` 如何进引擎 |
| P0 | `core/tools/aidlc-orchestrate.ts` | `main` `24009`；`routeNext` `11832`；Branch 10 `14209`；`emitForSlug` `18786`；`buildRunStageDirective` `10140`；`computeGate` `9946`；`handleReport` `22016`；`spawnState` `20298` | 下一站和回报的决策 |
| P0 | `core/tools/aidlc-directive.ts` | `DirectiveKind` `104`；`RunStageDirective` `270`；`validateDirective` `1959` | 引擎只能交出合同里的一种 directive |
| P0 | `core/tools/aidlc-graph.ts` | `GraphStage` `190`；`loadGraph` `1401` | `run-stage` 的字段从编译节点来 |
| P0 | `core/tools/aidlc-lib.ts` | `loadStageGraph` `36666`；`nextInScopeStage` `38144` | 图的读取，以及批准后下一站的名字 |
| P0 | `core/tools/aidlc-state.ts` | `approve` 分发 `1136`；`handleApprove` `8265` | `report` 的 gated 提交真正落在这里 |
| P0 | `core/aidlc-common/protocols/stage-protocol.md` | 人驱动 `5`；批准门 `189-191` | 站内提问、停住、完成话术的合同 |
| P0 | `core/aidlc-common/conductor.md` | 循环与本文件的分工 `1-13`；只经 report 改生命周期 `34-35` | 第一份 `run-stage` 会把全文交给 conductor |
| P0 | `core/aidlc-common/stages/ideation/intent-capture.md` | frontmatter `1-33`；Step 6–7 `229-240` | 这条路径执行的那一个站 |
| P1 | `tests/unit/t114-orchestrate-next.test.ts` | 文件头 `1-10`；在途当前站到 `run-stage` 的 describe `172` | 改 `next` 时要回归。本次没跑 |
| P1 | `tests/unit/t115.test.ts` | 文件头 `1-14`；report 前置 `263`；approved 派到 `approve` 的注释 `771` | 改 `report` / `approve` 时要回归。本次没跑 |
| P1 | `tests/unit/t-gate-choices-map-to-reports.test.ts` | 文件头 `1-12` | 门上的选项必须是协议里有报告方式的选项。本次没跑 |
| P1 | `core/aidlc-common/protocols/stage-protocol-reviewer.md` | skill 点名加载：`SKILL.md:111`；本站 `reviewer`：`intent-capture.md:12-15` | 改完成顺序或审阅条件时再读 |
| P1 | `core/tools/aidlc-message-store.ts` | `takePersonsOwnLine` 的注释指向它：`aidlc-orchestrate.ts:11387-11389` | 改「人的话怎么进 `next`」时再读 |
| P1 | `core/tools/aidlc-reply-reader.ts` | `aidlcEntryWords` 的 import：`aidlc-orchestrate.ts:142-147` | 同上 |
| P2 | `harness/cursor/skills/aidlc/tables.md` | 开头声明表不是派发逻辑 `3`；`intent-capture` 排在 1.1 `40` | 给人看的镜像。引擎读编译图 |
| P2 | `harness/cursor/skills/aidlc/composer.md` | skill 在 compose 时才让人读它：`SKILL.md:147-149` | 另一次入口 |
| P2 | `core/aidlc-common/protocols/stage-definition.md` | 未沿这条运行路径读 | 编译期 frontmatter 合同。运行时读的是图 |
| P2 | 其他 stage、传感器、学习环、插件、打包、其他 harness、`dist/` | 见暂不展开 | 特定场景再读 |

## 6. 证据

### 职责怎么拆

- 【事实】skill 写明 orchestrate 拥有站与站之间的路线，conductor 只负责把点名的那一步做好，并且不要在散文里重算路线。`SKILL.md:25`
- 【事实】引擎文件头写明它站在 skill 旁边，读工作记录和编译图，stdout 上只打一条 directive。普通 `next` 不改 Plan Approval。`aidlc-orchestrate.ts:1-16`
- 【事实】`report` 是写的一半：它自己不重写转移，只按门的状态派发 `aidlc-state.ts`。`19959-19990`
- 【事实】`conductor.md` 重申引擎决定下一站，生命周期只走 `aidlc-orchestrate.ts report`。`conductor.md:6-7`；`34-35`
- 【事实】Cursor 没有 plan/todo hook，也没有 statusline。stop hook 不能拒绝停止，转发循环只靠 skill 自己。`SKILL.md:117-122`
- 【事实】协议：除初始化三站外，每一站都要人明确批准才能继续。`stage-protocol.md:191`
- 【事实】`computeGate`：initialization 返回 false；walking-skeleton 那一站才可能是 `"unresolved"`；其余 EXECUTE 返回 true。`9946-9960`

### 注释和文档与实现不一致

- 【事实】同一文件开头写「SKILL.md 还没调用这个文件，只被单元测试锻炼」，后面的子命令表又写 `next` 和 `report` 是 LIVE。`aidlc-orchestrate.ts:1-6` 对比 `27-37`。skill 已经在跑这两个命令。`SKILL.md:39-42`。以 LIVE 段和 skill 为准。
- 【事实】`report` 注释写 `aidlc-state.ts` export 不出东西，导入 `handle*` 会构建失败，所以只能 spawn。`19964-19966`。同文件却从 `aidlc-state.ts` import 了 `guardPreflight`、`parkWorkflow` 等。`779-790`。`handleApprove` 确实没有 export。`aidlc-state.ts:8265`。转移子命令仍走 spawn；「什么都不 export」这句不成立。
- 【事实】persona 注释把路径写成 `.claude/aidlc-common/conductor.md`。`9081-9088`。`readConductorPersona` 用的是 `resolveHarnessPath(["aidlc-common", "conductor.md"])`。`9094-9095`。源码作者文件是 `core/aidlc-common/conductor.md`。
- 【事实】决策注释写本处理器不发出 `present-gate` 和 `dispatch-subagent`，并只列举了 `run-stage` / `invoke-swarm` / `print` / `error` / `ask` / `done`。`8904-8909`。这条主路径的 `kind` 表里还有 `load-steering`、`parked`、`notice`。`SKILL.md:80-90`。`DirectiveKind` 仍包含 `present-gate` 和 `dispatch-subagent`。`aidlc-directive.ts:104-115`。`speaksToPerson` 仍把 `present-gate` 当成会对人说话。`aidlc-orchestrate.ts:1614-1616`。`present-gate` 对象出现在指令合同文件当 main 运行时的示例数组里，不是 `routeNext` 的发出点。`aidlc-directive.ts:3810-3817`；`3888-3893`
- 【事实】开发者参考仍把 `present-gate` 列进「agent 会开口的 directive」。`docs/reference/03-orchestrator.md:512-515`。与 `8906-8909` 不一致。这条主路径上，门是 `run-stage` 之后的 `report`，再由 conductor 按协议展示，不是一条 `present-gate`。
- 【事实】`t114` 文件头写 `next` 不改变工作记录。`tests/unit/t114-orchestrate-next.test.ts:7-10`。引擎文件头写普通路由是只读的，但显式 `next config set` 可以改设置，人要求的 `--stage` / `--phase` 跳跃会先执行再路由。`aidlc-orchestrate.ts:10-16`。测试头的「一律不改」比文件头窄。本次没有运行该测试。

### 这一站

- 【事实】slug `intent-capture`，phase `ideation`，`execution: ALWAYS`，`mode: inline`，`summary_confirmation: required`，reviewer 是 `aidlc-product-lead-agent`，产物是 `intent-statement`、`stakeholder-map`、`intent-capture-questions`。`intent-capture.md:2-19`
- 【事实】`scopes` 列出 enterprise、feature、mvp、poc，没有 classic。`26-30`
- 【事实】测试源码期望 classic 计划拒绝 `"intent-capture" is not part of the classic plan.`。`tests/unit/t340-default-scope-resolver.test.ts:128`。这是测试里的字符串，不是本次运行结果。
- 【事实】阶段表声明自己不是派发逻辑，引擎读编译后的 `data/stage-graph.json`。`tables.md:3`；`31`。与 `loadGraph` 读 `stage-graph.json` 一致。`aidlc-graph.ts:1396-1407`。`loadStageGraph` 同样读该文件。`aidlc-lib.ts:36666-36686`
- 【事实】Step 6 只写 `--result <outcome>`，没有写 `awaiting-approval` 和 `approved` 的先后。那两步在 skill 的完成序列里。`intent-capture.md:231-233`；`SKILL.md:114-115`。两边一致的是：转移只能走 `report`，不能在散文里做。

### 未知

- 【未知】安装进 Cursor 之后 `{{INVOKE}}` 被替换成的具体命令。源码 skill 里仍是这个记号。本次没有读 `dist/`。
- 【未知】某个真实项目、某个 scope 上，当前站是不是 `intent-capture`。上面的「第一道人批的门」是从阶段顺序、`scopes` 和 `computeGate` 推的，没有对一份活的 `aidlc-state.md` 跑过 `next`。
- 【未知】`intent-capture` Step 2 到 Step 5 的提问和产物细节没有逐行核对。标题在 `101`、`151`、`158`、`200`。文件仍是 P0，后续写作要自己读完。
- 【未知】`takePersonsOwnLine` 在 Cursor 上是否常被走到。成功路径假定 argv 里已经有人的请求。`SKILL.md:46` 要求把 `$ARGUMENTS` 原样交给第一次 `next`。
- 【事实】本次没有运行测试，也没有在 Cursor 里走一轮。
