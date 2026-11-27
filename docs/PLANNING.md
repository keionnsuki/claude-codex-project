# 门诊心理测评单机软件 · 规划

> **给 Codex：开工前先读 `docs/OPEN_QUESTIONS.md` 和 `docs/CODEX_TASKS.md`。**
> 第一件事是 CODEX_TASKS 的 T0：把 OPEN_QUESTIONS 里每一条逐条问需求方，答复写回该文件。**还没确认的条目，对应的功能不要实现。**

## 0. 文档导航

| 文件 | 内容 |
|---|---|
| `docs/PLANNING.md`（本文） | 目标、范围、技术选型、目录结构、模块、键盘操作、配置开关、里程碑、风险 |
| `docs/OPEN_QUESTIONS.md` | 24 条待确认问题（8 条 P0），每条写好了可以直接问的话和「答复：」栏 |
| `docs/CODEX_TASKS.md` | 给 Codex 的任务顺序（T0–T14），每个任务有输入、输出、完成标准 |
| `docs/scales/sds.md` `sas.md` `psqi.md` `bssi.md` | 四个量表的规格：题目、选项、正反向、计分、分级、显示文字 |
| `docs/scales/scales.json` | 题目、选项、分档的唯一数据源，程序直接用它，不要手抄 |
| `docs/ACCEPTANCE.md` | 验收：自动用例说明 + 手工验收清单 |
| `docs/acceptance_cases.json` | 68 条手算并复核过的计分验收用例，另有 20 条 PSQI 时间输入解析用例 |

说明书里的“甲方”是写需求说明书的医生；“需求方”是把需求交给我们的人，Codex 的问题都问需求方，由他转问甲方。

## 1. 目标

给门诊医生用的离线小工具：患者在电脑上点选（或医生拿着纸质表用键盘录入）SDS、SAS、PSQI、BSSI 四个量表，提交后马上出分数和分级；做下一位之前一键清空。

成功标准（对应说明书的“交付要求”）：

1. 甲方用已知分数的病例测试，计分、反向计分、跳题全部正确。我们用 `docs/acceptance_cases.json` 的全部用例先测一遍。
2. 一个 `.exe`，在没有网络、没装 Python 的 Windows 上双击就能用，不用安装、不用注册登录。
3. 不联网、不上传任何数据；默认不往硬盘写任何东西。
4. 界面最上方始终显示：“本工具仅供临床辅助评估使用，不能替代专业精神科医生的诊断。”
5. 交付完整源代码。

## 2. 范围

要做：

- 四个量表，题目文字和顺序严格照说明书（PSQI 是否补第6题看 OQ-02）
- 鼠标点选 + 键盘快捷键（数字键 1–4 选选项）
- 提交即出分；“清空”一键回到初始状态
- 结果页：分数、分级、免责声明；BSSI 高分醒目提示
- 计分单元测试；验收用例自动跑
- 打包成 exe；使用说明

不做（除非 OPEN_QUESTIONS 的答复要求）：

- 联网、账号、云同步、自动更新、统计上报
- 数据库、历史记录、导出、打印（OQ-14、OQ-23）
- 患者信息（OQ-15）
- 任何诊断、治疗或处置建议，以及量表分级以外的解释文字

## 3. 临床相关的硬约束

- 结果只显示分数和分级文字（加上 OQ-16 同意的额外信息），以及免责声明。免责声明在每个页面最上方常驻，结果页再显示一次。
- 分级和提示文字只用说明书原文（或 OPEN_QUESTIONS 里甲方确认过的文字）。程序不自己加诊断、处置建议或解释。
- BSSI 6–9 分橙色、≥10 分红色横幅放在结果页最上方（`docs/scales/bssi.md` 第5节）；文字是说明书原文。
- 不实现任何“我们觉得更好”的计分口径。说明书和常见做法不同的地方都已列进 OPEN_QUESTIONS，由甲方定。
- 隐私：不联网；默认不写文件、不写日志（出错时也不能把患者答案写到磁盘）；清空后内存里的答案全部丢掉。

## 4. 技术选型

推荐：**Python 3 + tkinter（标准库自带）+ PyInstaller 打成单个 exe。**

逐条对照需求：

