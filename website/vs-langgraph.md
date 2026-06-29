# reinGraph vs LangGraph

诚实地讲清两者的区别,以及各自适合谁 —— 不吹、不踩。

## 一句话

**LangGraph** 是 LangChain 生态里成熟、主流、被大规模生产验证的图编排框架。
**reinGraph** 是建在 [rein](https://github.com/MaLunan/rein) 之上的新框架,最大特点是 ——【**内核全部复用 rein,而非重造**】。

## 核心区别:复用 vs 重造

LangGraph 必须**自己造** checkpoint / 状态机 / 中断恢复(因为 LangChain 本身没有这些原语);reinGraph 直接**继承** rein 的同一套机制,只新增「图拓扑」一维。

| 能力 | LangGraph(自己造) | reinGraph(复用 rein) |
|---|---|---|
| 执行单元 | 自定义 Runnable / node | `rein.Agent`(直接 `arun`) |
| checkpoint | 自定义 Checkpointer(SQLite/Postgres) | 复用 `rein.SessionStore` |
| 中断 / 恢复 | 自定义 interrupt / `Command` | 复用 `rein.Interrupt` + `aresume` |
| 状态机 | Pregel 超步引擎 | 照抄 rein 的无状态单步推进 |
| 熔断 | `recursion_limit` | rein 四道闸 + 图级四道闸 |
| 离线测试 | 需要 mock 基建 | 复用 `rein.MockProvider` |

> 结果:reinGraph 的整图(含被中断 agent 的会话)一句 `model_dump_json` 存盘、`resume` 一层层分发回 `agent.aresume` —— **图层和单 agent 层是同一套 checkpoint / HITL 内核**,不是两套拼起来的。

## 何时用 reinGraph

- 你已经(或打算)用 **rein** 写单 agent,想把它们编排成图 —— **内核统一,一套心智**;
- 你想要**极薄、源码能一眼看穿**的图引擎(反重抽象);
- 你看重「整图含 agent 会话一句话存盘、换进程恢复」这种**内核级一致性**;
- 你需要的拓扑就是:顺序 / 条件 / 循环 / 并行汇合 / 子图 / HITL —— reinGraph 全有。

## 何时用 LangGraph

- 你在 **LangChain 生态**里,需要大量**现成集成**(向量库 / 工具 / 各厂商封装);
- 你需要**成熟、被大规模生产验证、社区庞大**的框架;
- 你的团队已经熟悉 LangChain / LangGraph 的心智与文档。

## 诚实的话

**LangGraph 在生态、成熟度、社区、被采用规模上远胜 reinGraph** —— 它是经过海量生产验证的主流框架,reinGraph 是刚发布的 Beta。

reinGraph 的价值**不在"取代 LangGraph"**,而在:如果你认同 rein 的极薄 harness 哲学,reinGraph 让你**用同一套内核把单 agent 平滑升维到图**,不必再学一套独立的 checkpoint / 状态机 / 中断恢复。

> **关于"性能 / 代码量"的客观 benchmark**:跨框架跑"硬数字"其实难做公允 —— import 时间被 pydantic 主导、LOC 口径不一、功能集不同,硬塞容易误导。所以这里只给**定性对比**。如果你要量化,建议在**你自己的真实工作流**上各搭一遍、量端到端延迟与可维护性,那才是对你有意义的数字。
