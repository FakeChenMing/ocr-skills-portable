# 给另一台电脑 GPT 的安装任务

请把本仓库安装为本机 OCR 技能组合，先读取 README.md 和四项 SKILL.md。不要只复述步骤，应实际完成可执行的部分并报告缺口。

1. 检查 OS、架构、可用内存、Python、GPU、已有 OCR 环境、宿主技能目录。不得覆盖已有凭据、全局 HOME 或 CODEX_HOME。
2. 运行 install.py --dry-run；有同名技能先比较，明确替换时用 --replace 留备份。只安装本仓库四项技能，不复制本机私有数据。宿主不使用 Codex 默认目录时传 --skills-dir。
3. 先按原件 SHA-256 复用同目录 `OCR文本/<原件主名>.md`。逐页直接提取完整可靠的 PDF 文字层。其余页默认使用夸克 general-ocr，住宿发票用 vat-invoice-ocr，铁路电子客票用 general-ocr。本地 OCR 仅复核异常页。手写、签章和关键数字须对照原图，逐页记录原件指纹、实际引擎及复核状态。原件变化后更新，复核记录只适用于对应指纹和页码。 在运行入口的 Python 环境安装 `PyMuPDF==1.26.4` 和 `Pillow`。仅在允许将待识别页发送至夸克时配置 AI Agent 密钥，可设置环境变量 `SCAN_WEBSERVICE_KEY`，或在 `QUARK_OCR_DIR` 指定的私有目录中保存为 `agent-key`。日常和后台使用同一个缓存目录和账户。 日常入口为已安装技能目录下的 `mineru-document-parsing/scripts/quark_ocr.py`。用 `python3 quark_ocr.py 原文件` 调用，可指定 `--kind hotel` 或 `--kind rail`。核过原图后可用 `--review-record` 传入含 source_sha256、page、reviewed=true 和 findings 的 JSON。
4. 参照官方当前文档安装适配本机的独立 MinerU/PaddleOCR 运行环境，保留 PP-OCRv6、MinerU pipeline/hybrid、PaddleOCR-VL-1.6 目标能力。先检查版本参数是否支持，再下载模型。不要复制旧虚拟环境，不要不加判断照搬 CPU/GPU 参数。云服务需要的令牌由用户在安全界面配置，不回显、不提交仓库。
5. 本地并行必须建立整机内存监测和共享调度预算：每秒检查，启动前估算峰值，85% 停止新增并发/降并发，90% 取消本任务 worker、保存完成页、重排剩余页。只暂停不能释放内存。不能监测就不启动本地 OCR。不要把“写了规则”等同于“实现了强制限额”。
6. 用不含真实客户信息的合成测试页进行真实推理：中文、日期、金额、表格，以及至少一个旋转/复杂页面；验证实际文字、结构、页码、输出文件与进程退出码。记录实际 engine/model/device、耗时、峰值内存、修复页和未实现能力。帮助命令成功不等于真实推理通过。
7. MinerU 在受限环境遇到 ProcessPoolExecutor/SC_SEM_NSEMS_MAX 权限错误，应保持参数申请正常权限重试，不误报模型或源文件损坏；权限不允许则按准确性/速度选可用替代。PaddleOCR sysctl 警告需结合退出码及完整结果判断。
8. 验证技能格式，重新加载技能，展示安装目录、后端可用性和未完成事项。自动化不是技能文件的一部分；仅用户需要时建立每天 08:00 的单一版本检查任务，更新不得覆盖路由及内存规则。

验收必须区分：技能安装成功、命令可用、模型可加载、真实推理通过、云端认证可用、内存监测有效。不得合并为笼统“全部正常”。
