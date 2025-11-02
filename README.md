# kaggle-2025BYU-Competition
## 竞赛概览：
本次 BYU - Locating Bacterial Flagellar Motors 2025 竞赛要求参赛者自动判断 3D 冷冻电镜断层图（tomograms）中是否存在鞭毛马达，并精确给出其三维坐标。官方评估指标为先基于欧氏距离判断样本的 TP / FN，再在此基础上计算偏重召回的 F2 score，兼顾有无判定与位置误差。该任务关乎微生物运动机理的高通量研究，对加速分子生物学与药物靶点开发具有重要意义。

## 环境要求
Python 3.8+
PyTorch 1.12+
CUDA 11.3+
ultralytics (YOLOv10实现)

## 所用算法
1.数据预处理与切片筛选
仅加载 ∼5 %的关键切片（CONCENTRATION=0.05），显著缩短推理时间。
对每张8-bit JPEG切片使用 2-98% 直方图拉伸归一化，提高SNR，保持尺寸 1024×1024。
2.模型架构与训练策略
采用 MHAF-YOLOv2F（基于 Ultralytics YOLOv10 head 轻量改写），在内部标注集上以单类别“小目标”检测方式微调
同时使用Focal-EIoU loss 处理极端前景/背景不平衡。
3.推理与TTA
对每张切片执行三重 TTA（原图+水平翻转+多尺度）。
将不同 TTA 结果经 Weighted Box Fusion (WBF) 进行置信度加权融合，获得更稳定的 2D 预测框。
4.3D后处理
将 2D 框中心映射为三维点 (z,y,x)，对全部候选点做球形距离聚类 + 3D NMS，仅保留每个 tomogram 最高置信度聚类中心作为最终结果。
若无候选通过置信度阈值，即判定该 tomogram不存在马达并输出 (−1,−1,−1)。


 

