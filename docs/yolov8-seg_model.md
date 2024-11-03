修改 [config/yolov8s-seg_config_native.json](../config/yolov8s-seg_config_native.json) 带注释的内容为自己的配置
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
        "precision_analysis": true
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
            "tensor_name": "output1",
            "dst_perm": [0, 2, 3, 1]
        },
        {
            "tensor_name": "416",
            "dst_perm": [0, 2, 3, 1]
        },
        {
            "tensor_name": "357",
            "dst_perm": [0, 2, 3, 1]
        },
        {
            "tensor_name": "364",
            "dst_perm": [0, 2, 3, 1]
        },
        {
            "tensor_name": "371",
            "dst_perm": [0, 2, 3, 1]
        },
        {
            "tensor_name": "350",
            "dst_perm": [0, 1, 2, 3]
        }
    ],
    "compiler": {
        "check": 0
    }
}
```
转换模型
```bash
pulsar2 build --input models/onnx/yolov8s-seg_native_sim.onnx --output_dir models/axmodel/ --config config/yolov8s-seg_config_native.json
```