| 需求 | 这个方案怎么满足 |
|---|---|
| 断网运行、不上传 | 运行时只用 Python 标准库，代码里没有任何网络调用；有测试扫描源码，出现 `socket`、`urllib`、`http`、`requests` 等就失败 |
| 不用安装、双击即用 | PyInstaller 把 Python 解释器和 tkinter 一起打进一个 exe，目标电脑不用装 Python，也不用管理员权限 |
| 医生用键盘 1/2/3/4 快速录入 | tkinter 绑定全局按键很直接，主键盘和小键盘的数字都绑上；选完自动跳到下一题，录一张 20 题的表就是按 20 下 |
| 计分必须完全正确 | 计分是和界面无关的纯函数，用 pytest 跑单元测试和全部验收用例；Codex 在 Linux 或 macOS 上也能跑全部计分测试 |
| 交付源代码 | 纯 Python，十几个文件，以后别人也好改 |

为什么不选别的：

- **单个 .html 文件**：说明书允许，但交付要求第2条要的是 .exe；医院电脑的浏览器可能很旧（IE 内核），兼容性没法保证。
- **C# WinForms**：exe 小、启动快、不容易被杀毒误报，也是好选择；但测试和打包都要 Windows/.NET 环境，Codex 在云端不方便跑。需求方如果更熟 C#，可以换，计分规格和验收用例完全不变。
- **Electron / Qt**：安装包上百 MB 或依赖多，对这个规模没必要。

已知代价和对策：

- **exe 只能在 Windows 上打包**（PyInstaller 不能跨平台打包）→ 需求方在自己的 Windows 电脑上运行 `packaging\build_exe.bat`，或用 GitHub Actions 的 Windows 机器自动打包（T13）。
- **单文件 exe 启动要 1–3 秒**（要先解压到临时目录）→ 可以接受；嫌慢就同时交付“文件夹版”（onedir，压缩成 zip）。
- **部分杀毒软件可能误报** PyInstaller 打的 exe → 不用 UPX 压缩；必要时请医院 IT 加白名单；OQ-22 先问清楚。
- **Windows 7**：Python 3.9 起不支持 Win7。代码保持 Python 3.8 能运行：不用 `match`、`X | Y` 类型写法、`list[int]` 这种运行时求值的内置泛型（类型注解用 `typing.List`，或文件开头加 `from __future__ import annotations`）、`str.removeprefix` 等。默认用 Python 3.11 打给 Win10/11；OQ-22 说有 Win7 时，再用 Python 3.8 单独打一份并在真机上试。
- **高分屏字体发虚** → 启动时调用 Windows 的 DPI 感知设置（`ctypes.windll.shcore.SetProcessDpiAwareness(1)`，失败就忽略）。
- **字体** → 优先“Microsoft YaHei”（微软雅黑，Win7 起自带），找不到就退回“SimHei”“SimSun”。

## 5. 目录结构

```
psyassess/
  __init__.py          版本号 __version__
  __main__.py          入口：python -m psyassess
  config.py            所有可配置开关（见第8节），按 OPEN_QUESTIONS 的答复设置
  texts.py             读取 data/scales.json，提供题目、选项、分档文字
  data/scales.json     docs/scales/scales.json 的副本（随程序打包），测试保证两份一致
  scales/
    model.py           通用的小工具函数和类型（选项字母→序号、分档查找、错误类型）
    sds.py             SDS 计分（纯函数）
    sas.py             SAS 计分（纯函数）
    psqi.py            PSQI 时间解析、校验、七成分计分（纯函数）
    bssi.py            BSSI 跳题判断和计分（纯函数）
  ui/
    app.py             主窗口、免责声明条、全局快捷键、本次会话状态、清空
    home.py            主页：选量表
    questionnaire.py   通用单选题页（SDS、SAS、BSSI 共用；BSSI 带跳题）
    psqi_page.py       PSQI 页（时间、时长输入 + 单选）
    result.py          单个量表结果页；汇总页（OQ-13 同意才做）
tests/
  test_sds.py  test_sas.py  test_psqi.py  test_bssi.py
  test_acceptance.py   读 docs/acceptance_cases.json 全部跑一遍
  test_texts.py        程序里的数据和 docs/scales/scales.json 一致；题数、正反向题号核对
  test_no_network.py   扫描 psyassess/ 源码，禁止网络相关模块和写文件
packaging/
  psyassess.spec       PyInstaller 配置（单文件、无控制台窗口、不用 UPX、带上 data/scales.json）
  build_exe.bat        在 Windows 上一键打包
.github/workflows/build.yml   （可选）push 后在 Windows 机器上跑测试并打包 exe
requirements-dev.txt   只有开发用的 pytest、pyinstaller；运行时没有第三方依赖
README.md              怎么运行、怎么测试、怎么打包、配置开关、未完成事项
```

