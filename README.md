# 工业视觉流程编辑器

这是一个基于 FastAPI、OpenCV 和原生 Web 技术实现的节点式工业视觉流程编辑器。前端、API 通信层、后端工作流调度、OpenCV 算法以及独立脚本导出均已分层。

## 启动

```bash
python -m venv .venv
# Windows: .venv\Scripts\activate
# macOS / Linux: source .venv/bin/activate
pip install -r requirements.txt
python run.py
```

浏览器打开 `http://127.0.0.1:8000`。

## 节点预览交互

每个展开节点底部都有“节点效果”区域：

1. 运行流程后，预览区域使用 `object-fit: contain` 显示完整缩略图，不裁切原图。
2. 点击缩略图，会显示“下载”和“放大”两个按钮。
3. 点击“下载”，浏览器直接下载该节点的 PNG 结果图。
4. 点击“放大”，打开原始分辨率查看器。查看器允许滚动查看大图，不会压缩或裁切源图。
5. 点击“恢复缩略图”、原图背景或按 `Esc`，返回节点缩略图模式。
6. 再次点击节点缩略图，可以隐藏操作按钮。

右侧“运行预览”中的图片也采用完整适配显示，点击后可直接进入原图查看器。

## 节点参数和图片路径

所有可调参数都显示在节点卡片内部。`ReadImage` 节点通过 `image_path` 参数读取运行 FastAPI 后端的计算机上的图片：

```text
相对路径：samples/part.png
绝对路径：D:\images\part.png
```

相对路径以项目根目录为基准。前端不再使用全局图片上传框。

## 颜色转换

“灰度化”和“BGR 转 RGB”统一为 `ColorConvert` 节点，通过 `mode` 参数选择：

```json
{"mode": "GRAY"}
```

或：

```json
{"mode": "RGB"}
```

导入旧工作流时，`Gray` 和 `BGR2RGB` 会自动迁移。

## 导出为独立算法脚本

点击左侧“导出脚本”，当前流程会在后端通过 AST 编译为普通、顺序执行的 OpenCV 代码，并下载为 `vision_algorithm.py`。

生成脚本中不包含节点、连线、工作流 JSON、拓扑排序、算子注册表、FastAPI 或项目模块引用。新环境只需：

```bash
pip install opencv-python numpy
python vision_algorithm.py --input input.png --output result.png
```

也可以作为模块调用：

```python
import cv2
from vision_algorithm import run_algorithm

image = cv2.imread("input.png")
result = run_algorithm(image)
cv2.imwrite("result.png", result)
```

## 目录结构

```text
industrial_vision_refactored/
├── backend/
│   ├── api/                 # FastAPI 路由
│   ├── algorithms/          # OpenCV 算子、元数据与注册表
│   ├── models/              # Pydantic 工作流模型
│   └── services/            # 工作流执行、图片读取、AST 脚本生成
├── frontend/
│   ├── index.html
│   └── assets/
│       ├── css/styles.css
│       └── js/
│           ├── api.js       # 前后端通信
│           └── app.js       # 节点、画布、预览和下载交互
├── tests/
├── requirements.txt
└── run.py
```

## API

- `GET /api/operators`
- `POST /api/run`
- `POST /api/export-script`
- `POST /api/upload`：仅保留旧版兼容
