# ONLYOFFICE core 仓库分析

## 1. 项目做什么
- `core` 是 ONLYOFFICE Document Server 与 Desktop Editors 的核心组件，主要承担多格式文档转换能力。
- 转换覆盖文档、表格、演示、跨平台文档等（如 DOCX/XLSX/PPTX/PDF/HTML/EPUB/XPS 等）。

## 2. 整体架构和技术栈

### 2.1 架构分层
1. **入口层（x2t）**
   - `X2tConverter/src/main.cpp` 负责参数解析、任务分发、转换执行。
2. **调度层**
   - `X2tConverter/src/cextracttools.*` 负责根据文件类型/扩展名判定转换方向。
3. **能力层（格式库）**
   - 通过 `X2tConverter/build/Qt/X2tConverter.pri` 链接多格式库（Doc/Ppt/Odf/Pdf/Html 等）。

### 2.2 技术栈
- **语言**：C/C++。
- **构建**：Qt qmake（`.pro/.pri`），并有部分 CMake/Visual Studio 工程。
- **跨平台**：Windows / Linux / macOS / iOS / Android。
- **第三方**：boost、icu、freetype、openssl、cryptopp、v8 等。

## 3. 核心入口文件和主要模块
- **CLI 入口**：`X2tConverter/src/main.cpp`
- **动态库导出入口**：`X2tConverter/src/dylib/x2t.h/.cpp`
- **核心调度**：`X2tConverter/src/cextracttools.h/.cpp`
- **格式转换实现集合**：`X2tConverter/src/ASCConverters.cpp` + `X2tConverter/src/lib/*.h`

## 4. 启动开发、改动调试、编译

### 4.1 建议开发路径
- 优先以 `X2tConverter/build/Qt/X2tConverter.pro` 为入口在 Qt Creator 打开。
- Debug 时重点断点：
  - `main.cpp` 参数分支
  - `cextracttools.cpp` 转换方向判定

### 4.2 编译输出
- 默认输出 `x2t` 可执行程序。
- `build_x2t_as_library` 模式可构建为动态库并导出 `X2T_Convert`。

## 5. 对外服务能力、调用与验证

### 5.1 服务能力
- 文档格式转换（核心）。
- 文件类型识别（`GetOfficeFileFormat`）。
- 部分网络下载/上传与 WebSocket（用于资源下载、插件相关场景等）。

### 5.2 调用方式
- 命令行：
  - `x2t "path_to_params_xml"`
  - `x2t "input" "output" [font_dir]`

### 5.3 验证方式
- 使用 `Test/Applications/x2tTester` 批量回归（输入目录、输出目录、x2t 路径、并发核数等）。

## 6. 你追问的补充结论
- 不仅仅是转换：仓库包含网络层（FileTransporter/WebSocket）和插件下载等逻辑。
- 未发现“license key -> 并发限制”的显式业务代码；目前看到的是资源/并发层面的技术限制：
  - `x2t` 内存上限环境变量。
  - 下载管理器默认并发上限（5）。
  - 表格行/单元格上限错误码（更像格式兼容限制）。
