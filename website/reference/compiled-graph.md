# CompiledGraph

`StateGraph.compile()` 的产物 —— 执行图的句柄。

## 执行

```python
app.invoke(inputs: dict, *, thread_id="default") -> GraphResult         # 同步门面
await app.ainvoke(inputs: dict, *, thread_id="default") -> GraphResult  # 异步内核
```

返回 [`GraphResult`](types.md#会话与结果):

- `.status` ∈ `"done"` / `"interrupted"`;
- `.values`:最终图状态(dict);
- `.usage`:各节点用量之和(复用 `rein.Usage`);
- `.session`:可序列化的 [`GraphSession`](types.md#会话与结果)(存盘 / 恢复用);
- `.interrupt`:中断时的 [`GraphInterrupt`](types.md#会话与结果),否则 `None`。

> 同步 `invoke` 在已有事件循环里会报错并引导你改用 `await ainvoke`(同 rein 的同步门面策略)。

## 流式

```python
async for ev in app.astream(inputs, *, thread_id="default"):
    print(ev.type, getattr(ev, "node", None))
```

[`GraphEvent`](types.md#流式) 的 `type` 有:`superstep_start` / `node_start` / `node_end` / `state_update` / `interrupt` / `done`。

## 恢复(HITL / 错误)

```python
app.resume(session_or_thread_id, *, approve=True, answer=None) -> GraphResult
await app.aresume(session_or_thread_id, *, approve=True, answer=None) -> GraphResult
```

按中断类型分发(复用 rein 的 `aresume`):

| 中断类型 | 恢复语义 |
|---|---|
| `need_approval` | `approve=True` 放行工具 / `False` 拒绝 |
| `need_input` | `answer="..."` 回填问题答案 |
| `error` | `approve=True` **重试该节点** / `False` 放弃(`stop_reason=node_error_abandoned`) |

传入可以是 `GraphResult.session`、磁盘 `store.load(thread_id)` 出的 `GraphSession`,或(挂了 store 时)直接 `thread_id` 字符串。

## 可视化

```python
app.to_mermaid() -> str    # 返回 mermaid 流程图文本(零依赖,可直接贴进 Markdown)
```
