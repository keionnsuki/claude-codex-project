# 给 Codex 的任务清单

> 先读 `docs/PLANNING.md`（目标、选型、目录、接口、键盘、配置开关），再按下面的顺序做。
> **第一件事是 T0：把 `docs/OPEN_QUESTIONS.md` 里每一条逐条问需求方，把答复写回该文件。还没确认的条目，对应的功能不要实现。**

## 通用规则（每个任务都适用）

1. **先问后做。** 规格没写清、和说明书冲突、或者你觉得用例错了：先查 `docs/OPEN_QUESTIONS.md` 有没有这一条；没有就按同样格式在末尾新加一条（OQ-25 起）去问需求方。不要自己挑一个做法。
2. **不改期望值。** 测试不过就改代码。不要为了让测试通过去改 `docs/scales/scales.json` 或 `docs/acceptance_cases.json`。只有 OPEN_QUESTIONS 的答复要求改时才改，并在提交信息里写上 OQ 编号。
3. **文字只有一份。** 题目、选项、分级文字、免责声明都从 `scales.json` 读，代码里不要再手写一份。按钮、提示这些界面用语可以写在 `ui/` 里。
4. **只用标准库。** 运行时不装任何第三方包；开发用的 `pytest`、`pyinstaller` 写在 `requirements-dev.txt`。
5. **Python 3.8 能跑。** 不用 `match`、`X | Y` 类型写法、运行时的 `list[int]` 注解、`str.removeprefix` 等（见 PLANNING 第4节）。
6. **不联网、不落盘。** 没有任何网络调用；默认不写文件、不写日志；出错时也不能把患者答案写到磁盘或打印到控制台。
7. **不加临床内容。** 程序里不出现说明书和 OPEN_QUESTIONS 答复以外的诊断、建议或解释文字。
8. **一个任务一个提交。** 做完先跑 `pytest`，全绿再提交；提交信息以任务编号开头，例如 `T3: SDS/SAS 计分`。
9. **仓库是公开的。** 不要提交患者数据或甲方的真实病例。

每个任务下面写了「依赖」：列出的 OQ 还没答复时，按该条「未答复时」的规则做；写着“先不做”的就跳过那一部分，在 README 的「未完成」里记一笔。

---

## T0　问清楚待确认问题（写代码之前）

- 依赖：无
- 输入：`docs/OPEN_QUESTIONS.md`（24 条，P0 8 条）
- 要做：
  1. 把每一条的「要问的话」发给需求方，P0 先问。可以一次发一批，但每条单独编号，方便对方逐条回答。
  2. 对方回了，就把答复**原话**和日期写进该条的「答复：」，把「状态」改成「已答复」。回“按建议”也算答复。
  3. 对方说要问甲方的，状态写「已转问甲方」，等回复。
  4. 全部问过以后，在文件末尾加一节「答复汇总」：一张表，每条写最终采用的方案（已答复的按答复，未答复的按「未答复时」）。
  5. 按答复汇总设置 `psyassess/config.py` 的开关（T1 建文件时一起做；以后答复有变化随时改，注释里写 OQ 编号和日期）。
- 输出：更新后的 `docs/OPEN_QUESTIONS.md`
- 完成标准：
  - 24 条都已经问过，每条状态是「已答复」「已转问甲方」或「未答复」之一
  - 「答复汇总」表里每条都有采用的方案，且和该条的答复或「未答复时」一致
  - 在等答复的时候，可以先做不受影响的任务（见下面每个任务的「依赖」）；**OQ-02、03、04 没有全部答复之前，T4 和 T10 不做**

## T1　项目骨架

- 依赖：OQ-22（只影响打包，这里不用等）
- 输入：PLANNING 第5、8节
- 要做：
  - 按 PLANNING 第5节建目录和空模块
  - `python -m psyassess` 打开主窗口：标题“门诊心理测评 v0.1.0”（版本号来自 `psyassess/__init__.py`），最上方一条免责声明，下面暂时只有“主页”两个字
  - `config.py` 写好 PLANNING 第8节的全部开关，值按 T0 的答复汇总
  - Windows 上启动时设置 DPI 感知（失败就忽略），字体优先 Microsoft YaHei
  - `requirements-dev.txt`、一个占位测试、`README.md` 初稿（怎么运行、怎么测试、「未完成」清单）
- 输出：可运行的空壳程序
- 完成标准：
  - `python -m psyassess` 能打开窗口，免责声明和说明书一字不差
  - `pytest` 通过
  - 如果有 Python 3.8：`python3.8 -m compileall -q psyassess` 没有报错；没有的话，人工确认 PLANNING 第4节列出的写法没出现

## T2　量表数据

