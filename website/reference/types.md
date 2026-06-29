# 类型参考

reinGraph 的数据类型。设计铁律(继承 rein):所有 `*Session` / `*State` / `*Result` **零方法、永远可序列化**。

## 节点 {#节点}

### Node(协议)
统一节点协议:`async ainvoke(state_values) -> NodeResult` + `async aresume(node_session, *, approve, answer) -> NodeResult`。

### AgentNode
包 `rein.Agent`。`ainvoke` 调 `agent.arun(prompt)`;agent 在工具边界中断时,把 `rein.Interrupt` + `rein.Session` 装进 `NodeResult` 冒泡上来。prompt 来自 `input_key` 或 `prompt_fn(state_values)`,输出写 `{output_key: result.output}`(默认键 `output`)。

### FunctionNode
包一个 `async def f(state_values) -> dict` 普通函数,返回的 dict 作为状态更新。

### NodeResult
节点一次执行的结果:`updates`(状态更新 dict)、`interrupt`(`rein.Interrupt | None`)、`usage`、`rein_session` / `sub_session`(中断时存的可恢复快照)、`steps`。

## 状态 {#状态}

### GraphState
图的共享黑板:`values: dict[str, Any]` + `channels: dict[str, Channel]`。

### Channel + reducer
并行汇合时,多个节点写同一个键怎么合并 —— 用 **channel + reducer**。reducer 用**名字**(注册表)而非函数引用,所以 channels 可序列化。内置:`last_write_wins`(默认)、`append`、`add`、`merge_dict`。

```python
from reingraph import register_reducer
register_reducer("keep_max", lambda old, new: max(old or 0, new))
```

### apply_updates / make_state
`apply_updates(state, {key: delta})` 是纯函数,按各键 reducer 合并;`make_state(values, channels)` 造初始状态。

## 会话与结果 {#会话与结果}

### GraphSession
图执行的可序列化快照(对位 `rein.Session` 升一维):`thread_id` / `state` / `frontier` / `completed` / `superstep` / `node_sessions: dict[str, rein.Session]` / `sub_sessions` / `loop_counts` / `usage` / `pending_interrupt` / `done` / `stop_reason`。**命门**:把被中断节点的 `rein.Session` 整个嵌进来 → 图存盘 = `model_dump_json()`、图恢复 = 对中断节点 `agent.aresume`。

### GraphResult
`status`(`"done"` / `"interrupted"`)/ `values` / `session` / `steps` / `usage` / `stop_reason` / `interrupt`。

### GraphInterrupt
图级中断:`node`(哪个节点中断)+ `inner: rein.Interrupt`(原样复用,`type` ∈ `need_approval` / `need_input` / `error`)。

### GraphStep
图级流水账:`superstep` / `node` / `summary` / `inner_steps: list[rein.Step]` / `usage`。

## 持久化 {#持久化}

### GraphStore(协议)+ MemoryGraphStore / FileGraphStore
`save(thread_id, gs)` / `load(thread_id) -> GraphSession | None`,复用 `rein.SessionStore` 的形态。`FileGraphStore(dir)` 落 JSON 文件(防路径穿越),`MemoryGraphStore` 进程内。

```python
from reingraph import FileGraphStore
store = FileGraphStore("./checkpoints")
app = g.compile(store=store)
```

## 流式 {#流式}

### GraphEvent
`astream` 产出的事件:`type`(`superstep_start` / `node_start` / `node_end` / `state_update` / `interrupt` / `done`)+ `node` + 载荷。

## 日志 {#日志}

### enable_logging
```python
from reingraph import enable_logging
enable_logging()   # 默认 INFO 到 stderr,顺带开底层 rein 的日志
```
engine 在熔断 / 中断 / 完成处埋点,每条带 `thread_id` 作 trace,**脱敏不记 state / prompt**。
