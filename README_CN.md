# :rocket: BasicSR Examples

[![download](https://img.shields.io/github/downloads/xinntao/BasicSR-examples/total.svg)](https://github.com/xinntao/BasicSR-examples/releases)
[![Open issue](https://img.shields.io/github/issues/xinntao/BasicSR-examples)](https://github.com/xinntao/BasicSR-examples/issues)
[![Closed issue](https://img.shields.io/github/issues-closed/xinntao/BasicSR-examples)](https://github.com/xinntao/BasicSR-examples/issues)
[![LICENSE](https://img.shields.io/github/license/xinntao/basicsr-examples.svg)](https://github.com/xinntao/BasicSR-examples/blob/master/LICENSE)
[![python lint](https://github.com/xinntao/BasicSR/actions/workflows/pylint.yml/badge.svg)](https://github.com/xinntao/BasicSR/blob/master/.github/workflows/pylint.yml)

[English](README.md) **|** [简体中文](README_CN.md) <br>
[`BasicSR repo`](https://github.com/xinntao/BasicSR) **|** [`simple mode example`](https://github.com/xinntao/BasicSR-examples/tree/master) **|** [`installation mode example`](https://github.com/xinntao/BasicSR-examples/tree/installation)

在这个仓库中，我们通过简单的例子来说明：如何在**你自己的项目中**轻松地**使用** [`BasicSR`](https://github.com/xinntao/BasicSR)。

:triangular_flag_on_post: **使用 BasicSR 的项目**
- :white_check_mark: [**GFPGAN**](https://github.com/TencentARC/GFPGAN): 真实场景人脸复原的实用算法
- :white_check_mark: [**Real-ESRGAN**](https://github.com/xinntao/Real-ESRGAN): 通用图像复原的实用算法

如果你的开源项目中使用了`BasicSR`, 欢迎联系我 ([邮件](#e-mail-%E8%81%94%E7%B3%BB)或者开一个issue/pull request)。我会将你的开源项目添加到上面的列表中 :blush:

---

如果你觉得这个项目对你有帮助，欢迎 :star: 这个仓库或推荐给你的朋友。Thanks:blush: <br>
其他推荐的项目:<br>
:arrow_forward: [facexlib](https://github.com/xinntao/facexlib): 提供实用的人脸相关功能的集合<br>
:arrow_forward: [HandyView](https://github.com/xinntao/HandyView): 基于PyQt5的 方便的看图比图工具

---

## 目录

- [如何使用 BasicSR](#HowTO-use-BasicSR)
- [作为Template](#As-a-Template)

## HowTO use BasicSR

`BasicSR` 有两种使用方式：
- :arrow_right: Git clone 整个 BasicSR 的代码。这样可以看到 BasicSR 完整的代码，然后根据你自己的需求进行修改
- :arrow_right: BasicSR 作为一个 [python package](https://pypi.org/project/basicsr/#history) (即可以通过pip安装)，提供了训练的框架，流程和一些基本功能。你可以基于 basicsr 方便地搭建你自己的项目
    ```bash
    pip install basicsr
    ```

我们的样例主要针对第二种使用方式，即如何基于 basicsr 这个package来方便简洁地搭建你自己的项目。

使用 basicsr 的python package又有两种方式，我们分别提供在两个 branch 中:
- :arrow_right: [简单模式](https://github.com/xinntao/BasicSR-examples/tree/master): 项目的仓库不需要安装，就可以运行使用。但它有局限：不方便 import 复杂的层级关系；在其他位置也不容易访问本项目中的函数
- :arrow_right: [安装模式](https://github.com/xinntao/BasicSR-examples/tree/installation): 项目的仓库需要安装 `python setup.py develop`，安装之后 import 和使用都更加方便

这个`installation`分支使用 *安装模式* 进行说明。在实际使用中我们也推荐 *安装模式*。

```bash
git clone -b installation https://github.com/dawnlh/BasicSR-Lite.git
cd BasicSR-Lite
python setup.py develop  # need to install
```

**注意**: 安装模式需要一个 包的名字 来安装。在这里，我们使用 `basicsr_lite` 作为包名。

### 预备

大部分的深度学习项目，都可以分为以下几个部分：

1. **data**: 定义了训练数据，来喂给模型的训练过程
2. **arch** (architecture): 定义了网络结构 和 forward 的步骤
3. **model**: 定义了在训练中必要的组件（比如 loss） 和 一次完整的训练过程（包括前向传播，反向传播，梯度优化等），还有其他功能，比如 validation等
4. training pipeline: 定义了训练的流程，即把数据 dataloader，模型，validation，保存 checkpoints 等等串联起来

当我们开发一个新的方法时，我们往往在改进: **data**, **arch**, **model**；而很多流程、基础的功能其实是共用的。那么，我们希望可以专注于主要功能的开发，而不要重复造轮子。

因此便有了 BasicSR，它把很多相似的功能都独立出来，我们只要关心 **data**, **arch**, **model** 的开发即可。

为了进一步方便大家使用，我们提供了 basicsr package，大家可以通过 `pip install basicsr` 方便地安装，然后就可以使用 BasicSR 的训练流程以及在 BasicSR 里面已开发好的功能啦~

### 简单的例子

下面我们就通过一个简单的例子，来说明如何使用 BasicSR 来搭建你自己的项目。

我们提供了两个样例数据来做展示，
1. [BSDS100](https://github.com/xinntao/BasicSR-examples/releases/download/0.0.0/BSDS100.zip) for training
1. [Set5](https://github.com/xinntao/BasicSR-examples/releases/download/0.0.0/Set5.zip) for validation

在 BasicSR-example 的根目录运行下面的命令来下载：

```bash
python scripts/prepare_example_data.py
```

样例数据就下载在 `datasets/example` 文件夹中。

#### :zero: 目的

我们来假设一个超分辨率 (Super-Resolution) 的任务，输入一张低分辨率的图片，输出高分辨率的图片。低分辨率图片包含了 1) cv2 的 bicubic X4 downsampling 和 2) JPEG 压缩 (quality=70)。

为了更好的说明如何使用 arch 和 model，我们想要使用 1) 类似 SRCNN 的网络结构；2) 在训练中同时使用 L1 和 L2 (MSE) loss。

那么，在这个任务中，我们要做的是:

1. 构建自己的 data loader
1. 确定使用的 architecture
1. 构建自己的 model

下面我们分别来说明一下。

#### :one: data

这个部分是用来确定喂给模型的数据的。

这个 dataset 的例子在[basicsr_lite/data/example_dataset.py](basicsr_lite/data/example_dataset.py) 中，它完成了:
1. 我们读取 Ground-Truth (GT) 的图像。读取的操作，BasicSR 提供了[FileClient](https://github.com/xinntao/BasicSR/blob/master/basicsr/utils/file_client.py), 可以方便地读取 folder, lmdb 和 meta_info txt 指定的文件。在这个例子中，我们通过读取 folder 来说明，更多的读取模式可以参考 [basicsr/data](https://github.com/xinntao/BasicSR/tree/master/basicsr/data)
1. 合成低分辨率的图像。我们直接可以在 `__getitem__(self, index)` 的函数中实现我们想要的操作，比如降采样和添加 JPEG 压缩。很多基本操作都可以在 [[basicsr/data/degradations]](https://github.com/xinntao/BasicSR/blob/master/basicsr/data/degradations.py), [[basicsr/data/tranforms]](https://github.com/xinntao/BasicSR/blob/master/basicsr/data/transforms.py) 和 [[basicsr/data/data_util]](https://github.com/xinntao/BasicSR/blob/master/basicsr/data/data_util.py) 中找到
1. 转换成 Torch Tensor，返回合适的信息

**注意**：
1. 需要在 `ExampleDataset` 前添加 `@DATASET_REGISTRY.register()`，以便注册好新写的 dataset。这个操作主要用来防止出现同名的 dataset，从而带来潜在的 bug
1. 新写的 dataset 文件要以 `_dataset.py` 结尾，比如 `example_dataset.py`。 这样，程序可以**自动地** import，而不需要手动地 import

在 [option 配置文件中](options/example_option.yml)使用新写的 dataset：

```yaml
datasets:
  train:  # training dataset
    name: ExampleBSDS100
    type: ExampleDataset  # the class name

    # ----- the followings are the arguments of ExampleDataset ----- #
    dataroot_gt: datasets/example/BSDS100
    io_backend:
      type: disk

    gt_size: 128
    use_flip: true
    use_rot: true

    # ----- arguments of data loader ----- #
    use_shuffle: true
    num_worker_per_gpu: 3
    batch_size_per_gpu: 16
    dataset_enlarge_ratio: 10
    prefetch_mode: ~

  val:  # validation dataset
    name: ExampleSet5
    type: ExampleDataset
    dataroot_gt: datasets/example/Set5
    io_backend:
      type: disk
```

#### :two: arch

Architecture 的例子在 [basicsr_lite/archs/example_arch.py](basicsr_lite/archs/example_arch.py)中。它主要搭建了网络结构。

**注意**：
1. 需要在 `ExampleArch` 前添加 `@ARCH_REGISTRY.register()`，以便注册好新写的 arch。这个操作主要用来防止出现同名的 arch，从而带来潜在的 bug
1. 新写的 arch 文件要以 `_arch.py` 结尾，比如 `example_arch.py`。 这样，程序可以**自动地** import，而不需要手动地 import

在 [option 配置文件中](options/example_option.yml)使用新写的 arch:

```yaml
# network structures
network_g:
  type: ExampleArch  # the class name

  # ----- the followings are the arguments of ExampleArch ----- #
  num_in_ch: 3
  num_out_ch: 3
  num_feat: 64
  upscale: 4
```

#### :three: model

Model 的例子在 [basicsr_lite/models/example_model.py](basicsr_lite/models/example_model.py)中。它主要搭建了模型的训练过程。
在这个文件中：
1. 我们从 basicsr 中继承了 `SRModel`。很多模型都有相似的操作，因此可以通过继承 [basicsr/models](https://github.com/xinntao/BasicSR/tree/master/basicsr/models) 中的模型来更方便地实现自己的想法，比如GAN模型，Video模型等
1. 使用了两个 Loss： L1 和 L2 (MSE) loss
1. 其他很多内容，比如 `setup_optimizers`, `validation`, `save`等，都是继承于 `SRModel`

**注意**：
1. 需要在 `ExampleModel` 前添加 `@MODEL_REGISTRY.register()`，以便注册好新写的 model。这个操作主要用来防止出现同名的 model，从而带来潜在的 bug
1. 新写的 model 文件要以 `_model.py` 结尾，比如 `example_model.py`。 这样，程序可以**自动地** import，而不需要手动地 import

在 [option 配置文件中](options/example_option.yml)使用新写的 model:

```yaml
# training settings
train:
  optim_g:
    type: Adam
    lr: !!float 2e-4
    weight_decay: 0
    betas: [0.9, 0.99]

  scheduler:
    type: MultiStepLR
    milestones: [50000]
    gamma: 0.5

  total_iter: 100000
  warmup_iter: -1  # no warm up

  # ----- the followings are the configurations for two losses ----- #
  # losses
  l1_opt:
    type: L1Loss
    loss_weight: 1.0
    reduction: mean

  l2_opt:
    type: MSELoss
    loss_weight: 1.0
    reduction: mean
```

#### :four: training pipeline

整个 training pipeline 可以复用 basicsr_lite 里面的[basicsr_lite/train.py](basicsr_lite/train.py)，非常简洁。



#### :five: debug mode

至此，我们已经完成了我们这个项目的开发，下面可以通过 `debug` 模式来快捷地看看是否有问题:

```bash
python basicsr_lite/train.py -opt options/train/Example/train_example.yml --debug
```

只要带上 `--debug` 就进入 debug 模式。在 debug 模式中，程序每个iter都会输出，8个iter后就会进行validation，这样可以很方便地知道程序有没有bug啦~

#### :six: normal training

经过debug没有问题后，我们就可以正式训练了。

```bash
python basicsr_lite/train.py -opt options/train/Example/train_example.yml
```

如果训练过程意外中断需要 resume, 则使用 `--auto_resume` 可以方便地自动resume：
```bash
python basicsr_lite/train.py -opt options/train/Example/train_example.yml --auto_resume
```


#### :seven: test & inference

模型训练完成之后，就到了测试评估（test or evaluate）环节，测试评估时也是基于options文件夹给定的配置文件来进行：

```bash
python basicsr_lite/test.py -opt options/test/Example/test_example.yml
```

结果会保存在 `results` 文件夹下。

训练完成的模型也可能会直接用于实际应用中，即只进行推理（inference），获得模型输出，而不进行结果的指标评估等环节：

```bash
python inference/infer_example.py --model_path <MODEL_PATH> --input <INPUT_PATH> --output <OUTPUT_PATH>
```

至此，使用 `BasicSR_Lite` 开发你自己的项目就介绍完了，是不是很方便呀~ :grin:

## As a Template

你可以使用 BasicSR-Examples 作为你项目的模板。下面主要展示一下你可能需要的修改。

你需要根据需要修改以下文件:

1. 设置 *pre-commit* hook
    1. 在文件夹根目录, 运行
    > pre-commit install
2. 修改 `LICENSE` 文件<br>
    本仓库使用 *MIT* 许可, 根据需要可以修改成其他许可

由于安装模式需要包名，因此还需要将所有`basicsr_lite`名称修改为 YOUR_PACKAGE_NAME。
以下是包含`basicsr_lite`名称的详细位置：

1. The `basicsr_lite` folder
1. [setup.py](setup.py#L9); &emsp; [setup.py](setup.py#L48); &emsp;[setup.py](setup.py#L91)
1. [basicsr_lite/train.py](basicsr_lite/train.py#L4-L6)
1. [basicsr_lite/archs/\_\_init\_\_.py](basicsr_lite/archs/__init__.py#L11)
1. [basicsr_lite/data/\_\_init\_\_.py](basicsr_lite/data/__init__.py#L11)
1. [basicsr_lite/models/\_\_init\_\_.py](basicsr_lite/models/__init__.py#L11)

你也需要修改文件 [setup.py](setup.py#L88-L113) 中的相应信息。
