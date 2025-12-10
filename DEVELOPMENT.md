# Material Combiner Addon - 开发文档

## 1. 项目概述
Material Combiner 是一个 Blender 插件，旨在将多个材质的纹理合并为一个纹理图集（Atlas）。这对于游戏开发非常有用，因为它可以显著减少 Draw Calls，从而优化性能。该插件支持多材质混合、纹理优化，并能自动处理 UV 坐标。

## 2. 文件结构与作用

```text
material-combiner-addon-master/
├── __init__.py                 # 插件入口，包含插件元数据 (bl_info)
├── registration.py             # 负责注册和注销所有 Blender 类
├── globs.py                    # 全局变量和配置
├── type_annotations.py         # 类型提示定义
├── operators/                  # 包含插件的所有操作逻辑
│   ├── combiner/               # 核心合并逻辑
│   │   ├── combiner.py         # 主操作符 (Combiner Operator)，协调整个合并流程
│   │   └── combiner_ops.py     # 具体的实现函数 (UV处理, 图像生成, 材质分配等)
│   ├── ui/                     # UI 相关的操作符
│   ├── browser.py              # 打开浏览器的操作符
│   └── get_pillow.py           # 自动安装 Pillow 库的脚本
├── ui/                         # 用户界面定义
│   ├── main_panel.py           # 主面板 (N-panel)
│   ├── selection_menu.py       # 选择菜单
│   └── credits_panel.py        # 致谢面板
├── utils/                      # 工具函数库
│   ├── packers/                # 纹理打包算法 (Bin Packing)
│   │   ├── max_rects_bin_packer.py
│   │   └── ...
│   ├── images.py               # 图像处理辅助函数
│   ├── materials.py            # 材质处理辅助函数
│   ├── objects.py              # 对象/网格处理辅助函数
│   └── textures.py             # 纹理处理辅助函数
└── icons/                      # UI 图标资源
```

## 3. 主要功能模块详解

### 3.1 初始化与依赖管理
*   **位置**: `__init__.py`, `registration.py`, `operators/get_pillow.py`
*   **功能**:
    *   `__init__.py` 是插件加载的入口。
    *   `registration.py` 集中管理所有类（Panel, Operator）的注册。
    *   插件依赖 Python 的 `Pillow` 库进行图像处理。如果环境中未安装，`main_panel.py` 会检测并提示用户安装，由 `get_pillow.py` 执行安装。

### 3.2 用户界面 (UI)
*   **位置**: `ui/main_panel.py`
*   **功能**:
    *   在 3D 视图的侧边栏 (N-panel) 创建 "MatCombiner" 标签。
    *   显示待合并的材质列表 (`SMC_UL_Combine_List`)。
    *   提供图集属性设置（尺寸、填充算法、间距等）。
    *   "Create Atlas" 按钮触发核心合并逻辑。

### 3.3 核心合并逻辑 (Combiner)
*   **位置**: `operators/combiner/combiner.py`
*   **功能**:
    *   `Combiner` 类是核心 Operator (`smc.combiner`)。
    *   **Invoke 阶段**:
        1.  验证选中的对象和材质 (`validate_ob_data`)。
        2.  收集材质和 UV 数据 (`get_data`, `get_mats_uv`)。
        3.  弹出文件浏览器让用户选择保存路径。
    *   **Execute 阶段**:
        1.  **打包 (Packing)**: 调用 `utils/packers` 计算每个纹理在图集中的位置。
        2.  **生成图集**: `combiner_ops.get_atlas` 使用 Pillow 拼接纹理。
        3.  **UV 对齐**: `combiner_ops.align_uvs` 调整网格的 UV 坐标以匹配新的图集位置。
        4.  **材质重组**: 创建包含新图集的新材质，并分配给对象，清除旧材质。

### 3.4 纹理打包 (Packing)
*   **位置**: `utils/packers/`
*   **功能**:
    *   实现二维装箱算法 (2D Bin Packing)。
    *   支持多种算法 (MaxRects, BinaryTree 等) 以最优化地排列不同大小的纹理，减少空间浪费。

## 4. 关键流程图解

1.  **用户操作**: 选择物体 -> 点击 "Update Material List" -> 配置参数 -> 点击 "Create Atlas"。
2.  **数据准备**: 插件遍历选中物体的多边形，提取材质和 UV。
3.  **计算布局**: 算法计算所有纹理在最终大图中的坐标 (x, y, width, height)。
4.  **图像处理**: Pillow 将各个小纹理 Resize 并粘贴到大图的指定坐标。
5.  **网格更新**: 遍历物体的 UV 坐标，根据其对应纹理在新图集中的位置进行缩放和偏移。
6.  **清理**: 移除旧材质，应用新材质。

## 5. 开发注意事项
*   **Pillow 依赖**: 修改图像处理逻辑时，需确保兼容 Pillow 的 API。
*   **Blender 版本兼容性**: 代码中包含对旧版 Blender (2.7x) 和新版 (2.8+) 的兼容处理 (`globs.is_blender_legacy`)，开发时需注意保持兼容。
*   **性能**: 处理大量高分辨率纹理时，Pillow 操作可能耗时，需注意优化。