- 依赖：OQ-10、OQ-11（只影响 `texts.py` 返回什么文字）
- 输入：`docs/scales/scales.json`
- 要做：
  - 复制到 `psyassess/data/scales.json`
  - `texts.py` 读取它。路径要兼容 PyInstaller 打包后的情况（打包后数据在 `sys._MEIPASS` 下），用一个函数统一取路径
  - 提供：免责声明、每个量表的题目、选项、分档、指导语。`SAS_ITEM1_FIX_TYPO` 为真时 SAS 第1题用 `paper_text`；`SHOW_INSTRUCTIONS` 为假时不返回指导语
  - `tests/test_texts.py`：
    - 两份 `scales.json` 内容完全相同
    - SDS 20 题，反向题号正好是 2、5、6、11、12、14、16、17、18、20
    - SAS 20 题，反向题号正好是 5、9、13、17、19
    - PSQI 题号 1–9，第5题有 5a–5j 共 10 个小题
    - BSSI 第1–19题每题 3 个选项，第20、21题是文本题
    - 免责声明等于“本工具仅供临床辅助评估使用，不能替代专业精神科医生的诊断。”
    - 每个量表的分档首尾相接、不重叠，覆盖全部可能分数（SDS/SAS 25–100，PSQI 0–21，BSSI 0–38）
- 输出：`texts.py`、`data/scales.json`、`test_texts.py`
- 完成标准：`pytest` 通过

## T3　SDS、SAS 计分

- 依赖：OQ-01（开关 `ZUNG_ROUNDING`，未答复时四舍五入）、OQ-09（分界以 `scales.json` 为准）
- 输入：`docs/scales/sds.md`、`sas.md` 第5、6节；PLANNING 第6节的接口
- 要做：
  - `scales/sds.py`、`scales/sas.py` 的 `score(answers, rounding)`，返回 `raw`、`standard`、`grade`、`item_scores`
  - 标准分只用整数算：四舍五入 `(raw*5+2)//4`，只取整数部分 `(raw*5)//4`。不能用 `round()`，也不能用浮点数
  - 答案不足 20 个或有非法字母时抛出异常（界面保证不会发生，但计分函数自己也要拦）
  - `tests/test_sds.py`、`test_sas.py`：每道反向题单独测一次；粗分 20–80 全部 61 个值和 `sds.md`、`sas.md` 里的对照表一致（两种取整方式都测）
  - `tests/test_acceptance.py` 先实现 SDS、SAS 两组：按当前 `ZUNG_ROUNDING` 选 `expect` 或 `expect_if_floor_rounding`；另外用参数化把两种取整方式都跑一遍
- 输出：两个计分模块和测试
- 完成标准：`pytest` 通过；`acceptance_cases.json` 的 SDS 13 条、SAS 12 条在两种取整方式下都通过

## T4　PSQI 计分

- 依赖：**OQ-02、OQ-03、OQ-04 都已答复才做**；否则 `PSQI_ENABLED = False`，跳过本任务和 T10。OQ-02 的答复决定 `PSQI_INCLUDE_ITEM6`
- 输入：`docs/scales/psqi.md` 全文；`acceptance_cases.json` 的 `psqi` 和 `psqi_time_parse`
- 要做：
  - `parse_time(text)`：规则见 ACCEPTANCE A.3 最后一张表
  - `score(inp, include_item6)`：先校验（`SAME_TIME` → `SLEEP_ZERO` → `SLEEP_GT_TIB`，抛 `PsqiInputError`，带 `.code`），再算在床时间 `(起床−上床) % 1440`、七个成分、总分、分级、`warn_tib_long`（在床时间超过 `PSQI_TIB_WARN_HOURS` 小时）
  - 成分D 用整数比较：`sleep*100 >= 85*tib` 得 0，`>= 75*tib` 得 1，`>= 65*tib` 得 2，否则 3。显示用的效率按 `(sleep*2000 + tib) // (2*tib) / 10`
  - 5a 只进成分B，不进成分E；5j 的文字不影响分数
  - 如果 OQ-04 的答复和 `psqi.md` 第5节不同：先在 OPEN_QUESTIONS 记下答复，再同步改 `psqi.md` 和相关用例的期望值（提交信息写明 OQ-04）
  - `tests/test_psqi.py` 和 `test_acceptance.py` 的 PSQI 两组：`PSQI_INCLUDE_ITEM6` 为假时，有 `expect_if_no_item6` 的用例用它
- 输出：`scales/psqi.py` 和测试
- 完成标准：`pytest` 通过；PSQI 32 条、时间解析 20 条全部通过

## T5　BSSI 计分和跳题

