# Search Everywhere

双击 `Shift` 打开的搜索面板：文件名、文件内容、符号、命令，以及 Maven / Gradle 传递依赖，都在同一个面板里。

完整的安装说明、查询语法和打包注意事项见仓库根目录的 README。

## 安装

```bash
npm install
npm run compile
npx @vscode/vsce package --allow-missing-repository

# 然后安装生成的 vsix（macOS 上的 Cursor）
/Applications/Cursor.app/Contents/Resources/app/bin/cursor \
  --install-extension search-everywhere-0.0.1.vsix
```

安装后重载窗口（`Cmd+Shift+P` → `Reload Window`），双击 `Shift` 打开面板。

打包时不要加 `--no-dependencies`：它会把 `node_modules` 排除，`@vscode/ripgrep` 的平台二进制随之丢失，文件名和内容搜索会失效。

## 查询

| 输入 | 效果 |
| --- | --- |
| `User` | 模糊匹配，文件名命中优先 |
| `OCo` | 驼峰简写，命中 `OrderController` |
| `ui/btn` | 目录含 `ui`、文件名含 `btn` |
| `Button.tsx:12` | 打开并跳到第 12 行 |
| `.java` / `*.java,*.xml` | 按后缀过滤 |
| 空输入 | 最近打开的文件 |

## 开发

```bash
npm install
npm run compile
npm test
```

在 VS Code / Cursor 中按 `F5` 启动扩展开发主机。

文件匹配的排序来自 OpenIDEA 的 `find_files`，编译成 WebAssembly。`vendor/wasm/` 里是预编译产物，改动匹配逻辑后重新生成：

```bash
npm run build:wasm -- /path/to/openidea
```

## 目录

| 路径 | 内容 |
| --- | --- |
| `src/extension.ts` | 激活、面板、索引、打开结果 |
| `src/query/` | 查询解析：后缀词、`:行:列` |
| `src/rank/` | 匹配档、排序、高亮区间 |
| `src/search/` | 协调器、WASM 加载器 |
| `src/workspace/` | 文件名单、最近文件、ripgrep 封装 |
| `src/symbols/`、`src/commands/` | 符号与命令匹配 |
| `src/jar/`、`src/classpath/` | 依赖索引与构建工具解析 |
| `src/panel/` | 面板 HTML 与文件类型标识 |
| `media/` | 面板的前端资源 |
| `vendor/wasm/` | 预编译的文件匹配 WASM |
| `docs/` | 设计与实施文档 |
