# API 参考

reinGraph 的公开 API —— 所有符号都从顶层 `reingraph` 导出。

## 速查

| 类别 | 符号 |
|---|---|
| 构建 | [`StateGraph`](state-graph.md) · `START` · `END` |
| 执行 | [`CompiledGraph`](compiled-graph.md)(由 `StateGraph.compile()` 产出) |
| 熔断配置 | [`GraphConfig`](config.md) |
| 节点 | [`AgentNode` · `FunctionNode` · `Node` · `NodeResult`](types.md#节点) |
| 状态 | [`GraphState` · `Channel` · `register_reducer` · `apply_updates` · `make_state`](types.md#状态) |
| 会话 / 结果 | [`GraphSession` · `GraphResult` · `GraphInterrupt` · `GraphStep`](types.md#会话与结果) |
| 持久化 | [`GraphStore` · `MemoryGraphStore` · `FileGraphStore`](types.md#持久化) |
| 流式 / 日志 | [`GraphEvent`](types.md#流式) · [`enable_logging`](types.md#日志) |

## 最小例子

```python
from reingraph import StateGraph
from rein import Agent

g = StateGraph()
g.add_node("write", Agent(model="anthropic/claude-opus-4-8"))
g.set_entry_point("write")
g.set_finish_point("write")

app = g.compile()
print(app.invoke({"input": "写一首诗"}).values["output"])
```

> reinGraph 的内核(checkpoint / HITL / 状态机 / 熔断)全部复用 [rein](https://github.com/MaLunan/rein),自己只新增「图拓扑」一维。各类型与 rein 的对应关系见 [设计哲学](../design.md)。