- 依赖：OQ-05（不影响代码）、OQ-06（开关 `BSSI_ASK_HISTORY_WHEN_SKIPPED`，未答复时为假）
- 输入：`docs/scales/bssi.md` 第2、4、5节
- 要做：
  - `is_skipped(answers)`：第4、5题都选 A 时为真（第4或第5题还没答时为假，界面不允许提交）
  - `visible_items(answers, ask_history_when_skipped)`：返回当前该显示的题号
  - `score(answers, ask_history_when_skipped)`：只把 `visible_items` 里的第1–19题加起来；隐藏题的旧答案忽略；第20、21题永远不计分；返回 `skipped`、`shown`、`total`、`grade`、`screen_positive`
  - `tests/test_bssi.py` 和 `test_acceptance.py` 的 BSSI 组：开关为真时，有 `expect_if_ask_history_when_skipped` 的用例用它
- 输出：`scales/bssi.py` 和测试
- 完成标准：`pytest` 通过；BSSI 11 条全部通过；BSSI-09（改答案后隐藏题不计分）通过

## T6　验收用例全部接上

- 依赖：T3、T4（如果做了）、T5
- 输入：`docs/ACCEPTANCE.md` A 部分
- 要做：
  - `test_acceptance.py` 读 `docs/acceptance_cases.json`（用仓库里的这份，不要复制），五组全部跑；PSQI 未启用时整组 `skip`，原因写“等待 OQ-02/03/04”
  - 失败信息里打印用例编号、说明、期望值和实际值，方便对照
  - 加一个检查：json 里每条用例都被跑到了（数量对得上），防止以后加了用例却没跑
- 输出：完整的 `test_acceptance.py`
- 完成标准：`pytest` 通过；输出里能看到 68 条计分用例 + 20 条时间解析用例（PSQI 未启用时显示为 skipped）

## T7　界面框架：主窗口、主页、会话、清空

- 依赖：OQ-08（`HIDE_RESULTS_UNTIL_DOCTOR`）、OQ-13（`ENABLE_SUMMARY_PAGE`）
- 输入：PLANNING 第6、7节
- 要做：
  - `ui/app.py`：主窗口；最上方常驻免责声明条；本次会话状态（各量表的答案和结果，只放在内存里）；页面切换；全局按键（F5 清空、Esc 返回）
  - `ui/home.py`：四个大按钮，按 1–4 或点击打开；PSQI 未启用时灰色并显示“待确认”；已做完的量表在按钮上标“已完成”
  - 清空：作答中途或有结果时先确认一次；确认后清掉全部答案和结果，回主页
  - 窗口默认最大化；最小 1024×700 也要能用
- 输出：能在主页和空白作答页之间切换的程序
- 完成标准：用 `python -m psyassess` 运行时，ACCEPTANCE B1 除第一条（双击 exe，T13 再验）以外全部打勾；清空后会话状态对象里没有任何答案（加一个不依赖界面的单元测试测会话状态的清空）

## T8　单选题作答页（SDS、SAS）

- 依赖：OQ-11（指导语）、OQ-12（漏答）
- 输入：PLANNING 第6、7节；`sds.md`、`sas.md`
- 要做：
  - `ui/questionnaire.py`：一屏一题；顶部“SDS　第 3/20 题”；四个整行可点的选项；底部题号格子（已答、未答、当前，可点击跳转）和“提交”按钮；开关打开时第一屏前显示指导语
  - 键盘：1–4（含小键盘）选选项并自动跳下一题；↑↓、PageUp/PageDown 切题；最后一题选完焦点到“提交”；Enter 提交，有未答题时跳到第一道未答题并提示
  - 提交后调用 T3 的计分函数，进入结果页（T11 之前先用一个简单页面显示分数和分级）
- 输出：SDS、SAS 可以完整作答并出分
- 完成标准：ACCEPTANCE B2 全部打勾；用键盘照 SDS-01、SDS-04、SAS-02 录入，分数和用例一致

## T9　BSSI 作答页

- 依赖：OQ-06、OQ-19
- 输入：`bssi.md` 第3、4节
- 要做：
  - 复用 `questionnaire.py`，只认 1–3 键
  - 每次第4或第5题的答案变化后，用 `bssi.visible_items()` 重新决定显示哪些题；题号格子和“第 x/y 题”跟着变
  - 第20、21题是文本框（OQ-19），可以空着；文本框获得焦点时数字键是打字
  - 第4、5题都选 A 时，答完第5题就可以提交
- 输出：BSSI 可以完整作答并出分
- 完成标准：ACCEPTANCE B4 全部打勾；照 BSSI-01、BSSI-03、BSSI-08、BSSI-09 操作，结果和用例一致

## T10　PSQI 作答页

