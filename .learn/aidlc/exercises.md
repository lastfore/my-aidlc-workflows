先读指南的[打开顺序](module-guide.md#打开顺序)。

再读指南的[主路径索引](module-guide.md#主路径索引)。提示在 [exercises-hints.md](exercises-hints.md)，答案不在本文件。

每题的验收清单就是完成要求。

## 题 1

空工作区里，人输入 `/aidlc Build a notes app`。这段描述的开头不是计划名。谁决定下一步？这条路径最后停在哪一句给人看的话？工作流创建了没有？

A · 易 · 职责边界

- 相关符号：`harness/cursor/skills/aidlc/SKILL.md` 的 `The Forwarding Loop`，`core/tools/aidlc-orchestrate.ts` 的 `freshWorkRoute`
- 验收：
  - 说出指挥只把参数送进引擎，并照引擎返回的那一条去做
  - 说出引擎先请指挥提议一个计划，再打出计划确认问句
  - 说出这条路径停在那条问句上，创建还没有开始

## 题 2

同一次示例里，指挥第一次把参数送进引擎时，有没有把 `Build a notes app` 拆成旗标？引擎把这一行读成哪一种读法，工作描述写进了哪个字段？走到这一步时，有没有计划名，有没有问题文件？

B · 易 · 原样转发与自由文本

- 相关符号：`harness/cursor/skills/aidlc/SKILL.md` 的 `The Forwarding Loop`，`core/tools/aidlc-lib.ts` 的 `readNextLine`，`core/tools/aidlc-orchestrate.ts` 的 `parseNextFlags`
- 验收：
  - 说出第一次参数仍是整段 `Build a notes app`，指挥没有拆旗标
  - 说出读法是 `words`，`flags.intent` 写成 `Build a notes app`
  - 说出这一步还没有计划名，也还没有问题文件

## 题 3

按主路径索引，从「存下空计划的请求」走到「校验后结束回合」。示例里指挥提议的计划是 `bugfix`。请分开回答下面三件事。

1. 第一次问题文件和第二次问题文件，计划名各是什么状态？问句的 `question_id` 用哪一次的编号？
2. 人看到的三个选项，各自有没有命令？有命令的话，旗标是什么，`--request` 用哪一次编号？
3. 编号正文里，数字 2 对应哪一个选项？三个选项之外多出来的那一行 Other，是不是 `choices` 里的第四项？

B · 中 · 两次存档与计划确认问句

- 相关符号：`core/tools/aidlc-orchestrate.ts` 的 `planProposalDirective` 和 `scopeConfirmAskDirective`，`core/tools/aidlc-directive.ts` 的 `numberedProseQuestion`
- 验收：
  - 说出第一次 `proposedScope` 为空，第二次写成 `bugfix`，第一条的计划名保持为空
  - 说出 `question_id`、按该计划做的 `--request`、另做一份的 `--request` 都是第二次编号
  - 说出按该计划做的命令含 `next --scope bugfix --request`，换一个计划没有 `command`，另做一份的命令含 `next compose --request`
  - 说出数字 2 对上「换一个计划」那个选项；Other 是编号正文多出来的一行，不是第四个带命令的选项
  - 命令前面的安装前缀在源码里仍是占位；装好的 Cursor 换成哪一串，答「还不能确定」

## 题 4

第一次把描述存成问题文件时，计划名为什么留空？这条 `print` 是在问人，还是在吩咐指挥再做一次？这两次打出的指令里为什么没有人设？人设要挂上时，指令的 `kind` 是什么，还要多满足一个什么条件？

C · 中 · 提议步骤与人设挂载

- 相关符号：`core/tools/aidlc-orchestrate.ts` 的 `planProposalDirective` 和 `isFirstRunStageOfWorkflow`，`core/aidlc-common/conductor.md`
- 验收：
  - 说出工具不读这段描述在说什么，所以先把原话存下来，计划名留空，再把已安装的计划名单交给指挥去提议
  - 说出空的 `proposedScope` 让第二次仍能认出「还没有计划」；若第一次就已经写上计划名，这条路径不会再走进 `freshWorkOfferDirective`
  - 说出这条 `print` 吩咐指挥再跑一次 `next`，这一次不结束回合
  - 说出这两次指令没有 `conductor_persona`；这个字段挂在 `kind` 为 `run-stage` 的指令上，并且要是已有工作流之后的第一次实质阶段步骤；没有状态文件时该判断为假

## 题 5

下面五件事，逐件写能确定的结论。材料标成未知的，答「还不能确定」。

1. 指令校验失败时，标准输出上有没有这条问句？进程怎样停？
2. 只看 `tests/unit/t-plan-question-waits-for-the-person.test.ts` 的文件头：它在锁「问句要等人回答」，还是在锁「人同意之后怎样创建」？这些测试现在是否通过？
3. 源码技能里的 `{{INVOKE}}` 在装好的 Cursor 里换成哪条命令？
4. 第一次 `next` 在描述已经传入时，会不会按人自己打的那一行改 `flags.intent` 的用字？会不会换掉整组参数？带 `--request` 的第二次调用呢？
5. 人同意这个计划之后，第一次阶段步骤怎样从阶段图里选出来？

D · 中 · 出口、测试与未知项

- 相关符号：`core/tools/aidlc-orchestrate.ts` 的 `validateDirective` 和 `takePersonsOwnLine`，`tests/unit/t-plan-question-waits-for-the-person.test.ts`
- 验收：
  - 第 1 件：写到错误输出并退出，标准输出没有这条指令，也没有问句回合
  - 第 2 件：文件头锁的是计划问句要等人回答，工作要等这个人的消息记录之后才开始；测试现在是否通过，答「还不能确定」
  - 第 3 件：答「还不能确定」
  - 第 4 件：某一次真实运行会不会改用字，答「还不能确定」；同时写明这个分支不换掉整组参数，带 `--request` 的第二次在函数开头就返回
  - 第 5 件：答「还不能确定」