## 6. 模块和接口

计分函数和界面完全分开。界面只负责收集“选了哪个选项”，然后调用下面的函数。函数的返回值和 `docs/acceptance_cases.json` 里 `expect` 的字段一一对应，这样 `test_acceptance.py` 可以直接比较。

```python
# psyassess/scales/sds.py（sas.py 相同，只是题目和分档不同）
def score(answers, rounding="half_up"):
    """answers: 长度 20 的字符串或列表，每个是 "A"/"B"/"C"/"D"，按题号顺序。
    返回 {"raw": int, "standard": int, "grade": str, "item_scores": [int] * 20}。
    标准分用整数算：half_up = (raw*5+2)//4，floor = (raw*5)//4。不要用 round()。"""

# psyassess/scales/psqi.py
def parse_time(text):
    """"2330" "23:30" "930" "9:30" "7" "２３：３０" "23.30" → 一天中的分钟数；不合法返回 None。
    规则和 20 条用例见 acceptance_cases.json 的 psqi_time_parse。"""

def score(inp, include_item6=True):
    """inp 的字段同 acceptance_cases.json 里 psqi[].input：
    q1、q3 为 "HH:MM"；q4_minutes 为整数分钟；q2、q5a…q5j、q6…q9 为 "A"–"D"；q5j_text 为文字。
    合法时返回 {"tib_minutes", "efficiency_pct", "components": {"A"…"G"}, "total", "grade", "warn_tib_long"}；
    不合法时抛出 PsqiInputError，其 .code 为 "SAME_TIME" / "SLEEP_ZERO" / "SLEEP_GT_TIB"（按这个顺序检查）。"""

# psyassess/scales/bssi.py
def is_skipped(answers):
    """第4、5题都选 A 时为 True。answers: {"1": "A", ...}，键是题号字符串。"""

def visible_items(answers, ask_history_when_skipped=False):
    """当前应该显示的题号列表，例如 [1,2,3,4,5] 或 [1..19,20,21]。界面按它显示和隐藏。"""

def score(answers, ask_history_when_skipped=False):
    """返回 {"skipped", "shown", "total", "grade", "screen_positive"}。
    隐藏的题即使 answers 里有旧答案，也不计分。第20、21题永远不计分。"""
```

界面流程：

```
主页 ──按1–4或点击──▶ 作答页 ──提交──▶ 结果页 ──Esc──▶ 主页（再做下一个量表）
  ▲                                                        │
  └──────────────────── 清空（F5，任何页面）─────────────────┘
```

作答页的样子：一屏一题，大字号；顶部是免责声明条和“SDS　第 3/20 题”；中间是题目和四个整行可点的选项；底部是题号格子（已答、未答、当前，可点击跳转）和“提交”按钮。医生照纸质表录入时一路按数字键即可，最后一题选完焦点自动到“提交”。

## 7. 键盘操作

| 按键 | 作用 |
|---|---|
| 主页：`1` `2` `3` `4` | 打开 SDS、SAS、PSQI、BSSI |
| 作答页：`1`–`4`（含小键盘） | 选当前题的 A–D（BSSI 只有 `1`–`3`）；选完自动跳到下一题 |
| `↑` `↓` 或 `PageUp` `PageDown` | 上一题、下一题 |
| `Enter` | 全部答完时提交；还有没答的题时跳到第一道没答的题并提示 |
| `Esc` | 结果页回主页；作答页回主页前先确认（已作答的内容保留到清空为止） |
| `F5` 或点“清空” | 清空本次全部答案和结果，回主页；作答中途或有未看的结果时先确认一次 |

- 键位对应的是**选项**，不是分数。反向题在计分函数里换算，界面上选项永远是 A 在上、D 在下。
- PSQI 的时间和时长输入框获得焦点时，数字键是往框里打字，不是选选项；`Tab` 或 `Enter` 移到下一个框；方向键可以在下拉框里选。
- 文字输入框（PSQI 5j、BSSI 第20、21题）获得焦点时同理。