- 依赖：**T4 已完成**（OQ-02、03、04 已答复）；OQ-17、OQ-18
- 输入：`psqi.md` 第3、4节
- 要做：
  - `ui/psqi_page.py`：第1、3题“时:分”输入（下拉框 5 分钟一档，也可键入，用 `parse_time` 解析，解析失败就标红提示）；第4题“小时 + 分钟”；第2、6–9题单选；第5题 10 个小题单选，5j 选了 B/C/D 才出现填空框（OQ-18）
  - 输入框获得焦点时数字键是打字；其余时候 1–4 选选项
  - 提交时先做 T4 的校验，三种错误不能提交并提示；在床时间超长时弹窗请核对，确认后照常计算
  - 不预填时间，空着不能提交
- 输出：PSQI 可以完整作答并出分
- 完成标准：ACCEPTANCE B3 全部打勾；照 PSQI-03、PSQI-29、PSQI-E2 操作，结果和用例一致

## T11　结果页和汇总页

- 依赖：OQ-07、OQ-08、OQ-13、OQ-14、OQ-16、OQ-20、OQ-21
- 输入：各量表 md 的“分级和结果显示”一节
- 要做：
  - `ui/result.py`：大字显示“抑郁标准分：XX分”“焦虑标准分：XX分”“PSQI总分：XX分”“BSSI总分：XX分”，下一行分级文字；结果区下方再显示一次免责声明
  - BSSI：6–9 分橙色醒目条；≥10 分红色横幅放在结果页最上方，字号最大；文字只用 `scales.json` 里的原文
  - 只有对应开关打开时才显示：OQ-07 的第4、5题事实提示；OQ-16 的额外信息；OQ-21 的 SDS 第19题事实提示
  - `HIDE_RESULTS_UNTIL_DOCTOR` 为真时：提交后只显示“已完成，请告知医生”，按约定的键才显示结果
  - `ENABLE_SUMMARY_PAGE` 为真时做汇总页：本次已完成的量表结果放在一页
  - `ENABLE_COPY_RESULT` 为真时加“复制结果”按钮：只复制量表名、分数、分级（以及 OQ-16 同意显示的内容），只放剪贴板，不写文件
- 输出：完整的结果展示
- 完成标准：ACCEPTANCE B5 全部打勾；结果页上找不到 `scales.json` 和 OPEN_QUESTIONS 答复以外的临床文字

## T12　隐私和健壮性

- 依赖：OQ-14
- 要做：
  - `tests/test_no_network.py`：扫描 `psyassess/` 下所有 `.py`，出现 `socket`、`urllib`、`http.client`、`http.server`、`requests`、`ftplib`、`smtplib`、`xmlrpc`、`webbrowser`、`subprocess` 的 import 就失败；出现以写模式 `open(`、`logging.FileHandler`、`pickle.dump`、`shelve`、`sqlite3` 也失败（OQ-14 选了保存时，只允许在一个指定模块里写，并把这个模块列进白名单）
  - 全局异常处理：弹一个“程序出错，请重新打开”的提示，不显示也不写出答案内容
  - 清空时把答案对象整体丢掉重建，不是只清界面
- 输出：测试和异常处理
- 完成标准：`pytest` 通过；ACCEPTANCE B6 全部打勾

## T13　打包

- 依赖：OQ-22
- 要做：
  - `packaging/psyassess.spec`：单文件、无控制台窗口、不用 UPX、带上 `psyassess/data/scales.json`、exe 版本信息写上版本号；可选再出一个文件夹版（onedir）
  - `packaging/build_exe.bat`：在 Windows 上建虚拟环境、装 `requirements-dev.txt`、跑 `pytest`、打包，输出到 `dist\`
  - 可选 `.github/workflows/build.yml`：在 `windows-latest` 上跑测试并打包，exe 作为构建产物上传（不要发 Release，不要自动推任何东西）
  - OQ-22 说有 Windows 7 时，README 写明用 Python 3.8 + 兼容 3.8 的 PyInstaller 版本单独打包的步骤
- 输出：`dist\psyassess.exe`
- 完成标准：在一台没装 Python、断网的 Windows 上双击能运行；ACCEPTANCE B7 全部打勾

## T14　说明文档和交付

- 依赖：全部
- 要做：
  - `README.md`：功能、运行方法、测试方法、打包方法、配置开关和对应 OQ、隐私说明（不联网、默认不存）、BSSI 条目和分档的出处（OQ-05 的答复，没有就写“待甲方确认”）、量表授权提醒（OQ-24）、「未完成」清单（所有还没答复、因此没做的功能）
  - 按 ACCEPTANCE B 部分在 Windows 上完整走一遍，把打勾结果和日期、电脑型号写进 `docs/ACCEPTANCE.md`
  - 交给需求方：源代码（仓库）、exe（可选文件夹版 zip）、README
- 输出：可交付版本
- 完成标准：`pytest` 全绿；ACCEPTANCE B 部分全部打勾；README 的「未完成」如实列出所有未做的部分
