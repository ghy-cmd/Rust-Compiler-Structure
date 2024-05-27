Rust编译器的调用流程涉及多个阶段，每个阶段由特定的模块处理。以下是详细的调用流程描述，涵盖每个模块的角色及其调用顺序。

### 编译流程概述

1. **编译启动**
2. **解析**
3. **宏扩展和属性处理**
4. **高层中间表示（HIR）生成**
5. **类型检查和借用检查**
6. **中间代码生成（MIR生成）**
7. **优化**
8. **代码生成**

### 1. 编译启动

- **rustc_driver**
  - 作为编译器的主入口，接收命令行参数和源文件路径。
  - 调用 **rustc_interface** 初始化编译环境和会话。

### 2. 解析

- **rustc_interface**
  - 调用 **rustc_session** 创建和管理编译会话。
  - 调用 **rustc_parse** 解析源代码。

- **rustc_parse**
  - 调用 **rustc_span** 处理源代码位置和范围信息。
  - 将源代码解析为AST（抽象语法树）。

### 3. 宏扩展和属性处理

- **rustc_expand**
  - 调用 **rustc_feature** 检查和处理语言特性和实验性功能。
  - 扩展宏和属性，将其应用到AST中。
  - 调用 **rustc_span** 更新位置和范围信息。

### 4. 高层中间表示（HIR）生成

- **rustc_hir**
  - 将扩展后的AST转换为HIR（高层中间表示）。

- **rustc_hir_analysis**
  - 对HIR进行分析，包括类型检查和借用检查。

### 5. 类型检查和借用检查

- **rustc_hir_analysis**
  - 调用 **rustc_infer** 进行类型推断。
  - 调用 **rustc_borrowck** 进行借用检查。
  - 调用 **rustc_lint** 运行Lint检查。

### 6. 中间代码生成（MIR生成）

- **rustc_mir_build**
  - 将HIR转换为MIR（中间表示）。

- **rustc_mir_transform**
  - 对MIR进行优化和变换。

### 7. 优化

- **rustc_mir_transform**
  - 调用 **rustc_const_eval** 进行常量求值和优化。
  - 调用 **rustc_monomorphize** 进行泛型实例化和单态化。

### 8. 代码生成

- **rustc_codegen_ssa**
  - 将优化后的MIR转换为SSA形式。

- **rustc_codegen_llvm**
  - 使用LLVM将SSA形式的中间代码生成机器代码。

- **rustc_codegen_gcc**
  - 使用GCC将SSA形式的中间代码生成机器代码。

- **rustc_codegen_cranelift**
  - 使用Cranelift将SSA形式的中间代码生成机器代码。

### 辅助模块的调用

- **rustc_errors**
  - 在整个编译过程中被调用以处理错误和警告。

- **rustc_metadata**
  - 管理编译单元的元数据，在解析、扩展、分析和生成阶段都被调用。

- **rustc_query_system** 和 **rustc_query_impl**
  - 实现查询系统，用于延迟计算和缓存编译结果。

### 详细调用顺序

1. **rustc_driver** 调用 **rustc_interface**。
2. **rustc_interface** 调用 **rustc_session** 创建会话。
3. **rustc_interface** 调用 **rustc_parse** 解析源代码。
   - **rustc_parse** 调用 **rustc_span**。
4. **rustc_interface** 调用 **rustc_expand** 处理宏和属性。
   - **rustc_expand** 调用 **rustc_feature** 和 **rustc_span**。
5. **rustc_interface** 调用 **rustc_hir** 生成HIR。
6. **rustc_interface** 调用 **rustc_hir_analysis** 进行类型检查和借用检查。
   - **rustc_hir_analysis** 调用 **rustc_infer**、**rustc_borrowck**、**rustc_lint**。
7. **rustc_interface** 调用 **rustc_mir_build** 生成MIR。
8. **rustc_interface** 调用 **rustc_mir_transform** 优化MIR。
   - **rustc_mir_transform** 调用 **rustc_const_eval** 和 **rustc_monomorphize**。
9. **rustc_interface** 调用 **rustc_codegen_ssa** 生成SSA形式的中间代码。
10. **rustc_codegen_ssa** 调用 **rustc_codegen_llvm**、**rustc_codegen_gcc** 或 **rustc_codegen_cranelift** 生成最终机器代码。

### 额外的模块

- **rustc_baked_icu_data**：处理编译器内置的ICU数据，在解析和宏扩展阶段可能会被调用。
- **rustc_fluent_macro**：处理fluent格式的国际化消息，在错误处理阶段可能会被调用。
- **rustc_fs_util**：文件系统操作的实用工具，在编译过程中需要文件操作时被调用。
- **rustc_graphviz**：生成Graphviz图形，用于可视化分析结果，主要在开发和调试阶段使用。
- **rustc_log**：记录编译过程中的日志信息，贯穿整个编译过程。
- **rustc_symbol_mangling**：处理符号名称，主要在代码生成阶段使用。
- **rustc_sanitizers**：集成各种sanitizer（地址、线程等），主要在代码生成阶段使用。
- **stable_mir**：提供稳定的中间表示（MIR），主要在MIR生成和优化阶段使用。

通过以上的详细描述，可以看到Rust编译器在每个阶段如何调用不同的模块，以及这些模块之间的相互关系。