## 8. 配置开关（`psyassess/config.py`）

所有和 OPEN_QUESTIONS 有关的行为都集中在这里，T0 问完以后按答复设置，并在注释里写上 OQ 编号和答复日期。

| 开关 | 对应 | 建议默认 | 未答复时 |
|---|---|---|---|
| `ZUNG_ROUNDING` | OQ-01 | `"half_up"` | `"half_up"` |
| `PSQI_ENABLED` | OQ-02、03、04 | `True` | `False`（主页 PSQI 灰色显示“待确认”） |
| `PSQI_INCLUDE_ITEM6` | OQ-02 | `True` | 不适用（PSQI 不做） |
| `PSQI_TIB_WARN_HOURS` | OQ-03 | `16` | 不适用 |
| `BSSI_ASK_HISTORY_WHEN_SKIPPED` | OQ-06 | 甲方定 | `False` |
| `BSSI_SHOW_SCREEN_LINE` | OQ-07 | `True` | `False` |
| `HIDE_RESULTS_UNTIL_DOCTOR` | OQ-08 | 甲方定 | `False` |
| `SAS_ITEM1_FIX_TYPO` | OQ-10 | `True` | `False` |
| `SHOW_INSTRUCTIONS` | OQ-11 | `True` | `False` |
| `ENABLE_SUMMARY_PAGE` | OQ-13 | `True` | `False` |
| `ENABLE_COPY_RESULT` | OQ-14 | `True` | `False` |
| `SHOW_EXTRA_DETAILS` | OQ-16 | `{"raw", "psqi_components"}` | 空集合 |
| `SDS_ITEM19_NOTE` | OQ-21 | 甲方定 | `False` |

分级分界不设开关，以 `scales.json` 的 `bands` 为准（OQ-09 如果甲方给了别的分界，同步改 `docs/scales/scales.json` 和 `psyassess/data/scales.json`，并按新答复补验收用例）。漏答必须答完（OQ-12）、5j 选填（OQ-18）、第20题文本框（OQ-19）这几条如果甲方要求改，直接改代码，不设开关。

## 9. 里程碑（每个都能验证）

| 里程碑 | 包含任务 | 怎么验证 |
|---|---|---|
| M0 问清楚 | T0 | `docs/OPEN_QUESTIONS.md` 每条都有答复或标明“未答复”；`config.py` 按答复设好 |
| M1 计分正确 | T1–T6 | `pytest` 全绿；`acceptance_cases.json` 全部用例通过（PSQI 未确认时跳过并在输出里写明原因） |
| M2 界面能用 | T7–T11 | `python -m psyassess` 能用纯键盘做完四个量表；ACCEPTANCE 手工清单第 A、B 部分全部打勾 |
| M3 隐私和健壮 | T12 | `test_no_network.py` 通过；一整轮操作后磁盘上没有新文件；清空后没有残留答案 |
| M4 能交付的 exe | T13 | 在一台没装 Python、断网的 Windows 上双击 exe 能跑；ACCEPTANCE 手工清单全部打勾 |
| M5 交付 | T14 | README 完整；源代码 + exe（+ 文件夹版 zip）交给需求方；“未完成”清单如实列出 |

## 10. 风险

| 风险 | 对策 |
|---|---|
| 计分口径和甲方手上的算法不同，验收对不上 | OPEN_QUESTIONS 先问；所有口径差异都有配置开关或备用期望值 |
| PSQI 说明书自相矛盾，迟迟定不下来 | PSQI 单独开关，其余三个量表先交付 |
| 量表版权和授权（PSQI 属匹兹堡大学，Beck 量表由 Pearson 出版） | OQ-24 提醒甲方自己确认；我们不做判断 |
| 如果在医院正式部署，可能涉及医疗器械软件方面的规定 | 由甲方自行确认；软件只做计分辅助，不做诊断 |
| 杀毒软件误报、医院电脑禁止运行外来 exe | 不用 UPX；OQ-22 先问；准备文件夹版 |
| Windows 7 | 代码保持 Python 3.8 兼容；需要时单独打包并真机验证 |
| **这个 GitHub 仓库是公开的** | 任何人都能看到仓库内容。不要提交患者数据，也不要提交甲方用来验收的真实病例；那些只在本地用 |
