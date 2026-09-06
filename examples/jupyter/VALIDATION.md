# Jupyter 课程实际验收

验收时间：2026-09-06T09:59:42.568870+00:00。22 篇主线与附录 A1 共 23 个 Notebook，全部在真实 OceanBase 中执行。
第 11 篇额外验证 Bearer 鉴权，当前源码共 **24 个验收配置、196 个代码单元格，全部通过，没有跳过**。
含浏览器场景的额外复验，本次累计执行 26 次、204 个单元格。中间一次报告范围断言失败的记录保留在运行目录中，最终整篇复验已通过。每次使用新 Kernel 和独立数据库；26 个本次新建数据库均已删除，结束后再次查询确认全部不存在。

## 环境与证据

- Python 3.14.6；OceanBase `5.7.25-OceanBase_CE-v4.3.5.4`。
- Generation、Embedding 和 Agent 使用配置的真实模型；没有使用测试模型替代服务调用。
- 20 通过实际 Chromium 操作审核、Skill Library 和报告页面；18、22 运行独立 Receiver 进程；19 使用真实 MCP HTTP 会话。
- 16、22 的 Agent 执行实际文件工具；22 由三个独立 Python/LLM 进程完成交接、修改、验收与 Skill 使用。测试文件的摘要保持不变。
- 基础提交为 `8550f9f939f2903cc22d265bf629be06d04e8dfa`，验收对象包含当前工作区修改。下表与 [完整机器记录](VALIDATION.json) 保留实际执行时的文件摘要。提交文件的摘要单独记录在 `publication` 中：仅增加 Apache 许可证声明并统一 JSON 缩进，23 个 Notebook 的代码单元格摘要保持一致，辅助 Python 与测试样例的 AST 摘要也保持一致。

| 篇目 | 执行单元格 | 耗时（秒） | 结果 | Notebook SHA-256 前 12 位 |
| --- | ---: | ---: | --- | --- |
| [01](01_memory_across_sessions.ipynb) | 8/8 | 20.29 | 通过 | `dff10ce98942` |
| [02](02_memory_lifecycle.ipynb) | 9/9 | 22.85 | 通过 | `2ff58d7860a3` |
| [03](03_scopes.ipynb) | 8/8 | 22.16 | 通过 | `19e6e1db2159` |
| [04](04_prepared_context.ipynb) | 9/9 | 35.92 | 通过 | `44dfe550f824` |
| [05](05_work_handoff.ipynb) | 11/11 | 25.22 | 通过 | `bfc260e8754a` |
| [06](06_reviewed_experience.ipynb) | 9/9 | 22.01 | 通过 | `3386ae8b0182` |
| [07](07_managed_skill.ipynb) | 8/8 | 33.53 | 通过 | `4e704d27bfeb` |
| [08](08_automatic_extraction.ipynb) | 12/12 | 106.17 | 通过 | `f3498858e4c3` |
| [09](09_search_comparison.ipynb) | 9/9 | 57.24 | 通过 | `08b01ecb4d6d` |
| [10](10_agent_memory.ipynb) | 11/11 | 69.67 | 通过 | `a4367dd1128b` |
| [11](11_http_api_lifecycle.ipynb) | 11/11 | 3.19 | 通过 | `56c786c1174f` |
| [12](12_team_knowledge_sharing.ipynb) | 6/6 | 24.11 | 通过 | `aa1806716109` |
| [13](13_team_access_control.ipynb) | 6/6 | 25.22 | 通过 | `cb580dc915d7` |
| [14](14_continuous_source_ingestion.ipynb) | 6/6 | 25.37 | 通过 | `3cfc21e04de7` |
| [15](15_background_learning.ipynb) | 5/5 | 99.49 | 通过 | `dd99dc81ffe9` |
| [16](16_skill_feedback_loop.ipynb) | 7/7 | 249.00 | 通过 | `33a058316c0c` |
| [17](17_external_skill_packages.ipynb) | 7/7 | 61.16 | 通过 | `3cf8fedc4c53` |
| [18](18_skill_distribution.ipynb) | 8/8 | 33.62 | 通过 | `28fe84b81682` |
| [19](19_mcp_agent_tools.ipynb) | 5/5 | 51.94 | 通过 | `15d8c184cf9e` |
| [20](20_team_dashboard_reports.ipynb) | 5/5 | 31.13 | 通过 | `1cea1da21e88` |
| [21](21_observability_recovery.ipynb) | 6/6 | 84.38 | 通过 | `6c5250926c9f` |
| [22](22_complete_team_workflow.ipynb) | 12/12 | 256.37 | 通过 | `de0925ee1bb1` |
| [A1](A1_custom_components.ipynb) | 7/7 | 26.97 | 通过 | `1997721a810f` |
| [11 Bearer](11_http_api_lifecycle.ipynb) | 11/11 | 5.50 | 通过 | `56c786c1174f` |

## 其他检查

- OpenDAL Connector 测试：8 passed，包含复用同一 Connector 时发现外部新增文件的回归测试。
- Ruff 检查与格式检查、`ty check`、`uv lock --check`、`git diff --check` 均通过。
- 23 个源 Notebook 格式有效，未保留运行输出或 execution count。

OpenDAL 的实际文件实验暴露了目录缓存导致新增文件漏扫的问题；Connector 现在禁用该目录缓存。
验收同时检查了真实文件新增、重复扫描、失败窗口和恢复处理。

## 复现与查看

在仓库根目录安装依赖并准备模型和 OceanBase 配置，然后执行：

```bash
uv sync --locked --group notebooks
uv run --locked --group notebooks playwright install chromium
POWERCONTEXT_SERVER_INFERENCE_GENERATION_TIMEOUT_SECONDS=120 \
  uv run --locked --group notebooks python examples/jupyter/verify_oceanbase.py \
  --env-file /absolute/path/to/test.env --create-temporary-databases
```

已有兼容浏览器时可设置 `POWERCONTEXT_NOTEBOOK_BROWSER_EXECUTABLE`；本次验收使用已有 Chromium。
配置方式、逐篇运行和清理说明见 [课程入口](README.md)。

本次本地完整执行记录位于 `.powercontext/oceanbase-complete-20260906/`，第 20 篇的最终可见性与下载验收位于 `.powercontext/oceanbase-browser-final-20260906/`：
`verification.json` 记录每个临时库的创建、执行与删除；每个篇目目录保留执行后的 Notebook、HTML 和 `summary.json`。
`data/` 保留实际项目文件、Agent 输出与浏览器截图，便于检查过程。运行产物被 Git 忽略；机器验收摘要保存在本目录的 `VALIDATION.json`。
临时 HTTP 凭证文件已删除，实验服务和 Receiver 进程已结束；18、22 已撤回本次安装并撤销目标凭证。

Receiver 在同机独立进程中通过 HTTP 验证；Agent 是真实 Python/LLM 工具进程。本次结果不涵盖跨机器部署、Codex/Claude Code 原生客户端或性能基准。
