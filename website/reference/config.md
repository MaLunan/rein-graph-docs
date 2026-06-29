# GraphConfig

图执行的熔断与节点级韧性配置。全部有保守默认,生产可调。

```python
from reingraph import GraphConfig

GraphConfig(
    max_supersteps=100,
    max_node_visits=25,
    timeout_s=300,
    max_total_tokens=None,
    node_timeout_s=None,
    node_max_retries=0,
)
```

## 图级四道闸

| 字段 | 默认 | 作用 |
|---|---|---|
| `max_supersteps` | `100` | ① 整图最多推进多少超步(防死循环,对位 rein 的 `max_iterations`) |
| `max_node_visits` | `25` | ② 任一节点最多被进入多少次(循环节点的局部安全网) |
| `timeout_s` | `300` | ③ 整图墙钟超时(秒),`None` 不限 |
| `max_total_tokens` | `None` | ④ 各节点 token 累加上限,`None` 不限 |

任一触顶 → 图安全停,[`GraphResult`](types.md#会话与结果)`.stop_reason` 标明是哪道闸(`max_supersteps` / `max_node_visits` / `timeout` / `max_tokens`)。

## 节点级韧性

| 字段 | 默认 | 作用 |
|---|---|---|
| `node_timeout_s` | `None` | 单个节点执行超时(秒)。超时按节点异常处理 → 重试 / 错误中断 |
| `node_max_retries` | `0` | 节点异常(含超时)的自动重试次数。瞬时错误(网络抖动 / 限流)自愈,耗尽才转错误中断 |

```python
# 瞬时错误自动重试 2 次;单节点超过 30s 算失败
config = GraphConfig(node_max_retries=2, node_timeout_s=30)
app = g.compile(config=config)
```

> 节点失败的**三层防线**:① 自动重试(瞬时)→ ② [错误中断](compiled-graph.md)(持续,人决定重试 / 放弃)→ ③ 熔断(全局失控兜底)。中断(审批)不算异常、不会被重试。
