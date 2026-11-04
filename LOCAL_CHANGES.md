# 我的 PointNet / PointNet++ 实验改动

本仓库基于 yanx27/Pointnet_Pointnet2_pytorch，保留原作者 README、MIT 许可证和提交历史。上游 README 中的成绩属于上游实现。我的实验入口是 ModelNet40 分类，分割代码未纳入本次验证。

## 模型相关

- `models/pointnet_cls.py`：关闭 feature transform，保留输入坐标变换。损失仅在 `trans_feat is not None` 时加特征变换正则项。
- `models/pointnet2_cls_msg.py`：第一层 MSG 半径由 `[0.1, 0.2, 0.4]` 改为 `[0.2, 0.4, 0.8]`；邻域上限仍为 `[16, 32, 128]`。
- `models/pointnet2_utils.py`：增加可选的首次前向调试输出，观察分组形状和首个中心的去重邻居数。分类、部件分割、语义分割的 MSG 第一层启用此开关。

这些是探索性改动，尚无受控实验支持精度提升结论。

## 运行与复现

在 Python 3.11 / PyTorch 2.5.1 环境完成了分类 CPU 冒烟验证。安装依赖后，在本仓库根目录执行：

```powershell
python -m pip install -r requirements.txt
$env:MODELNET40_ROOT = "D:\datasets\modelnet40_normal_resampled"
python train_classification.py --model pointnet_cls --log_dir pointnet_experiment --workers 0
python test_classification.py --log_dir pointnet_experiment --num_votes 1 --workers 0
```

`--data_root` 覆盖环境变量；两者都省略时使用本仓库下的 `data/modelnet40_normal_resampled`。目录需包含类别名称、train/test split 和各类别的逗号分隔点云文件。CPU 运行追加 `--use_cpu`；PointNet++ MSG 使用 `--model pointnet2_cls_msg` 和新的实验目录名。

我也修复了 CPU 投票张量、检查点设备映射、数据检查入口缺少参数，以及特征正则项的 Tensor 布尔判断。DataLoader 默认 0 个工作进程，可通过 `--workers` 调整；调度器在每轮优化后更新。

原始日志和检查点不是当前代码的重新评估结果。`.gitignore` 阻止新增训练产物进入提交，但不会自动删除已经跟踪的产物或改写历史。完整项目与改动记录见 [PointNet_Explore](https://github.com/ATree-PointCloud/PointNet_Explore)。
