# Video-Swin-Transformer-Pytorch
This repo is a simple usage of the official implementation ["Video Swin Transformer"](https://github.com/SwinTransformer/Video-Swin-Transformer).

![teaser](https://github.com/SwinTransformer/Video-Swin-Transformer/blob/master/figures/teaser.png)

## Introduction

**Video Swin Transformer** is initially described in ["Video Swin Transformer"](https://arxiv.org/abs/2106.13230), which advocates an inductive bias of locality in video Transformers, leading to a better speed-accuracy trade-off compared to previous approaches which compute self-attention globally even with spatial-temporal factorization. The locality of the proposed video architecture is realized by adapting the Swin Transformer designed for the image domain, while continuing to leverage the power of pre-trained image models. Our approach achieves state-of-the-art accuracy on a broad range of video recognition benchmarks, including action recognition (`84.9` top-1 accuracy on Kinetics-400 and `86.1` top-1 accuracy on Kinetics-600 with `~20x` less pre-training data and `~3x` smaller model size) and temporal modeling (`69.6` top-1 accuracy on Something-Something v2).

## Usage

###  Installation
```
pip install -r requirements.txt
```
If this does not work, please refer to the official [install.md](https://github.com/SwinTransformer/Video-Swin-Transformer/blob/master/docs/install.md) for installation.


### Prepare
```
git clone https://github.com/haofanwang/video-swin-transformer-pytorch.git
```
```
cd video-swin-transformer-pytorch
mkdir checkpoints && cd checkpoints
wget https://github.com/SwinTransformer/storage/releases/download/v1.0.4/swin_base_patch244_window1677_sthv2.pth
cd ..
```
If you want to try different models, please refer to [Video-Swin-Transformer](https://github.com/SwinTransformer/Video-Swin-Transformer) and download corresponding pretrained weight, then modify the config and pretrained weight.

### Inference
```
import torch
import torch.nn as nn
from video_swin_transformer import SwinTransformer3D

model = SwinTransformer3D()
print(model)

dummy_x = torch.rand(1, 3, 32, 224, 224)
logits = model(dummy_x)
print(logits.shape)
```

If you want to use the pre-trained model provided in the official repository, please follow the [INSTALL](https://github.com/SwinTransformer/Video-Swin-Transformer/blob/master/docs/install.md).

```
from mmcv import Config, DictAction
from mmaction.models import build_model
from mmcv.runner import get_dist_info, init_dist, load_checkpoint

config = './configs/recognition/swin/swin_base_patch244_window1677_sthv2.py'
checkpoint = './checkpoints/swin_base_patch244_window1677_sthv2.pth'

cfg = Config.fromfile(config)
model = build_model(cfg.model, train_cfg=None, test_cfg=cfg.get('test_cfg'))
load_checkpoint(model, checkpoint, map_location='cpu')

# [batch_size, channel, temporal_dim, height, width]
dummy_x = torch.rand(1, 3, 32, 224, 224)

# SwinTransformer3D without cls_head
backbone = model.backbone

# [batch_size, hidden_dim, temporal_dim/2, height/32, width/32]
feat = backbone(dummy_x)

# alternative way
feat = model.extract_feat(dummy_x)

# mean pooling
feat = feat.mean(dim=[2,3,4]) # [batch_size, hidden_dim]

# project
batch_size, hidden_dim = feat.shape
feat_dim = 512
proj = nn.Parameter(torch.randn(hidden_dim, feat_dim))

# final output
output = feat @ proj # [batch_size, feat_dim]
```

## Acknowledgement
The code is adapted from the official [Video-Swin-Transformer](https://github.com/SwinTransformer/Video-Swin-Transformer) repository. This project is inspired by [swin-transformer-pytorch](https://github.com/berniwal/swin-transformer-pytorch), which provides the simplest code to get started.


## Citation
If you find our work useful in your research, please cite:

```
@article{liu2021video,
  title={Video Swin Transformer},
  author={Liu, Ze and Ning, Jia and Cao, Yue and Wei, Yixuan and Zhang, Zheng and Lin, Stephen and Hu, Han},
  journal={arXiv preprint arXiv:2106.13230},
  year={2021}
}

@article{liu2021Swin,
  title={Swin Transformer: Hierarchical Vision Transformer using Shifted Windows},
  author={Liu, Ze and Lin, Yutong and Cao, Yue and Hu, Han and Wei, Yixuan and Zhang, Zheng and Lin, Stephen and Guo, Baining},
  journal={arXiv preprint arXiv:2103.14030},
  year={2021}
}
```
