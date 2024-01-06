修改 [config/yolov8s-pose_config_native.json](../config/yolov8s-pose_config_native.json) 带注释的内容为自己的配置
```json
{
  "model_type": "ONNX",
  "npu_mode": "NPU3",
  "quant": {
    "input_configs": [
      {
        "tensor_name": "images",
        "calibration_dataset": "/data/datasets/coco_10.tar",
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
      "tensor_name": "output0",
      "dst_perm": [0, 2, 3, 1]
    },
    {
      "tensor_name": "383",
      "dst_perm": [0, 2, 3, 1]
    },
    {
      "tensor_name": "398",
      "dst_perm": [0, 2, 3, 1]
    },
    {
      "tensor_name": "339",
      "dst_perm": [0, 2, 3, 1]
    },
    {
      "tensor_name": "346",
      "dst_perm": [0, 2, 3, 1]
    },
    {
      "tensor_name": "353",
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
pulsar2 build --input models/onnx/yolov8s-pose_native_sim.onnx --output_dir models/axmodel/ --config config/yolov8s-pose_config_native.json
```
