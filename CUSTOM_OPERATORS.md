# 添加自定义算子

项目已经提供自动发现机制，不需要再手动修改 `registry.py`、`metadata.py` 和前端节点库。

## 最快方式

1. 复制：

```text
backend/custom_operators/_template.py
```

2. 把复制后的文件改成不以下划线开头的名字，例如：

```text
backend/custom_operators/my_defect_score.py
```

3. 修改算子函数、参数和 `OPERATOR_DEFINITION`。
4. 重启 FastAPI 服务。
5. 新算子会自动出现在左侧“自定义算子”分组。

以下划线开头的文件会被加载器忽略，因此 `_template.py` 只作为模板，不会出现在节点库。

## 算子返回格式

最基本的图像算子：

```python
return {
    "out": output_image,
    "out_color_space": "BGR",  # 也可以是 RGB 或 GRAY
}
```

需要输出测量数据时，增加 `metrics`：

```python
return {
    "out": output_image,
    "out_color_space": inputs.get("in_color_space", "BGR"),
    "metrics": {
        "defect_count": 3,
        "max_area": 182.5,
        "scores": [0.82, 0.91, 0.76],
    },
}
```

`metrics` 会自动沿着后续节点传递。批量流程连接到“保存批量结果”后，CSV 会生成类似列：

```text
n5_缺陷检测.defect_count
n5_缺陷检测.max_area
n5_缺陷检测.scores
```

一个算子可以返回多个指标；多个测量算子也可以串联，所有指标都会进入同一行 CSV。

## 参数定义

```python
"params": {
    "threshold": 100,
    "mode": "FAST",
    "model_path": "",
},
"param_schema": {
    "threshold": {
        "label": "阈值",
        "type": "number",
        "help": "0 到 255。",
    },
    "mode": {
        "label": "模式",
        "type": "select",
        "options": [
            {"value": "FAST", "label": "快速"},
            {"value": "ACCURATE", "label": "高精度"},
        ],
    },
    "model_path": {
        "label": "模型路径",
        "type": "text",
        "placeholder": "models/model.onnx",
    },
},
```

前端会根据 `param_schema` 自动生成节点内参数控件。

## 独立算法脚本导出

自定义算子需要导出到 `vision_algorithm.py` 时，提供 `script_compiler`：

```python
def compile_script(input_var, input_color, params):
    threshold = int(params.get("threshold", 100))
    statements = []
    expression = f"cv2.threshold({input_var}, {threshold}, 255, cv2.THRESH_BINARY)[1]"
    output_color = "GRAY"
    return statements, expression, output_color
```

然后在 `OPERATOR_DEFINITION` 中配置：

```python
"script_compiler": compile_script,
```

没有配置时，算子仍可在网页和批量流程中运行，但导出纯算法脚本时会给出明确提示。

## 已提供示例

`backend/custom_operators/mean_brightness.py` 是一个完整示例：

- 自动出现在“自定义算子”分组；
- 不改变图像；
- 输出平均亮度和亮度标准差；
- 批量运行时写入 CSV；
- 支持导出纯算法脚本。
