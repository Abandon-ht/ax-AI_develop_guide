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

[检测模型](./docs/yolov8_native.md)
[分割模型](./docs/yolov8-seg_native.md)
[姿态模型](./docs/yolov8-pose_native.md)

## 2. 准备工具链

<a href="https://pulsar2-docs.readthedocs.io/zh_CN/latest/pulsar2/introduction.html"> 获取工具链 </a>
<a href="https://pan.baidu.com/s/1lVo-HhuF4F4Q79vtMzDXew?pwd=pqo2"> 百度网盘 </a>
<a href="https://drive.google.com/file/d/1-NW7ExBXj5-nTha40iwYshjNJb74Zfer/view?usp=drive_link"> Google Drive </a>

```bash
sudo docker load -i ax_pulsar2_${version}.tar.gz
sudo docker run -it --net host --rm -v $PWD:/data pulsar2:${version}
```

## 3. 转换模型

[检测模型](./docs/yolov8_model.md)
[分割模型](./docs/yolov8-seg_model.md)
[姿态模型](./docs/yolov8-pose_model.md)

## 4. 部署模型
在 models/axmodel/ 目录下找到文件 compiled.axmodel 分别重命名为 （注意每次转换会覆盖之前的模型）

yolov8s.axmodel

yolov8s_seg.axmodel

yolov8s_pose.axmodel

上传至 M4N 开发板即可使用 ax-samples 或 ax-pipeline 加载推理
