# DEX AI 4.4.8 本地构建与验证

日期：2026-11-10。Windows x64 手动安装包已生成，Tauri 构建退出码 0。

安装包：DEX AI_4.4.8_x64-setup.exe
大小：305,442,277 字节（291.29 MiB）
SHA-256：1a469170254f118e0fafd3d8d1edf890ca8cdceb4b43bb5de09680b3f7fb5937
程序 Windows 文件版本：4.4.8.0。

## 验证结果

- 定向回归：4个独立单元测试（lib/bin各运行一遍）和2个集成测试全部通过。集成测试覆盖 HTTP 与 SSE、两条入口路由、页面上下文保留、正常工具历史及 previous_response_id。
- 修复测试夹具：同步 test_state 的全局 upstream 为本地模拟地址，避免 /v1/responses 请求默认 example.com；局部使用 no_proxy 保证测试隔离。不改变生产路由行为。
- cargo check 通过；本次修改的 handlers.rs 和 integration.rs 经 rustfmt --check 通过。
- 完整测试：lib 881通过/5失败，bin 878通过/5失败，兼容测试5通过，集成142通过/1失败。
- 对照修改前两份源码：相同6个不同用例失败，未新增失败。详细名称见 validation/baseline-comparison.json。
- cargo clippy -- -D warnings 未通过，存在原有平台相关 unused imports/dead code 等问题。cargo fmt --check 全仓未通过，未批量修改无关代码。
- Windows MSVC release GUI 编译及 NSIS 打包成功。未运行 cargo xwin；本次直接使用原生 Windows MSVC 工具链，不是交叉编译。
- 尚未执行真实安装、GUI视觉验收或使用真实上游账号新建聊天；不能将合成上游测试等同于线上验收。

## 安装说明

此为本地构建供手动安装，未使用官方自动更新签名。发布配置中的 createUpdaterArtifacts 仅在本次临时打包配置中关闭，没有修改官方更新公钥。

建议先备份现有 Dex 配置及安装目录，在准备更新时退出旧 Dex，再运行本安装包并选择 E 盘安装位置。安装会中断正在使用旧代理的任务，请自行选择空闲时机。本次未替换安装版，也未重启安装版进程。

安装后选择 OpenAI Responses 直连，新建空白聊天，验证首轮回复、一次只读工具调用和后续回复；如果异常，保留错误时间并回退旧版本。未知类型的无 call_id 工具结果仍交由上游验证，本补丁只处理已识别页面上下文。

## 构建来源及限制

源目录 VERSION=4.4.8，无Git元数据。使用 E:/DevTools/rust 中复制的 Rust/Cargo 1.99.1、已安装 MSVC、Tauri CLI 2.12.4。保留Cargo.lock进行Rust依赖解析。OpenCode Windows资源从已安装Dex复制，复制前后SHA-256一致；没有重新从未知源获取。

可选 Mobile Codex/Desktop Sync Windows插件二进制未提供，按项目既有构建规则跳过。Windows打包资源按 tauri.windows.conf.json 选取。安装包包含 WebView2 离线运行库。

## 清理

已将 .validation-4.4.8（299,701,303字节）与 target-local（6,037,428,488字节）移入回收站，总计约5.90 GiB；回收站未清空，不能视为磁盘空间已最终释放。测试日志、补丁、校验值保留在本目录，修改前备份仍保留在项目.backups。

E盘Rust工具链、Cargo依赖缓存、Tauri CLI以及打包资源保留用于后续维护，不属于已确认无用的临时脚本。没有生成额外.py脚本。
