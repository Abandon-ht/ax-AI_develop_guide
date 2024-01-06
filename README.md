# M4N YOLOv8 检测、分割、姿态模型部署全流程
## 0. 准备环境
```bash
git clone https://github.com/ultralytics/ultralytics.git  # clone
cd ultralytics
pip install -r requirements.txt  # install
pip install -e '.[dev]'  # develop
pip install onnxsim
```
## 1. 修改导出代码并导出模型

[检测模型](yolov8_native.md)
[分割模型](yolov8-seg_native.md)
[姿态模型](yolov8-pose_native.md)

## 2. 准备工具链转换模型

<a href="https://pulsar2-docs.readthedocs.io/zh_CN/latest/pulsar2/introduction.html"> 获取工具链 </a>
<a href="https://pan.baidu.com/s/1lVo-HhuF4F4Q79vtMzDXew?pwd=pqo2"> 百度网盘 </a>
<a href="https://drive.google.com/file/d/1-NW7ExBXj5-nTha40iwYshjNJb74Zfer/view?usp=drive_link"> Google Drive </a>

```bash
sudo docker load -i ax_pulsar2_${version}.tar.gz
sudo docker run -it --net host --rm -v $PWD:/data pulsar2:${version}
```
修改 config/yolov8s_config_b1.json 带注释的内容为自己的配置
```
{
  "model_type": "ONNX",
  "npu_mode": "NPU3", #NPU1为单核NPU，NPU2为双核NPU，NPU3为三核NPU
  "quant": {
    "input_configs": [
      {
        "tensor_name": "images", #onnx模型输入名
        "calibration_dataset": "coco_10.tar", #量化数据集包
        "calibration_size": 4,
        "calibration_mean": [0, 0, 0],
        "calibration_std": [255.0, 255.0, 255.0]
      }
    ],
    "calibration_method": "MinMax",
    "precision_analysis": true,
    "precision_analysis_method":"EndToEnd"
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
      "tensor_name": "407", #onnx模型第一个输出头
      "dst_perm": [0, 1, 3, 2]
    },
    {
      "tensor_name": "output0", #onnx模型第二个输出头
      "dst_perm": [0, 2, 1]
    }
  ],
  "compiler": {
    "check": 0
  }
}
```
## 3. 转换模型
```bash
pulsar2 build --input onnx/yolov8s_sim.onnx --output_dir axmodel/ --config config/yolov8s_config_b1.json
```
## 4. 部署模型
在 axmodel/ 目录下找到文件 compiled.axmodel 重命名为 yolov8s.axmodel 上传至 M4N 开发板即可使用 ax-samples 或 ax-pipeline 加载推理
