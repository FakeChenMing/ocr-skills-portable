# Portable OCR Skills

可迁移的四技能组合。保留既有 OCR 分工、准确性优先/速度其次，以及整机内存不超过 90% 的运行规则。此仓库只含技能、安装工具和迁移说明，不包含令牌、模型、虚拟环境、案件文件或识别结果。不会修改原电脑的现有安装。

## 分工

| 输入/任务 | 首选候选 |
|---|---|
| 纯文字、截图、需要文字坐标 | PP-OCRv6 |
| 标准结构化文档、长而清晰的 PDF、批量材料 | MinerU pipeline |
| 多栏、混合内容、跨页语义 | MinerU hybrid-engine medium |
| 跨页表格、关键复杂文档 | MinerU hybrid-engine high |
| 畸变扫描、照片、印章、生僻字、复杂表格/公式/图表 | PaddleOCR-VL-1.6 |

结构化文档通常由 MinerU 主解析 → 检查空白、乱码、漏识别、阅读顺序和结构 → 仅对异常页/区域用适当 PaddleOCR 能力修复 → 按页码/坐标合并去重 → 回看原图验证。不得默认全量双跑，不得虚构置信度。关键金额、日期、主体、条款和签章须回看原图。

本地和云端没有优先级偏见：先选预期更准确的，再在准确性相近时比较总耗时（上传、排队、分片、启动、推理、下载、合并均计入）。用户当次明确的限制优先。

## 安装技能

需要 Python 3.10+。本仓库以 [ocr-skills-portable.zip](ocr-skills-portable.zip) 分发完整源码包；仓库首页文件用于阅读。下载该文件、核对 SHA256SUMS.txt 后解压，进入解压出的 `ocr-skills-portable` 目录运行：

```sh
python3 install.py --dry-run
python3 install.py
```

默认安装到现有 `CODEX_HOME` 下的 skills，未设置则使用 `~/.codex/skills`。可指定 `--skills-dir`。已有同名技能默认拒绝覆盖；明确使用 `--replace` 时先把原目录移至同级 `.ocr-backups/<唯一编号>/`。安装后重启或重新加载宿主的技能列表。

安装器只安装技能。另一台电脑的 GPT 请先完整阅读 [MIGRATE.md](MIGRATE.md)，再配置本机运行环境并验证真实推理。

## 发给其他人使用

把本仓库链接交给具备本地文件和终端权限的 Codex/GPT，并告诉它：

> 请阅读此仓库的 README.md 和 MIGRATE.md，下载并核对迁移包，在我的电脑上安装四项 OCR 技能；按照实际硬件配置后端，保留准确性优先、速度其次、异常区域修复和整机内存 90% 上限规则。不要只告诉我步骤，请实际安装、测试并报告未完成能力。缺少云端令牌时不要误判本地 OCR 不可用。

普通网页聊天窗口若没有本机操作能力，不能仅凭链接自动安装。此项目是技能编排与迁移包，不是免配置 OCR 网站；依赖、模型和可选云服务仍需配置。使用问题请提交 [Issues](https://github.com/FakeChenMing/ocr-skills-portable/issues)，提供系统、版本、已脱敏的错误信息和合成复现材料，不要上传客户文件或令牌。当前属于预览版本。

## 运行环境

分别配置 MinerU、PaddleOCR 的独立虚拟环境；按官方文档选择适配目标 OS、Python、CPU/GPU 的依赖。不要复制旧电脑的虚拟环境，也不要覆盖 HOME/CODEX_HOME。命令可放入 PATH，定时任务应使用验证过的绝对路径。模型首次运行可能需要下载。

- [MinerU](https://github.com/opendatalab/MinerU)：本地 pipeline/hybrid。
- [PaddleOCR](https://github.com/PaddlePaddle/PaddleOCR)：本地 PP-OCRv6 与 PaddleOCR-VL-1.6，安装对应推理依赖。
- [MinerU API CLI](https://github.com/opendatalab/MinerU-Ecosystem/tree/main/cli)：可用 `npm install -g mineru-open-api`；限额以当时服务为准。

先验证 `mineru --help`、`paddleocr ocr --help`、`paddleocr doc_parser --help` 和实际支持的版本参数。技能中的模型版本是需要保留的目标能力，不代表任意平台都已可运行。若目标设备不支持某本地模型，选择可用远程后端并明确报告，不能仅凭安装成功宣称模型可用。

`MINERU_TOKEN` / `PADDLEOCR_ACCESS_TOKEN` 仅在对应云服务需要时于新机器单独配置，切勿提交仓库。缺少云令牌不代表本地模型不可用。

## 并行与内存

允许多线程/多进程，但预算按**整机物理内存**计算，包含其他应用和全部 OCR 任务。85% 为运行警戒线，90% 为终止/回退线；每秒及启动新 worker 前监测，新增 worker 的预计峰值必须保持低于 85%。到 85% 停止扩容并降低并发；到 90% 取消本任务足够多的 worker 释放内存，保留已完成页并重排未完成页。不能杀其他应用/其他任务的进程；仅暂停进程不能释放内存。

无法可靠监测、单 worker 也放不下或当前已达 90% 时，不启动不受监测的本地 OCR，改可用远程路径或报告限制。跨任务必须共用预算。此包保留的是必须执行的调度规则，不是 OS 级强制内存隔离器；外部应用突增和采样间隔可能造成短暂超限，不承诺绝对无瞬时越界。

## 更新

更新前备份并审查差异。官方后端技能更新不得覆盖自定义路由、内存规则、本地/云端平等原则、无令牌可本地运行和权限错误处理。每天检查的自动化属于宿主设置，不随技能文件自动迁移；只有用户需要时在新机器建立单一每日 08:00 检查任务，避免重复。

## 来源和验证边界

四项技能取自已有工作安装的可迁移快照；不是宣称上游最新版本。PaddleOCR 与 MinerU-Ecosystem 的许可证保留于 licenses；上游地址见 [SOURCES.md](SOURCES.md)。安装器的测试不等于目标电脑完成 OCR 推理验收。
