# StateGraph

图的构建器。声明节点和边,`compile()` 产出可执行的 [`CompiledGraph`](compiled-graph.md)。

## 构造

```python
from reingraph import StateGraph
g = StateGraph()
```

## 添加节点

```python
g.add_node(name: str, node)
```

`node` 会按类型智能适配:

| 传入 | 自动包成 | 行为 |
|---|---|---|
| `rein.Agent` | `AgentNode` | 调 `agent.arun(prompt)`,输出写 `{output: ...}` |
| async 函数 `async def f(state_values) -> dict` | `FunctionNode` | 返回的 dict 作为状态更新 |
| `CompiledGraph` | `SubGraphNode` | 把一张图当作一个节点(子图) |

## 添加边

```python
g.add_edge(source: str, target: str)   # 静态边,target 可为 END
g.set_entry_point(name)                # 等价 add_edge(START, name)
g.set_finish_point(name)               # 等价 add_edge(name, END)
```

## 条件边(路由 / 扇出 / 循环)

```python
g.add_conditional_edges(source, routing_fn, path_map=None)
```

- `routing_fn(state_values) -> str | list[str]`:返回下一个(些)节点名,或 `END`;
- 返回单个名字 = 普通路由;返回 `list` = **扇出**到多个节点;返回自己 = **循环**;返回 `END` = 收口;
- `path_map`:可选 `{routing 返回值: 真实节点名}` 映射,让 `routing_fn` 返回语义标签而非节点名。

```python
# 条件路由
g.add_conditional_edges("triage", lambda s: "vip" if s["score"] > 9 else "normal")
# reflexion 循环:不达标回到自己,达标收口
g.add_conditional_edges("improve", lambda s: END if s["score"] >= 3 else "improve")
```

## 编译

```python
app = g.compile(store=None, config=GraphConfig())
```

- `store`:可选 [`GraphStore`](types.md#持久化),挂上后可跨进程存盘 / 恢复;
- `config`:[`GraphConfig`](config.md) 熔断与节点级超时 / 重试;
- **编译期校验**:端点存在、`START` 有出边、可达 `END`、汇合节点 reducer 合法 —— 非法图在 `compile()` 就快速失败,不留到运行期。

> ⚠️ `routing_fn` 不进 checkpoint(按 source 名登记在运行期),所以它应是**纯函数**、只读 `state_values`。
