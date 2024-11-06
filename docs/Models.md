# Models

[English](Models.md) **|** [简体中文](BasicSR_docs_CN.md)

#### Contents

1. [Supported Models](#Supported-Models)
1. [Inheritance Relationship](#Inheritance-Relationship)

## Supported Models

| Class         | Description    |Supported Algorithms |
| :------------- | :----------:| :----------:    |
| [BaseModel](../basicsr_lite/models/base_model.py) | Abstract base class, define common functions||
| [SRModel](../basicsr_lite/models/sr_model.py) | Base image SR class | SRCNN, EDSR, SRResNet, RCAN, RRDBNet, etc |
| [SRGANModel](../basicsr_lite/models/srgan_model.py) | SRGAN image SR class | SRGAN |
| [ESRGANModel](../basicsr_lite/models/esrgan_model.py) | ESRGAN image SR class|ESRGAN|
| [VideoBaseModel](../basicsr_lite/models/video_base_model.py) | Base video SR class | |
| [EDVRModel](../basicsr_lite/models/edvr_model.py) | EDVR video SR class |EDVR|
| [StyleGAN2Model](../basicsr_lite/models/stylegan2_model.py) | StyleGAN2 generation class |StyleGAN2|

## Inheritance Relationship

In order to reuse components among models, we use a lot of inheritance. The following is the inheritance relationship:

```txt
BaseModel
├── SRModel
│   ├── SRGANModel
│   │   └── ESRGANModel
│   └── VideoBaseModel
│       ├── VideoGANModel
│       └── EDVRModel
└── StyleGAN2Model
```
