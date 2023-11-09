# 知识图谱补全代码模版



# 1 代码结构介绍

- base_models:基础模型，模型基类模块包含各个模型的基类，基类仅包含初始化和前向传播的过程，训练和评估都集成在模型模块的模型类中。

- models:在models文件夹中，其实就是对base_models中的一个个trainer。因为每个模型的具体训练过程是不同的。
  models中都继承基类 `MateModel`
  
  ```python
    class MateModel(nn.Module):
        def __init__(self):
            super(MateModel, self).__init__()
    		
        # 训练
        def train_epoch(self,
                        batch_size=128):
            raise NotImplementedError
    		
        # 测试
        def test(self,
                 batch_size=128,
                 dataset='valid',
                 metric_list=None,
                 filter_out=False):
            raise NotImplementedError  # it will be call in main.evaluate and main.train
    		
        # 获取配置
        def get_config(self):
            raise NotImplementedError
  ```
  
- data: 存放数据集


- llm: 存放大语言模型或者预训练模型的权重


- shell: 存放一些执行脚本命令


- test: 存放一些测试用例


- uils: 存在一些工具函数



# 2 模型类别介绍

### 静态模型（静态知识图谱上进行推理通常需要采取过滤）：

- [TransE (2013)](https://proceedings.neurips.cc/paper/2013/hash/1cecc7a77928ca8133fa24680a88d2f9-Abstract.html)
- [DistMult (2015)](http://arxiv.org/abs/1412.6575)
- [R-GCN (2018)](https://link.springer.com/chapter/10.1007/978-3-319-93417-4_38)
- [SACN (2019)](https://ojs.aaai.org/index.php/AAAI/article/view/4164)

### 时序图谱：

- [CyGNet (2021)](https://ojs.aaai.org/index.php/AAAI/article/view/16604)
- [RE-GCN (2021)](https://dl.acm.org/doi/abs/10.1145/3404835.3462963)
- [CNE (2022)](https://arxiv.org/abs/2203.07782)
- [CENET (2023)](http://arxiv.org/abs/2211.10904)

## 多模态图谱：

- MKGFormer
- Mose
- DRGAT
- HRGAT



使用者可以将集成的模型作为基线模型使用。也可以通过研究这些模型，在其基础上设计新模型。
方法库的模型训练与主干网络都尽量进行了解耦合，可以将模型基类模块作为新模型的一个模块去搭建网络。

此外方法库还包含一个可执行main.py脚本，提供了模型训练、评估、保存checkpoint、根据实验数据绘图等功能。

# 3 环境配置

个人使用的配置，仅供参考，大版本保持一致应该都可以。

```
python 3.9.16
pytorch 2.0.1
```

# 4 快速开始

训练模型时至少需要指定模型和数据集两个参数，其他参数都有默认值，例如：

```shell
python main.py --dataset ICEWS14s --model cygnet
```

其他常用参数设置示例如下：

```shell
# --epoch 迭代轮数，默认30
python main.py --dataset ICEWS14s --model cygnet --epoch 100

# --batch-size batch的大小，默认1024
python main.py --dataset ICEWS14s --model cygnet --batch-size 256

# --lr 学习率，默认1e-3
python main.py --dataset ICEWS14s --model cygnet --lr 1e-5

# --gpu 选取gpu号（单gpu只能指定为0），默认自动选择显存占用最低的gpu
python main.py --dataset ICEWS14s --model cygnet --gpu 0

# --early-stop 早停轮数，默认不采取早停
python main.py --dataset ICEWS14s --model cygnet --early-stop 3

# --filter 采取过滤，默认不采取过滤
python main.py --dataset ICEWS14s --model cygnet --filter

# 随机种子，默认为0
python main.py --dataset ICEWS14s --model cygnet --seed 123

# 配置模型参数，默认采用模型默认参数，激活配置参数后需要手动输入模型的各个超参数
python main.py --dataset ICEWS14s --model cygnet --config
```

以下是各个模型推荐的配置，不一定是最佳，可以自己再对超参数进行调整

static

```shell
# TransE
python main.py --model transe --dataset FB15k  --early-stop 3 --filter

# R-GCN
pyhton main.py --mldel  rgcn --dataset FB15k --early-stop 3 --filter

# DistMult
pyhton main.py --model distmult --dataset FB15k --early-stop 3 --filter --lr 1e-4
```

temporal

```shell
# CyGNet
python main.py --dataset ICEWS14s --model cygnet --amsgrad --early-stop 3

# RE-GCN
python main.py --dataset ICEWS14s --model regcn  --weight-decay 1e-5 --early-stop 3

# CEN
python main.py --dataset ICEWS14s --model cen  --weight-decay 1e-5 --early-stop 3

# CeNet
python main.py --dataset ICEWS14s --model cenet  --amsgrad --weight-decay 1e-5 --early-stop 3

```

multimodal

```shell

```

加载保存的模型继续训练，至少需要指定模型和 checkpoint id，训练轮数和早停等参数可选

```shell
# 指定 checkpoint id, 以 checkpoint id 为 20230808085552 为例
python main.py --model cygnet --checkpoint 20230808085552
```

执行以下脚本加载checkpoint在测试集上进行评估

```shell
# 激活--test参数并指定 checkpoint id
python main.py  --model cygnet --test --checkpoint 20230808085552
```

# 5 历史

- 2023-09-24：初始化了该仓库，该仓库参考了KGMH而来，后续将在这里一直更新
