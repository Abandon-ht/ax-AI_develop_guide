修改 [config/yolov8s_config_native.json](../config/yolov8s_config_native.json) 带注释的内容为自己的配置
```json
{
  "model_type": "ONNX",
  "npu_mode": "NPU3", #NPU1为单核NPU，NPU2为双核NPU，NPU3为三核NPU
  "quant": {
    "input_configs": [
      {
        "tensor_name": "images", #onnx模型输入名
        "calibration_dataset": "/data/datasets/coco_10.tar",#量化数据集包
        "calibration_size": 4,
        "calibration_mean": [0, 0, 0],
        "calibration_std": [255.0, 255.0, 255.0]
      }
    ],
    "calibration_method": "MinMax",
  },
  "input_processors": [
    {
      "tensor_name": "images",
      "tensor_format": "BGR",
      "src_format": "BGR",
      "src_dtype": "U8",
      "src_layout": "NHWC"
    }
  ],
  "output_processors": [
    {
      "tensor_name": "output0",#onnx模型第一个输出头
      "dst_perm": [0, 2, 3, 1]
    },
    {
      "tensor_name": "344",#onnx模型第二个输出头
      "dst_perm": [0, 2, 3, 1]
    },
    {
      "tensor_name": "359",#onnx模型第三个输出头
      "dst_perm": [0, 2, 3, 1]
    }
  ],
  "compiler": {
    "check": 0
  }
}
```
转换模型
```bash
pulsar2 build --input models/onnx/yolov8s_native_sim.onnx --output_dir models/axmodel/ --config config/yolov8s_config_native.json
```