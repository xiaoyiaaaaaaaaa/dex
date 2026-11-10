# 修复交接：Codex Desktop 新聊天与 Responses HTTP 兼容

用户已明确授权：在本项目另开聊天进行代码修改，修复 OpenAI Responses 直连不能直接新建对话的错误。工作区 E:/deecodex-main/deecodex-main；这是无 .git 的源码快照，VERSION=4.4.7。不要初始化 Git、覆盖安装版或重启正在使用的 Dex/Codex。

## 已核实证据

- 本机 config.toml 当前 provider=dex_ai，base_url=http://127.0.0.1:4446/codex-router/v1，wire_api=responses，requires_openai_auth=true。未显式配置 supports_websockets，但实际运行日志为 false。features 仅 js_repl=false、default_mode_request_user_input=true。不要盲改身份验证或插件配置。
- 错误：function_call_output requires call_id on HTTP requests; continuation via previous_response_id is only supported on Responses WebSocket v2。
- 失败会话 response_item 含两条 function_call_output，name=codex_apps_open_page 与 codex_apps_selected_page_part_1_of_1，namespace=agent，有 id、output、internal_chat_message_metadata_passthrough，但没有 call_id，也无对应 function_call。
- 当前运行日志客户端版本 26.1103.60933，CLI 会话元数据 0.167.0-alpha.3.2。用户认为今天更新后出现。对最近90条线程元数据、其中存在文件的前36行抽样：今天8份存在上述条目，9份未发现；11月7—9日同一CLI版本的12份未发现。样本有限，同一CLI版本更早存在，不能断言具体更新提交导致。
- Dex Chat 日志明确 dropped 2 orphan tool message(s) before upstream translation。Responses 是 bypass 路径。
- 一份会话 Chat 19:50:15成功后，Responses 19:50:23/26仍失败；Chat 19:50:40—41出现带call_id的exec_command及结果后，Responses 19:50:53成功。另一会话也先出现正常工具调用再恢复。原始会话仍保存缺call_id条目，不能称为清理了持久历史，也不能证明工具调用是恢复原因。

## 源码定位（需独立复核）

src/translate.rs:237 缺call_id回退空字符串；:387 调用 sanitize_tool_messages；:507—580删除孤立工具消息。
src/handlers.rs:10618—10622 Responses预处理：reasoning、conversation、DeepSeek配对、namespace、arguments，无通用页面上下文处理。
:4407 工具配对仅provider=deepseek，因此不解释OpenAI路径恢复。
:4452 output_by_call为空直接返回，恰好放过仅有缺call_id上下文的新聊天；:4500后续循环才删除缺call_id输出。此为额外缺口，不要为了修复OpenAI把provider改成deepseek。
:10162 namespace修补仅处理function_call；:10186参数修补同样不处理function_call_output。
:9929 previous_response桥接仅requires_computer时执行。

## 实施目标

1. 阅读AGENTS.md并复核以上流程，在Responses转发边界做最小兼容修复。已知桌面上下文不是模型真实工具输出；保留页面内容及不可信数据语义，不抬升为system/developer指令，不随机补call_id，不无差别删除正常工具历史。明确未知无call_id条目的处理。不要把仅无call_id的新聊天处理依赖于已有正常工具输出。
2. 优先写有意义的回归用例：两个真实形状上下文+用户输入、混合正常调用/输出、普通工具结果不变、无页面上下文不变；验证真正发往上游的body与解析结构一致。考虑含previous_response_id时合法历史关联，不套用全量孤儿删除。
3. 先检查Python/Rust/cargo可用性及现有cargo进程；构建、临时文件、缓存置E盘。用Python执行脚本，遵守本地Windows要求。针对性测试成功后按仓库要求执行可行检查；缺依赖/平台限制如实报告。勿未经核实安装大工具链。
4. 不发送真实用户聊天到新上游，不暴露密钥。优先合成输入和本地模拟上游。完成源码修复与测试不等于安装版已修复；明确还需编译部署/真实新会话验证。
5. 中文报告变更文件、测试结果、剩余风险。清理本次无用临时脚本到回收站并报告大小；保留有用测试及交接文件。修改前在E盘备份需改文件（因无Git）。

本交接只要求定向兼容修复，不要求通用代理重构、增设多套开关或改动Codex本体。
