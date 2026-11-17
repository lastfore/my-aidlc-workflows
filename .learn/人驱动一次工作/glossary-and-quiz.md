这份只收术语、容易混淆的差别，以及不打开仓库也能做的自测。
示例仍是执行路径里那一次继续工作：工作记录的当前站已是 `intent-capture`。
核对在指南文末「核对与未知」。自测不附答案。

## 术语

### 转发循环

conductor 反复做的三拍：向引擎要下一步，做那一件事，再把结果交回。见指南的 [转发循环](module-guide.md#转发循环)。

### directive

引擎在标准输出上交给 conductor 的一张工单，每次一张，上面有 `kind`。见指南的 [directive](module-guide.md#directive)。

### gate

这一站做完后要不要停下来等人批准。见指南的 [gate](module-guide.md#gate)。

### 工作记录

这次工作做到哪一站的进度单，当前站和复选框都写在这里。见指南的 [工作记录](module-guide.md#工作记录)。

### 批准门

站做完后给人的那道选择题，选择再用报告交回引擎。见指南的 [批准门](module-guide.md#批准门)。

### conductor 正文

这一站里怎样把判断做对的执行要求。需要时由引擎放进工单，skill 不自己按路径去读。见指南的 [conductor 正文](module-guide.md#conductor-正文)。

### run-stage

这条成功路径上，引擎交给 conductor 去执行的那一种工单。这一张的站是 `intent-capture`。

### present-gate

合同里仍然列着的一种工单种类。这条成功路径上，人看到的门不是由这种工单交出来的。

### gate_only

复选框已经是等待批准、人再次进入时，打在 `run-stage` 上的标记。conductor 只展示已经打开的门，不把站正文再做一遍。

### 阶段图

编译之后的 `stage-graph.json`。引擎用它填工单上的阶段、文件和 `gate`。给人看的阶段表不是这份图。

## 容易混淆的概念

### gate 和 ALWAYS

`gate` 回答做完后停不停。`execution: ALWAYS` 回答这一站在不在计划里。`intent-capture` 两件都成立：它在计划里，并且做完要等人批准。把 ALWAYS 理解成「不用等人」，这一站就会自己往下走。

### present-gate 和批准门

`present-gate` 是合同里的一种 `kind`。这条路径上，人看到的 Approve 和 Request Changes，是 conductor 在报告「等待批准」之后，按协议讲出来的。那时引擎交回的是一条 `print`，上面才有下一站的显示名和产物位置。

### 阶段表和阶段图

阶段表把站排给人看，并声明自己不负责派发。引擎读的是编译后的阶段图。`classic` 不在 `intent-capture` 的 scopes 里；照着表的常见顺序往下数，会把这种计划也送进这一站。

### 问题日志和批准报告

站里普通的澄清问题，先记日志再给人看。批准门上的选择只走 `report`：先报告等待批准，人批准后再报告通过，要求修改则报告拒绝。Cursor 上两者都可以画成编号散文，画法相同，记录不是同一条。

### 站正文还要执行，和只展示已经打开的门

复选框为 in-progress 时，这一轮还要按站正文做完，再去开门。复选框已经是 awaiting-approval 时，再问下一步得到的仍是 `run-stage`，但带上 `gate_only`，只把已经打开的门再给人看。

### 「什么都不 export」和 `handleApprove` 没有 export

编排文件旁的注释说 state 工具什么都不 export，导入就会构建失败。这句和同文件已经 import 的其他函数不符。`handleApprove` 本身仍然没有 export。批准落账走的是 state 子进程，不是编排文件里的一份进程内转移。

## 综合自测

不打开仓库也能答。不附答案。

### 自测 1

给人看的阶段表已经把 `intent-capture` 排在初始化之后。若 conductor 不再向引擎要下一步，只照着这张表往下派，路线会多出哪个主人？`classic` 计划有什么风险？

### 自测 2

本示例第一次 `next` 返回时，工作记录里的复选框还是 in-progress。这次调用有没有把「通过」写进去？要等到哪一次报告，state 工具才会去落账？

### 自测 3

初始化阶段的 `gate` 和 `intent-capture` 的 `gate` 各是什么？站做完之后，conductor 对这两种站的下一步有什么不同？

### 自测 4

conductor 正文的作者文件在源码的哪一处？注释里的 `.claude/aidlc-common/conductor.md` 能不能当作这次读取用的路径？skill 为什么不让 conductor 自己按一个固定路径去读它？

### 自测 5

人离开之后再次进入，工作记录里这一站的复选框已经是等待批准。conductor 还应不应该把站正文再做一遍？工单上比普通 `run-stage` 多出来的标记是什么？

### 自测 6

引擎文件开头写着 skill 还没调用这个文件、只被测试锻炼。Cursor 上这次工作实际在跑哪两条子命令？判断「引擎还用不用」时，以哪一处为准？

### 自测 7

下面三件事，哪一件必须答「还不能确定」？另外两件，现有材料已经能确定什么？

- `intent-capture` 的 phase 是不是 ideation
- 人的原话没有放进参数时，从消息记录补回的分支在 Cursor 上是不是常被走到
- `handleApprove` 有没有 export

### 自测 8

人批准之后，引擎交回 `done`，并且这次工作还没有结束。下一站的正文要不要在同一次报告里做完？conductor 接下来只做哪一件事？

## 尚未解决的问题

见指南的 [未知项](module-guide.md#未知项)。
