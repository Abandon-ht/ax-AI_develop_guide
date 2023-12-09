准备环境
```bash
git clone https://github.com/ultralytics/ultralytics.git  # clone
cd ultralytics
pip install -r requirements.txt  # install
pip install -e '.[dev]'  # develop
pip install onnxsim
```
修改 ultralytics/nn/modules/block.py class DFL(nn.Module):
```python
    def __init__(self, c1=16):
        """Initialize a convolutional layer with a given number of input channels."""
        super().__init__()
        self.conv = nn.Conv2d(c1, 1, 1, bias=False).requires_grad_(False)
        x = torch.arange(c1, dtype=torch.float)
        self.conv.weight.data[:] = nn.Parameter(x.view(1, c1, 1, 1))
        self.c1 = c1

    def forward(self, x):
        """Applies a transformer layer on input tensor 'x' and returns a tensor."""
        b, c, a = x.shape  # batch, channels, anchors
        # return self.conv(x.view(b, 4, self.c1, a).transpose(2, 1).softmax(1)).view(b, 4, a)
        return self.conv(x.view(b, 4, self.c1, a).transpose(2, 1).softmax(1))
        # return self.conv(x.view(b, self.c1, 4, a).softmax(1)).view(b, 4, a)
```
修改 ultralytics/nn/modules/head.py class Detect(nn.Module):
```python
    def forward(self, x):
        """Concatenates and returns predicted bounding boxes and class probabilities."""
        shape = x[0].shape  # BCHW
        for i in range(self.nl):
            x[i] = torch.cat((self.cv2[i](x[i]), self.cv3[i](x[i])), 1)
        if self.training:
            return x
        elif self.dynamic or self.shape != shape:
            self.anchors, self.strides = (x.transpose(0, 1) for x in make_anchors(x, self.stride, 0.5))
            self.shape = shape

        x_cat = torch.cat([xi.view(shape[0], self.no, -1) for xi in x], 2)
        if self.export and self.format in ('saved_model', 'pb', 'tflite', 'edgetpu', 'tfjs'):  # avoid TF FlexSplitV ops
            box = x_cat[:, :self.reg_max * 4]
            cls = x_cat[:, self.reg_max * 4:]
        else:
            box, cls = x_cat.split((self.reg_max * 4, self.nc), 1)
        # dbox = dist2bbox(self.dfl(box), self.anchors.unsqueeze(0), xywh=True, dim=1) * self.strides
        dbox = self.dfl(box)

        if self.export and self.format in ('tflite', 'edgetpu'):
            # Normalize xywh with image size to mitigate quantization error of TFLite integer models as done in YOLOv5:
            # https://github.com/ultralytics/yolov5/blob/0c8de3fca4a702f8ff5c435e67f378d1fce70243/models/tf.py#L307-L309
            # See this PR for details: https://github.com/ultralytics/ultralytics/pull/1695
            img_h = shape[2] * self.stride[0]
            img_w = shape[3] * self.stride[0]
            img_size = torch.tensor([img_w, img_h, img_w, img_h], device=dbox.device).reshape(1, 4, 1)
            dbox /= img_size
        x = dbox
        y = cls.sigmoid()
        # y = torch.cat((dbox, cls.sigmoid()), 1)
        return (y, x) if self.export else (y, x)
        # return y if self.export else (y, x)
```
导出模型
```bash
yolo export model=yolov8s.pt format=onnx
onnxsim yolov8s.onnx yolov8s_sim.onnx 
```
![](./images/000.png)

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
转换模型
```bash
pulsar2 build --input onnx/yolov8s_sim.onnx --output_dir axmodel/ --config config/yolov8s_config_b1.json
```
在 axmodel/ 目录下找到文件 compiled.axmodel 重命名为 yolov8s.axmodel 上传至 M4N 开发板即可使用 ax-samples 或 ax-pipeline 加载推理
