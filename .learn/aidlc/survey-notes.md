# 调查笔记：aidlc 主路径

证据索引，给后续写作用。不是讲义。没有运行测试。

## 1. 规模

**中。**

- 入口一个：人在 Cursor 里输入 `/aidlc`（`harness/cursor/onboarding.fills.ts` 的 `invoke`）。指挥技能的转发循环把参数交给 `core/tools/aidlc-orchestrate.ts` 的 `next`。
- 主路径一条：空工作区、描述没有点名计划。第一次 `next` 返回 `print`（让指挥提议计划）；指挥再跑 `next --propose <plan> --request <id>`，引擎返回 `ask`（`scope-confirm`）。停在这条问句。同意之后的创建和第一次阶段步骤只记到边界，不往下走。
- 必读文件 6 个（契约：6 到 12 为中）。同一目录里的其他分支没有算进必读。

必读：

1. `harness/cursor/skills/aidlc/SKILL.md`
2. `core/tools/aidlc-orchestrate.ts`
3. `core/tools/aidlc-lib.ts`（`readNextLine`、`validScopes`、`loadScopeMetadata`）
4. `core/tools/aidlc-question-store.ts`（`saveQuestion`、`readQuestion`）
5. `core/tools/aidlc-directive.ts`（`validateDirective`、`withAgentNotes`、`numberedProseQuestion`）
6. `core/aidlc-common/conductor.md`

## 2. 边界六问

1. **边界：** 从 Cursor 里的 `/aidlc` 加一段描述，到引擎打出并校验过的一条指令。自由文本且还没有工作流时，序列是提议计划的 `print`，然后是计划确认 `ask`。
2. **位置：** 指挥技能只转发和照指令做事。下一步由 `aidlc-orchestrate.ts` 的 `next` 决定。指令形状和校验在 `aidlc-directive.ts`。人设正文在 `conductor.md`，由引擎在第一次实质 `run-stage` 读入，不在这次计划问句上。
3. **问题：** 人说了要做什么之后，指挥不能自己选下一步。引擎每次只给一条。没有点名计划时，先问用哪个计划，不直接开阶段。
4. **内部：** 转发循环、`parseNextFlags` / `readNextLine`、Branch 8、`planProposalDirective`、`freshWorkOfferDirective`、`scopeConfirmAskDirective`、问题存档、`emit` → `validateDirective`。人设文件和挂载点属于这条路径的边界，不属于计划问句本身。
5. **依据：** 上面 6 个必读文件，加 `harness/cursor/onboarding.fills.ts` 的入口名。测试文件只用来确认有调用，没有运行。
6. **仍不可得：** 见文末未知项。

未知项：

- 没有运行测试，不知道现有用例是否通过。
- 源码技能里的命令仍是 `{{INVOKE}}`。打包后 Cursor 实际执行的那串命令没有核对。安装和打包不在这次范围内。
- 人在问句上选「就用这个计划」之后，`confirm_command` 会再进 `next --scope`。创建打印和第一次 `run-stage` 怎么从阶段图选出来，没有沿那条读完。
- `takePersonsOwnLine` 在代理已经传入描述时，何时把 `flags.intent` 换成人自己那一行，只读到替换条件和开头几支，没有把每个分支读完。

## 3. 本次主路径

起点：空工作区。人输入 `/aidlc`，后面是没有计划名的描述（技能里的例子形状是要做的事，不是 `classic` 这类计划名）。

