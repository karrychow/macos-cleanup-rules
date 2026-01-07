# macos-cleanup-rules

社区维护的 macOS 应用残留文件清理规则集。目标是透明、可复用、可审计，帮助用户安全清理 `~/Library` 及相关目录中的缓存、日志与状态文件。

## 文件结构
- `rules.json`：规则清单（根目录放置；或分模块存于 `rules/apps/...`）
- `schema.json`：JSON Schema（用于本地与 CI 校验 PR 格式）

## 规则模型
- `version`：清单版本号（整数）
- `rules[]`：每个应用一条规则，字段：
  - `id`：应用 Bundle ID（如 `com.google.Chrome`）
  - `name`：应用名称
  - `paths[]`：路径规则列表，字段：
    - `path`：路径字符串，支持 `~`、`${HOME}`、通配 `*`、`?`、`**`
    - `type`：文件类别（`cache`、`logs`、`state`、`support`、`preference`、`temp`、`misc`）
    - `danger_level`：风险等级（`safe`、`caution`、`danger`）
    - `recommendation`：建议（`remove`、`review`、`keep`）
    - `comment`：说明（可选）
    - `os`：默认 `macos`

> 清理器应对 `danger` 项默认不删除，仅提示用户，并优先移动到废纸篓而非永久删除。

## 路径匹配规范
- `~` 或 `${HOME}` 展开为用户主目录
- `*` 匹配任意字符（不含分隔符），`?` 匹配单字符
- `**` 允许递归匹配子目录（建议谨慎使用）
- 路径应尽量具体，避免误伤。如需宽泛匹配必须设置 `danger_level: danger` 并给出 `comment`

## 贡献指南
1. Fork 仓库，修改或新增规则
2. 本地校验格式：
   - 使用 Node（AJV）：
     ```bash
     npm i -g ajv-cli
     ajv validate -s schema.json -d rules.json
     ```
   - 或使用 Python：
     ```bash
     python -m jsonschema -i rules.json schema.json
     ```
3. 提交 PR，描述：
   - 规则来源（官方文档/开发者经验/社区共识）
   - 风险评估与说明
   - 截图或链接（可选）
4. CI 校验：
   - PR 将自动运行 JSON Schema 校验和基本结构检查
   - 未通过校验的 PR 将阻止合并

## 版本与兼容
- 使用语义化版本标签（如 `v1.0.0`）
- 客户端应依赖 `version` 字段判断兼容性，并可锁定次版本（minor）
- 发布时同步更新 `rules.json` 的 `version` 字段
- 历史变更请在 PR 与 Release Notes 中记录

## 许可与免责声明
- 建议采用 CC0 或 MIT，便于广泛复用
- 清理存在风险，请务必审阅规则并在清理前备份或使用废纸篓策略