先读指南的 [打开顺序](module-guide.md#打开顺序)。
阶段名以指南的 [主路径索引](module-guide.md#主路径索引) 为准。
[打开顺序](module-guide.md#打开顺序) 决定先建立哪一层认识；[主路径索引](module-guide.md#主路径索引) 决定追踪题用哪一套阶段名。

## 题 1

这条路径要解决的问题是什么？它从哪里开始，负责到哪里停？人、conductor、引擎、state 工具在这一轮里各自交出去什么？打包、插件、其他站的正文，为什么不在这次范围内？

A · 易 · 职责边界

- 相关符号：`harness/cursor/skills/aidlc/SKILL.md` 的转发循环，`core/tools/aidlc-orchestrate.ts` 的 `computeGate`，`core/aidlc-common/conductor.md`
- 验收：
  - 用自己的话写明：下一站由工作记录和编译好的阶段图决定，conductor 不在对话里重算。
  - 起点是 Cursor 里继续一次已经在进行的工作，当前站是 `intent-capture`，复选框还没有完成。
  - 停住的地方是这一站的批准门。人明确批准之后，引擎交回这一步已记下；下一站的正文不在这条路径里。
  - 四者的交接说得清：人继续工作，conductor 问下一步并做这一站、再交回结果，引擎读记录和阶段图并派转移，state 工具负责门能不能打开、批准能不能落账。
  - 至少写出两类这次不读的东西，并说明它们不在「问下一步、做 `intent-capture`、把结果交回」这条范围里。

## 题 2

人在 Cursor 里说「继续」，第一次 `next` 要经过哪些入口，才变成一条 `run-stage`？阅读路线点名的六个符号，各自吃进什么、交出去什么？

A · 中 · 入口和核心接口

- 相关符号：`core/tools/aidlc.ts` 的 `engine-orchestrate`，`core/tools/aidlc-orchestrate.ts` 的 `main`、`handleNext`、`routeNext`、`buildRunStageDirective`、`computeGate`、`handleReport`，`core/tools/aidlc-directive.ts` 的 `RunStageDirective`，`core/tools/aidlc-state.ts` 的 `handleApprove`
- 验收：
  - 写出从 Cursor skill 的转发循环，到 `engine orchestrate`，再到 `main` 和 `handleNext` 的入口，并写明 `handleNext` 只把工作交给 `routeNext`。
  - 六个符号各有一句输入和一句输出：`routeNext`、`buildRunStageDirective`、`computeGate`、`handleReport`、`RunStageDirective`、`handleApprove`。
  - 写明这一站的 `gate` 为什么是 true：phase 是 ideation，不是 initialization，也不是 construction 里的骨架门。
  - 写明 `handleApprove` 没有 export，通过是由 `handleReport` 用 `spawnState` 拉起 state 子进程之后才落账。
  - 答案停在这六个符号。不展开审阅协议正文、其他站正文、composer。

## 题 3

按主路径索引的阶段名，把本示例走完。每个阶段写两句：这一步之后业务对象变成什么，相对上一步哪个字段变了。

B · 中 · 主路径上的数据变化

- 相关符号：`core/tools/aidlc-orchestrate.ts` 的 Branch 10、`currentIsInFlight`、`emitForSlug`、`buildRunStageDirective`，`core/aidlc-common/stages/ideation/intent-capture.md`，`core/tools/aidlc-lib.ts` 的 `nextInScopeStage`
- 验收：
  - 七个阶段名与索引逐字相同，顺序不变，不另起一套名字。
  - 起点写明：Scope 为 `poc`，Current Stage 为 `intent-capture`，复选框为 in-progress，第一次 `next` 没有额外参数。这是站正文还要执行的那一轮，不是带 `--stage` 的点名跳跃。
  - 第 1 步写明工作记录的 Current Stage、复选框、Scope 没有被这次 `next` 改写。
  - 第 3 步写明新出现的 `kind`、`stage`、`gate`，以及同一次赋值从节点抄来的 `phase`、`lead_agent`、`mode`、`produces`。
  - 第 5 步写明三份产物：意图说明、干系人图、这一站的提问记录。复选框在这一步还不变成通过。
  - 第 6 步写明复选框变成 awaiting-approval，交回的是带 `next_stage` 和产物位置的 `print`，人看到的选择是 Approve 和 Request Changes。
  - 第 7 步写明返回的 `kind` 变成 `done`。工作还没结束时才再问一次 `next`，下一站正文不在这条路径里。

## 题 4

还是这条示例。下面四件事，材料里已经能确定的，写出确定的内容；材料标成未知的，回答「还不能确定」，并写明缺的是什么。

1. `stage_file` 指向哪一份正文？
2. 这次 `next` 是否一定带上 `conductor_persona`？
3. 打开门时，`next_stage` 的显示名是什么？
4. 安装进 Cursor 之后，`{{INVOKE}}` 被换成什么命令？

B · 中 · 分清已核实和尚未核实

- 相关符号：`core/tools/aidlc-orchestrate.ts` 的 `stageFileFor`、`readConductorPersona`，`core/tools/aidlc-lib.ts` 的 `nextInScopeStage`，`harness/cursor/skills/aidlc/SKILL.md`
- 验收：
  - 第 1 件写出阶段文件在 ideation 下的 `intent-capture` 正文。可以写它和 `kind`、`gate` 在同一次赋值里，不是另一次 `next`。
  - 第 2、3、4 件的答案都包含「还不能确定」。
  - 第 2 件说明：缺一份活的工作记录，不能确定这次是不是工作流的第一份 `run-stage`。出现条件可以写（第一份，或这个聊天还没有这份正文），不要写成这次一定有，或一定没有。
  - 第 3 件说明：显示名来自当时的计划和范围，这次没有对一份活记录算过。不要自己补一个站名。
  - 第 4 件说明：源码 skill 里仍是这个记号，这次没有读安装后的 `dist/`。不要把记号换成一个你猜测的命令。

## 题 5

有人提议：`execution` 为 ALWAYS 的站做完后不必等人，把 `computeGate` 改成「不是 ALWAYS 才过门」。对 `intent-capture` 来说，这会破坏哪条已经讲过的规则？现有哪一条测试在守卫现在的行为？测试 13 的入口和本示例的入口是不是同一种？不要改代码。

C · 中 · gate 为什么不看 ALWAYS

- 相关符号：`core/tools/aidlc-orchestrate.ts` 的 `computeGate`、`isSkeletonGateStage`，`core/aidlc-common/stages/ideation/intent-capture.md`，`tests/unit/t114-orchestrate-next.test.ts` 的测试 13
- 验收：
  - 写明 ALWAYS 回答的是这一站在不在计划里，`gate` 回答的是做完后停不停。
  - 写明这一站的 phase 是 ideation，所以现在是 true；initialization 才是 false。
  - 写明这个改法会让这一站不再向人要批准，和「除初始化三站外，每一站都要人明确批准」相反。
  - 点名 `t114` 的测试 13：它断言 `gate` 为 true，入口是 `next --stage intent-capture` 且范围为 `poc`。写明那是点名跳跃，本示例的入口是当前站已经在途、不带 `--stage`。
  - 材料没有运行过这组测试。答案依据测试源码里的断言即可；若你另外跑了，把运行结果和「材料未跑」分开写。

## 题 6

同事看到批准门上的编号选项，做了两件事：把 Approve 用问题日志记成一条 decision，并认为上一条引擎输出的 `kind` 是 `present-gate`，所以不必再 `report`。另一位同事说，编排文件的注释写着 state 工具什么都不 export，因此 conductor 在复选框还是 in-progress 时应该直接调用批准函数。这两处分别错在哪？要打开哪里核对？替代做法的代价是什么？不要改代码。

D · 中 · 用记录和测试定位错误落账

- 相关符号：`core/aidlc-common/protocols/stage-protocol.md`，`harness/cursor/skills/aidlc/SKILL.md` 的完成序列，`core/tools/aidlc-orchestrate.ts` 的 `handleReport`、`spawnState`，`core/tools/aidlc-state.ts` 的 `handleApprove`、`validateSlugInState`，`tests/unit/t-gate-choices-map-to-reports.test.ts`，`tests/unit/t115.test.ts`
- 验收：
  - 写明站内问题和批准门是两类记录：不是门上的问题先记日志再给人看；批准选择只走 `report`。
  - 写明这条成功路径上，`routeNext` 交出的是 `run-stage`。人看到的选择来自「等待批准」之后的 `print`，不是一条 `present-gate`。
  - 写明完成序列先 `report --result awaiting-approval`，人批准后再 `report --result approved`，Approve 出现在后一条的 `--user-input` 里。
  - 点名 `t-gate-choices-map-to-reports`：它检查门上的选项都有协议里的报告方式，并且不跑命令行。
  - 写明 `handleApprove` 没有 export；「什么都不 export」这句不成立，因为同文件还 import 了别的函数。非团队门的批准要求复选框已经是 awaiting-approval。
  - 写明复选框仍是 in-progress 时，`report --result approved` 会先派发 `gate-start`，再派发 `approve`。conductor 直接调用 state 工具的生命周期动词，是 skill 不允许的。
  - 替代做法（把 `handleApprove` export 出来在进程内调用）的代价要写到：转移规则会有两份，子进程自己持有的锁也盖不住。不要把它写成现在已经采用的做法。

## 题 7

`t114` 里，在途当前站变成 `run-stage` 的用例播种的是 `feasibility`。测试 13 又用点名跳跃看 `intent-capture` 的 `gate`。本示例因此要靠两条用例拼出来。请补一条范围很小的用例，让「当前站已是 `intent-capture`、复选框为 in-progress」直接走出 `run-stage`。

E · 中 · 为在途的这一站补一条回归

- 相关符号：`tests/unit/t114-orchestrate-next.test.ts` 的 `seedStateFile`、`runNext`，在途用例使用的 `MID_IDEATION`，测试 13
- 验收：
  - 只新增用例（可以在测试里改写已有的在途播种）。不改 `core/` 下的生产代码，不改已有用例的断言。
  - 这条新用例的工作记录里，Current Stage 是 `intent-capture`，这一站的复选框是 in-progress（`[-]`），Scope 是 `poc`。
  - 调用不带 `--stage`，也不带 `--phase`。
  - 断言同时包含 `"kind":"run-stage"`、`"stage":"intent-capture"`、`"gate":true`。
  - 用 bun 只跑这一条新用例，把命令和通过结果写进答案。没有跑整份 `t114` 也可以，写明你跑的是哪一条。
  - 若第一次得到的不是 `run-stage`，只许改播种内容，不许改 `routeNext`。答案里写明你多改了哪个字段，才避开 Branch 10 里那些本示例假定不会命中的提前返回。
