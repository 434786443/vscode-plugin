# VS Code 插件集

自用的 VS Code / Cursor 插件集合。每个插件一个独立目录，互不依赖，可以单独构建、打包和安装。

```
vscode-plugin/
├── search-everywhere/     双击 Shift 的全局搜索
└── <下一个插件>/           同样的结构
```

## 目录约定

每新增一个插件，在根目录建同名文件夹，内部保持这样的结构：

```
<插件名>/
├── package.json           插件的清单：命令、键位、依赖
├── src/                   TypeScript 源码
├── media/                 面板的前端资源（如有）
├── vendor/                需要随包分发的二进制（如有）
├── docs/                  设计与实施文档
├── .vscodeignore          打包时排除的文件
├── tsconfig.json
└── README.md              该插件的说明
```

根目录不放代码，只放聚合性的 README 和 `.gitignore`。

## 通用安装步骤

下面每个插件章节都会给出对应的 `.vsix` 文件名。通用的安装方式有两种：

**命令行（推荐）**

```bash
# Cursor
/Applications/Cursor.app/Contents/Resources/app/bin/cursor --install-extension <插件目录>/<插件名>-<版本>.vsix

# VS Code
code --install-extension <插件目录>/<插件名>-<版本>.vsix
```

**界面**

扩展面板（`Cmd+Shift+X`）→ 右上角 `...` → 「从 VSIX 安装...」→ 选择 `.vsix` 文件。

安装后**必须重载窗口**（`Cmd+Shift+P` → `Reload Window`），插件才会激活。

## 通用开发步骤

```bash
cd <插件名>
npm install
npm run compile
npm test
```

在 VS Code / Cursor 中按 `F5` 启动扩展开发主机调试（`launch.json` 已配好）。

---

# Search Everywhere

双击 `Shift` 打开的搜索面板，在一个面板里搜索工作区的文件名、文件内容、符号和命令，并把 Maven / Gradle 的传递依赖纳入同一套搜索。

## 安装

```bash
cd search-everywhere
npm install
npm run compile
npx @vscode/vsce package --allow-missing-repository
```

然后按上面的通用步骤安装 `search-everywhere/search-everywhere-0.0.1.vsix`。

打包有两个坑：

- **不要加 `--no-dependencies`**。它会把 `node_modules` 排除出包，`@vscode/ripgrep` 的平台二进制就丢了，文件名和内容搜索会直接失效。
- **包里只有当前平台的 ripgrep 二进制**。打出的包在当前系统可用；要在别的平台分发，请在目标平台重新 `npm install` 后打包。

## 使用

双击 `Shift` 打开面板。

| 操作 | 按键 |
| --- | --- |
| 打开 / 关闭 | 双击 `Shift` / `Esc` |
| 上下选择 | `↑` `↓` |
| 打开并关闭 | `Enter` 或点击 |
| 切换标签 | `Tab` / `Shift+Tab` |

四个标签：全部、文件、符号、命令。全部标签每组最多 8 条，其余标签各 100 条。

## 查询语法

| 输入 | 效果 |
| --- | --- |
| `User` | 模糊匹配，文件名命中优先于路径命中 |
| `OCo` | 驼峰简写，命中 `OrderController` |
| `ui/btn` | 目录含 `ui`、文件名含 `btn` |
| `Button.tsx:12` | 搜索 `Button.tsx`，打开后跳到第 12 行 |
| `.java` | 只列 Java 文件 |
| `*.java,*.xml` | 多个后缀 |
| `Order .java` | 搜 `Order`，限定 Java |
| 空输入 | 列出最近打开的文件 |

后缀过滤在取前 100 条**之前**生效，所以不会被无关文件挤掉名额。多词查询要求每个词都命中，不要求相邻。

## 结果标识

每个结果前有一个彩色类型标识，风格接近 IntelliJ 的文件图标：

| 类型 | 标识 | 类型 | 标识 |
| --- | --- | --- | --- |
| Java | `J` | TypeScript | `TS` |
| Go | `GO` | JavaScript | `JS` |
| Kotlin | `K` | JSON | `{}` |
| Markdown | `M` | CSS | `#` |
| Shell | `>_` | 命令 | `>` |

## 依赖包搜索

Java 项目会自动解析 Maven / Gradle 的传递依赖，包含多模块和 compile / runtime / test 的传递依赖。搜 `@Service` 能命中注解类，搜 `OrderController` 能命中依赖包里的类（结果行标出 JAR 文件名）。

第一次打开 Java 项目时依赖索引在后台建立，状态栏显示「正在更新依赖索引」。构建文件（`pom.xml`、`build.gradle` 等）改动后 1 秒重建。也可手动执行：

`Search Everywhere: 刷新依赖索引`

源码只使用本地已存在的 `*-sources.jar`，不下载缺失的源码包。

只索引依赖的结构信息：类名、路径、方法签名。不反编译方法体，不搜索字节码里的字符串。

## 文件匹配的实现

文件名匹配的排序代码来自 [OpenIDEA](https://github.com/l0ng-ai/openidea) 的 `find_files`，编译成 WebAssembly 调用。桌面版 IDE 与这个插件共用同一份打分逻辑，排序结果逐位一致。

`vendor/wasm/` 里是预编译产物，安装者不需要 Rust 工具链。改动匹配逻辑后重新生成：

```bash
npm run build:wasm -- /path/to/openidea
```

WASM 加载失败或调用出错时，文件名搜索会退回内置的 TypeScript 匹配，其余标签不受影响，输出通道 `Search Everywhere` 里会留下一条记录。

## 设计文档

- `docs/superpowers/specs/2026-11-01-search-everywhere-design.md` — 整体设计
- `docs/superpowers/specs/2026-11-02-file-search-wasm-design.md` — 文件搜索改用 WASM
- `docs/superpowers/plans/2026-11-02-file-search-wasm.md` — 实施计划

## 已知限制

- 不反编译方法体，不索引 `.class` 里的字符串。
- 不下载缺失的 sources jar。
- 不解析 Maven / Gradle 以外的构建工具。
- 打包的 ripgrep 二进制只对应打包所在平台。