1. 转发循环从 `/aidlc` 被调用时开始。第一次 `next` 把 `$ARGUMENTS` 原样送进引擎，不预先拆旗标。`harness/cursor/skills/aidlc/SKILL.md` `The Forwarding Loop`。第一回合在 `next` 返回前先说一句关于这个请求的话，同文件 “the very first turn”。
2. `main` 认子命令 `next`，调用 `handleNext`，再进 `routeNext`。`core/tools/aidlc-orchestrate.ts` `main`、`handleNext`、`routeNext`。
3. `parseNextFlags` 先调用 `readNextLine`（`core/tools/aidlc-lib.ts`）。纯描述返回 `kind: "words"`，不走 `readingStep` 早退。词进入 `intentWords`，再写成 `flags.intent`。`parseNextFlags`、`readNextLine`、`flags.intent = intentWords.join`。
4. `routeNext` 调用 `takePersonsOwnLine`。没有 `--request`、且这条描述已经在旗标里时，引擎仍可能按人自己的 `/aidlc` 行改 `flags.intent`。细节见未知项。
5. 没有状态文件、只有这段描述、没有 `--scope`、也没有位置上的计划名：Branch 8 调用 `freshWorkRoute`。还没有 `--propose` 时，`freshWorkRoute` 调用 `planProposalDirective`。
6. `planProposalDirective` 用 `saveQuestion` 把描述存成 `origin: "front"`、`proposedScope` 为空的问题，返回一条 `print`。打印告诉指挥：读描述，提议一个计划，再跑 `next --propose <plan> --request <id>`；没有现成计划就 `--propose compose`。计划名单来自 `planChoicesLine`（`validScopes`、`loadScopeMetadata`、`effectiveScopeCostSummary`）。指挥不得自己创建工作流。
7. `emit` → `prepareEmission` → `validateDirective`。不合法就拒绝打出并退出。合法则 `writePrepared` 把这一条 JSON 写到标准输出。
8. 指挥按 `print` 的 `message` 执行，再调用 `next --propose <plan> --request <id>`。`routeNext` 用 `readQuestion` 读回那条问题。`origin === "front"` 且还没有 `proposedScope` 时，调用 `freshWorkOfferDirective`。
9. 提议的不是 `compose` 时，`freshWorkOfferDirective` 调用 `scopeConfirmAskDirective`，得到 `kind: "ask"`、`ask_type: "scope-confirm"`。三个选项：按该计划做（`confirm_command` 是 `next --scope <plan> --request <id>`）、换一个计划（没有命令）、按这个任务另做一份计划（`compose_command`）。
10. 打出前 `withAgentNotes` 给这个 `ask` 补上 `question_note`，并用 `numberedProseQuestion` 写入 `numbered_prose_question`。校验仍是同一次 `validateDirective`。
11. 指挥按技能里 `ask` 那一行展示问句并结束回合。这是这条路径上第一次要人回答的下一步。

人设不在第 6 步和第 9 步的指令上。`readConductorPersona` 读 `conductor.md`。挂载在构造 `run-stage` 时：`isFirstRunStageOfWorkflow` 在没有状态文件时返回 false，初始化阶段也不挂。所以要等工作流已经存在、且这是第一次实质阶段步骤，`conductor_persona` 才会出现。Cursor 这条注释写的是命令不带聊天名，因此只送这一次。

## 4. 暂不展开

- 描述以计划名开头（`plan-word` / 位置计划名，Branch 7b）：不先问计划，直接走向创建。
- 显式 `--scope` 且没有工作流（Branch 9a），以及只有计划名、没有描述的 `planNamedAloneStep`。
- `--propose compose`、`composeOfferAskDirective`、Branch 4c、`harness/cursor/skills/aidlc/composer.md`。
- 已经有工作流时的自由文本（Branch 9c）、`--new-intent`（Branch 4a）。
- 同意计划之后的 `createPrintDirective`、`intent-create`、第一次 `run-stage` 的阶段正文、规则分片、门和评审。
- 安装、打包、其他 harness、`{{INVOKE}}` 替换结果。
- `question-rendering.md`、`tables.md`：计划问句的编号正文由引擎写入；annex 是 Cursor 展示阶段协议问句的约定，不是这条 `ask` 的数据来源。
- Code Generation 的 plan approval。那是另一条「计划」，不是这次开工前的计划问句。
- `report`、`park`、守卫恢复、实用命令（`--status`、`--doctor`、`config`）。

## 5. 文件优先级

| 文件 | 关键符号 | 优先级 | 原因 |
|---|---|---|---|
| `harness/cursor/skills/aidlc/SKILL.md` | 转发循环；`ask` / `print` 行；第一回合那一句 | P0 | 指挥何时把参数原样送进 `next`，以及拿到 `print` 和 `ask` 之后做什么 |
| `core/tools/aidlc-orchestrate.ts` | `main`、`handleNext`、`routeNext`、`parseNextFlags`、`freshWorkRoute`、`planProposalDirective`、`freshWorkOfferDirective`、`scopeConfirmAskDirective`、`emit`、`prepareEmission`、`readConductorPersona`、`isFirstRunStageOfWorkflow` | P0 | 自由文本如何变成一条指令；人设挂在哪 |
| `core/tools/aidlc-lib.ts` | `readNextLine`、`readNextTokens`、`validScopes`、`loadScopeMetadata` | P0 | 一行是自由文本还是计划名，在这里定一次 |
| `core/tools/aidlc-question-store.ts` | `StoredQuestion`、`QuestionOrigin`、`saveQuestion`、`readQuestion` | P0 | `--request` 读回的就是这里的记录；`origin` 和空的 `proposedScope` 决定第二次 `next` 走问句 |
| `core/tools/aidlc-directive.ts` | `validateDirective`、`withAgentNotes`、`numberedProseQuestion`、`ScopeConfirmAskDirective`、`PrintDirective`、`RunStageDirective.conductor_persona` | P0 | 每次只打出一条合法指令；问句的编号正文在打出前补上 |
| `core/aidlc-common/conductor.md` | 文首「baked into the first next」 | P0 | 人设正文。送达时机以引擎代码为准，和这几句不完全同一句 |
| `harness/cursor/skills/aidlc/question-rendering.md` | `Mechanism` | P1 | Cursor 没有结构化提问工具时，阶段协议问句如何编成编号散文。计划问句的正文来自引擎 |
| `core/tools/aidlc-orchestrate.ts` | `createPrintDirective`、Branch 7b、Branch 9a、Branch 9c、Branch 4a | P1 | 改「同意之后」或「已经有工作」时要读。这次主路径停在问句前 |
| `harness/cursor/skills/aidlc/composer.md` | 另做一份计划 | P2 | 第三个选项。这次不走 |
| `harness/cursor/skills/aidlc/tables.md` | 范围表 | P2 | 人读的阶段表，不是 `next` 的路由 |

