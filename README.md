# claude-codex-project · 门诊心理测评单机软件

门诊医生用的离线小工具：SDS、SAS、PSQI、BSSI 四个量表，点选或键盘录入，提交后马上出分数和分级，一键清空换下一位。断网可用，不上传任何数据。

> **Codex 开工前先读 [`docs/OPEN_QUESTIONS.md`](docs/OPEN_QUESTIONS.md) 和 [`docs/CODEX_TASKS.md`](docs/CODEX_TASKS.md)。**
> 第一件事（T0）：把 OPEN_QUESTIONS 里每一条逐条问需求方，答复写回该文件。还没确认的条目，对应的功能不要实现。

## 文档

| 文件 | 内容 |
|---|---|
| [`docs/PLANNING.md`](docs/PLANNING.md) | 目标、范围、技术选型（Python + tkinter + PyInstaller）、目录结构、接口、键盘操作、配置开关、里程碑、风险 |
| [`docs/OPEN_QUESTIONS.md`](docs/OPEN_QUESTIONS.md) | 24 条待确认问题，每条有可以直接问的话、选项、建议默认和「答复：」栏 |
| [`docs/CODEX_TASKS.md`](docs/CODEX_TASKS.md) | 任务顺序 T0–T14，每个任务有依赖、输入、输出、完成标准 |
| [`docs/scales/`](docs/scales/) | 四个量表的规格（`sds.md` `sas.md` `psqi.md` `bssi.md`）和唯一数据源 `scales.json` |
| [`docs/ACCEPTANCE.md`](docs/ACCEPTANCE.md) | 验收：自动用例说明 + 手工验收清单 |
| [`docs/acceptance_cases.json`](docs/acceptance_cases.json) | 手算并复核过的验收用例 |

本工具仅供临床辅助评估使用，不能替代专业精神科医生的诊断。

这个仓库是公开的：不要提交任何患者数据或真实病例。
