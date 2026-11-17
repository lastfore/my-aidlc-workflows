这份只给练习的分级提示，不给完整答案，也不给补丁。
示例和阶段名以指南的 [打开顺序](module-guide.md#打开顺序)、[主路径索引](module-guide.md#主路径索引) 为准。
第三级只缩到符号；核对仍在指南和执行路径各自的核对块。

## 题 1

1. 先分清「谁决定下一站」和「谁把这一站做完」。停下来的时机是站做完之后，不是决定要不要做这一站。
2. 运行时协作图里有四个人物。范围内只有问下一步、做 `intent-capture`、把结果交回。初始化三站会自己往下走，但它们的正文不在这里。
3. 看指南八条结论的前三条和第五条，以及文末「暂不展开」。符号是转发循环、`computeGate`、conductor 正文里生命周期只走 `report` 的那一段。

## 题 2

1. 六个符号不要平均展开。先找出「问下一步」的那一条链，再找出「交回通过」的那一条链。
2. `next` 不执行站正文。`gate` 的 true 来自阶段，不来自 `execution`。通过不在编排文件里重写转移。
3. 入口侧看 `engine-orchestrate`、`main`、`handleNext`、`routeNext`。工单侧看 `buildRunStageDirective`、`computeGate`、`RunStageDirective`。落账侧看 `handleReport`、`spawnState`、`handleApprove`。审阅协议正文不要打开。

## 题 3

1. 阶段名必须用索引里的七个标题。本示例的工作记录是推断出来的最小例子，入口不是测试 13。
2. 前两步几乎不落新字段。工单上的 `kind`、`stage`、`gate` 出现在交出 `run-stage` 的那一步。复选框变成等待批准，发生在打开门的那一步，不是澄清意图的那一步。
3. 在途判定看 Branch 10 和 `currentIsInFlight`。工单字段看 `emitForSlug` 和 `buildRunStageDirective`。三份产物名看这一站 frontmatter 的 `produces`。门上看 `handleReport` 和 `nextInScopeStage`。通过之后的 `kind` 看 `done`。

## 题 4

1. 四件事里只有文件路径是这条路径已经写明的。另外三件，材料自己标了还不能核实，不要用一个看起来合理的值填上。
2. 执行要求是否附上，取决于是不是第一份 `run-stage`，以及这个聊天里有没有已经拿到正文。本示例没有活的工作记录。下一站的显示名要等计划和范围都在，才能算。安装后的命令前缀在未知项里。
3. 路径看 `stageFileFor`。是否附上执行要求看 `readConductorPersona`，以及把它放进 directive 的条件。显示名看 `nextInScopeStage`。`{{INVOKE}}` 看指南文末「未知项」，不要去读 `dist/` 来填一个具体命令。

## 题 5

1. 把两个问题拆开：这一站会不会被放进计划，以及做完后会不会停下来等人。ALWAYS 只回答其中一件。
2. 改法的代价是这一站自己往下走，和阶段协议对初始化以外各站的要求相反。测试 13 专门防止把 `gate` 从 `execution` 推出来。它的参数里有 `--stage`。
3. 看 `computeGate` 和 `isSkeletonGateStage`，以及这一站 frontmatter 的 phase 和 `execution`。测试源码在 `t114` 的测试 13。本示例的在途入口在执行路径的示例约定里，和那条跳跃并排放着。

## 题 6

1. 编号选项只说明 Cursor 上怎么画。画成编号，不等于用同一条记录落账。合同里有一种叫 `present-gate` 的种类，不等于这条成功路径把它交了出来。
2. 批准是生命周期报告。先报告等待批准，人选择之后再报告通过或要求修改。注释说「什么都不 export」，要和同文件实际 import 的函数分开看；`handleApprove` 本身仍然没有 export。
3. 两类记录看阶段协议，以及 skill 完成序列里对这扇门禁止调用问题日志的那一句。门上的选项看 `t-gate-choices-map-to-reports`。派发看 `handleReport`、`spawnState`、`validateSlugInState`。复选框仍是 in-progress 就报告通过时，在 `handleReport` 里看它先放入哪一条子命令。

## 题 7

1. 不要改生产代码，也不要把测试 13 的 `--stage` 拿来当这条新用例的入口。已有在途用例的当前站不是 `intent-capture`。
2. 从已经能走出在途 `run-stage` 的那份播种改起，同时改当前站、这一站的复选框和 Scope。等待批准的复选框会走到只展示门的形状，不是站正文还要执行的这一轮。
3. 看 `seedStateFile`、`MID_IDEATION` 和 `runNext`。在途复选框在那份播种里写成 `[-]`。测试 13 的 `runNext` 带了 `--stage`，新用例不要带。断言只要求 `kind`、`stage`、`gate` 三个字段。提前返回仍在 Branch 10 里，命中了就只改播种。