行号见下一节。改代码后行号会移，符号名仍是入口。

## 6. 证据

- 【事实】技能入口名是 `/aidlc`。`harness/cursor/onboarding.fills.ts:9` `invoke`。
- 【事实】转发循环第 1 步把 `$ARGUMENTS` 原样交给 `engine orchestrate next`，不自己解析旗标。开头若是计划名，才把那个名字拆成单独参数。`harness/cursor/skills/aidlc/SKILL.md` `The Forwarding Loop`（约 33–46 行）。
- 【事实】每次 `next` 只返回一条 JSON。同段。
- 【事实】`ask` 要按给出的选项展示并结束回合；`print` 要照 `message` 做。同文件 “Acting on a step”（约 78–89 行）。
- 【事实】第一次 `next` 返回前，先说一句关于人自己请求的话。同文件 “the very first turn”（约 70 行）。
- 【事实】`main` 在子命令 `next` 时调用 `handleNext`。`core/tools/aidlc-orchestrate.ts:24009` `main`，`:24209` `case "next"`，`:11349` `handleNext`，`:11832` `routeNext`。
- 【事实】`parseNextFlags` 把怎么读这一行交给 `readNextLine`。只有 `value`、`plan-word`、`command`，以及非工作区名词的 `noun`，才立刻返回 `readingStep`。`core/tools/aidlc-orchestrate.ts:6976-6996`。
- 【事实】没有被旗标吃掉、开头也不是已安装计划名时，`readNextLine` 返回 `{ kind: "words" }`。计划名只有在整行就是它，或「计划名 + 一个带空格的参数」时才是 `exact`；松散接在计划名后面的句子是 `plan-word`。`core/tools/aidlc-lib.ts:39916` `readNextLine`，`:39997-40012`。
- 【事实】`words` 不走上面的早退。剩下的词在 `:7423-7424` 写入 `flags.intent`。
- 【事实】`routeNext` 在分支之前调用 `takePersonsOwnLine`。`core/tools/aidlc-orchestrate.ts:11836`。函数说明在 `:11382-11431`：代理没把人的话传进来时，引擎可以按人自己的 `/aidlc` 行来读。`:11432-11434` 在没有会话、只读探测、或已经有 `--request` 时返回 null。
- 【推断】代理已经传入描述时，`takePersonsOwnLine` 仍可能改 `flags.intent`。依据是 `:11435` 起在 `flags.intent !== undefined` 时继续读最新消息。替换条件没有逐支读完。
- 【事实】没有状态文件、只有描述、没有 scope、没有位置计划名时，Branch 8 调用 `freshWorkRoute`。`core/tools/aidlc-orchestrate.ts:13668-13714`。
- 【事实】`freshWorkRoute` 在没有 `--propose` 时返回 `planProposalDirective`。`:7784-7811`。
- 【事实】`planProposalDirective` 把描述存成 front 问题（`proposedScope` 为 `""`），返回 `print`。再跑同一 `--request` 且仍是没有计划的 front 问题时，复用同一条。`:7655-7688`。`printDirective` 在 `:5294`。
- 【事实】`saveQuestion` 写入 `StoredQuestion`：`id`、`text`、`proposedScope`、`origin`（`"front"` | `"routing"` | `"compose"`）。`core/tools/aidlc-question-store.ts:43` `QuestionOrigin`，`:51` `StoredQuestion`，`:478` `saveQuestion`。`readQuestion` 在 `:240`。
- 【事实】第二次 `next` 带 `--propose` 和 `--request` 时，读回的问题若是 front 且 `proposedScope` 为空，就调用 `freshWorkOfferDirective` 并返回。`core/tools/aidlc-orchestrate.ts:12151-12189`。`--propose` 缺计划名或缺 `--request` 时，先返回给指挥的 `print`，不报给用户的 `error`。`:12080-12108`。
- 【事实】提议的计划不是 `compose` 时，`freshWorkOfferDirective` 返回 `scopeConfirmAskDirective`。`:7577-7613`。
- 【事实】该 `ask` 的 `ask_type` 是 `"scope-confirm"`，`response_route` 是 `"next"`。三个 `choices`：按提议的计划做（带 `confirm_command`）、换计划（无命令）、另做一份（带 `compose_command`）。`:3909-3953`。类型在 `core/tools/aidlc-directive.ts:685` `ScopeConfirmAskDirective`。
- 【事实】`emit` 经 `prepareEmission` 调用 `validateDirective`。不合法则写到错误输出并 `process.exit(1)`，不打出指令。`core/tools/aidlc-orchestrate.ts:2981` `emit`，`:1863` `prepareEmission`，`:2121-2127`。`validateDirective` 在 `core/tools/aidlc-directive.ts:1959`。`writePrepared` 把序列化结果写到标准输出 `:2410-2411`。
- 【事实】`kind` 必须是 `VALID_KINDS` 之一，多余的键会被拒绝。`core/tools/aidlc-directive.ts:104` `DirectiveKind`，`:1972-1990`。
- 【事实】打出前 `withAgentNotes` 对非 `agent_work` 的 `ask` 写入 `question_note`；`scope-confirm` 用 `planOfferQuestionNote`。有选项时写入 `numbered_prose_question`（`numberedProseQuestion`）。`core/tools/aidlc-directive.ts:1854-1876`，`numberedProseQuestion` 在 `:1720`。`prepareEmission` 调用 `withAgentNotes` 在 `core/tools/aidlc-orchestrate.ts:2068`。
- 【事实】`AskDirectiveBase` 写明 `numbered_prose_question` 由引擎拥有编号，只在问题是指挥自己的活时才缺。`core/tools/aidlc-directive.ts:653-675`。
- 【事实】`readConductorPersona` 读 `aidlc-common/conductor.md`。文件缺失则返回 null，不因此失败。`core/tools/aidlc-orchestrate.ts:9081-9101`。
- 【事实】`conductor_persona` 在构造 `run-stage` 时挂上。条件包括 `isFirstRunStageOfWorkflow`。该函数在 `stateContent` 为空时返回 false，初始化阶段也返回 false。`:10389-10412`，`:9298-9317`。字段声明在 `core/tools/aidlc-directive.ts:514-519`，注释写的是只在工作流的第一次 `run-stage` 上。
- 【事实】`conductor.md:9-13` 写的是引擎把它烤进会话的第一次 `next`，技能不按路径引用它。
- 【推断】计划问句这两次 `next`（`print` 和 `scope-confirm` `ask`）不会带 `conductor_persona`。依据是挂载点只在 `run-stage` 构造里，且没有状态文件时 `isFirstRunStageOfWorkflow` 为 false。人设正文仍是 `conductor.md`，但要到第一次实质阶段步骤才进指令。
- 【事实】同段注释写 Cursor 的命令不带聊天名，因此人设只送这一次，不按聊天重复。`core/tools/aidlc-orchestrate.ts:9290-9297`。
- 【事实】「同意」那条命令是 `next --scope <计划> --request <id>`。`:3928` `confirmCommand`。没有状态且 scope 来自旗标时，Branch 9a 走 `createPrintDirective`。`:13717-13756`，函数在 `:7989`。这次没有读创建之后如何选出第一次 `run-stage`。
- 【事实】`question-rendering.md:48-51` 写 Cursor 没有结构化提问工具，问句渲染成编号散文。这是阶段协议问句的展示约定。计划确认问句的编号字符串由 `numberedProseQuestion` 写入指令。
- 【事实】存在调用 `next --propose` / 覆盖 `planProposalDirective` 的测试文件，例如 `tests/unit/t-plan-question-waits-for-the-person.test.ts`、`tests/unit/t-newest-request-wins.test.ts`（文件头写了 `function:planProposalDirective`）、`tests/harness/fixtures.ts` 约 964–1008 行。没有运行它们。
- 【未知】这些测试现在是否通过。
- 【未知】源码里的 `{{INVOKE}}` 在 Cursor 安装里换成哪条命令。`harness/cursor/skills/aidlc/SKILL.md` 仍是令牌。打包不在这次范围内。
- 【未知】`takePersonsOwnLine` 在「代理已传入描述」时的每一条替换是否都会发生。只读了 `:11432-11459` 附近。
