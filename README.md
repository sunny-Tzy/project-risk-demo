# 项目进度与风险整理工具

一个本地运行的 Flask Demo：上传包含“项目进度”工作表的 `.xlsx` 文件，按固定的业务规则整理项目当前状态，并在网页上展示项目统计、风险清单、项目级结果、记录级明细和数据质量问题。
本次开发严格按照企业级vibe coding开发必要流程，该demo为企业级vibe coding开发的一次流程实例。

## 已实现功能

- `GET /` 提供单页上传与结果展示页面；`POST /api/analyze` 接收 `multipart/form-data` 的 `file` 字段。
- 仅接受 `.xlsx`，且只读取名为“项目进度”的工作表；文件不落盘，仅在内存中处理。
- 校验必填列、空值、重复记录编号和日期可解析性；单条异常记录不会中断其他记录的分析。
- 以固定基准日期 `2026-09-21` 判定“已完成、逾期、临期、数据不足/无法判断、正常”五种状态；“已完成”优先于日期规则。
- 以归一化后的项目名称聚合重复记录，使用唯一最新记录日期确定项目当前状态；缺失/非法记录日期或最新日期并列时标记为“数据不足/无法判断”。
- 输出五类项目状态统计和按“逾期 → 临期 → 数据不足/无法判断”排序的项目级风险清单。
- 提供 pytest 单元/接口测试，覆盖导入、校验、状态规则、项目聚合、风险列表、服务编排和 Flask 路由。

## 技术栈

- Python 3.11
- Flask、Jinja2
- Pandas、openpyxl
- python-dotenv
- 原生 HTML、CSS、JavaScript
- pytest 与 Flask Test Client

## 目录结构

```text
.
├── app.py                      # 本地运行入口
├── app/
│   ├── config.py                # 环境与输入契约配置
│   ├── domain/                  # 校验、状态规则、项目聚合
│   ├── infrastructure/          # Excel 读取
│   ├── routes/                  # 页面与上传 API
│   ├── services/                # 单次分析编排、风险统计
│   ├── static/                  # 页面 CSS 与 JavaScript
│   └── templates/               # Jinja 页面模板
├── tests/                       # 单元与接口测试
├── .env.example                 # 脱敏环境变量模板
├── requirements.txt             # 运行依赖
└── requirements-dev.txt         # 开发与测试依赖
```

## 本地运行

需要 Python 3.11。

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements-dev.txt
python -m flask --app app run --host 127.0.0.1 --port 5000 --no-reload --no-debugger
```

然后在浏览器打开 `http://127.0.0.1:5000`。

运行测试：

```powershell
python -m pytest -q
```

## 配置

默认使用 `development` 环境；本地演示不需要创建 `.env`。如需显式配置，可复制 `.env.example` 为 `.env` 后设置：

```dotenv
APP_ENV=development
```

支持 `development`、`testing`、`staging`、`production`。`staging` 与 `production` 必须在本机或部署环境以环境变量提供唯一的 `SECRET_KEY`；不要把实际密钥提交到仓库。

上传限制和输入契约集中在 `app/config.py`：最大文件大小为 10 MB，目标工作表为“项目进度”，必填列为“记录编号、项目名称、当前进展、记录日期”。“计划完成日期”允许为空，但会导致未完成记录被标记为“数据不足/无法判断”。

## Excel 输入说明

本整理副本刻意未包含原项目的 Excel 文件，以避免公开未知来源的数据。你可以创建一份脱敏 `.xlsx` 文件，并建立名称严格为“项目进度”的工作表。

| 列名 | 是否必填 | 用途 |
| --- | --- | --- |
| 记录编号 | 是 | 记录唯一标识；重复会报告数据质量问题 |
| 项目名称 | 是 | 项目级聚合键 |
| 当前进展 | 是 | 仅精确匹配“已完成”“完成并归档”“已完成并归档”视为完成 |
| 记录日期 | 是 | 确定同一项目的最新记录 |
| 计划完成日期 | 否 | 风险状态判定日期 |
| 负责人、当前节点、问题情况 | 否 | 页面展示字段 |

测试会在内存中构造虚构工作簿；仓库不需要下载数据集、模型权重或数据库初始化文件。

## 接口与错误处理

- `GET /`：上传与结果页面。
- `POST /api/analyze`：分析文件。成功返回 JSON；未选择文件或非 `.xlsx` 返回 400；可预期的工作表、字段、空表或解析问题返回 422；未预期异常返回不含堆栈信息的 500。

## 当前限制

- 基准日期固定为 `2026-09-21`，尚未提供页面配置。
- 仅支持单次本地分析；不保存上传文件、结果或历史记录。
- 未实现 CSV 导出、登录/权限、数据库、部署、复杂图表和自动化端到端浏览器测试。
- 当前仓库没有截图或可公开演示数据；建议在确认数据脱敏后补充截图与样例工作簿。

## 开源使用说明

本整理副本未附带许可证，当前不应被视为已授予第三方使用、修改或分发权限。公开发布前请由项目权利人选择并加入合适的 `LICENSE`（例如 MIT 或 Apache-2.0），并确认示例数据、截图和项目名称均可公开。

更多筛选依据见 [PROJECT_AUDIT.md](PROJECT_AUDIT.md)，上传前步骤见 [UPLOAD_CHECKLIST.md](UPLOAD_CHECKLIST.md)。
