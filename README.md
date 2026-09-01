# UniV2X_Fusion_SQSA

> **Attribution and Scope**
>
> This repository is a derivative research implementation built on the official
> **UniV2X** project.
>
> The original UniV2X framework, inherited network modules, data-processing
> pipeline, communication mechanism, training/evaluation infrastructure, and
> official pretrained checkpoints are third-party work and are **not claimed as
> original contributions of this repository**.
>
> This repository focuses on the SQSA-specific modification for cross-agent
> Query association and feature fusion.

---

## Base Project: UniV2X

This project is built upon the following open-source work:

**UniV2X: End-to-End Autonomous Driving through V2X Cooperation**

- Authors: Haibao Yu, Wenxian Yang, Jiaru Zhong, Zhenwei Yang, Siqi Fan, Ping Luo, and Zaiqing Nie
- Official Repository: https://github.com/AIR-THU/UniV2X
- arXiv: https://arxiv.org/abs/2404.00717
- AAAI 2025 Paper: https://ojs.aaai.org/index.php/AAAI/article/view/33040

We sincerely thank the UniV2X authors for releasing their source code,
pretrained models, and experimental framework.

The original UniV2X framework and inherited components remain the work of the
original UniV2X authors. This repository does not claim authorship of those
parts.

---

## SQSA-Specific Modifications

Compared with the original UniV2X implementation, the SQSA-related modifications
in this repository mainly include:

- **Soft-gated Query-level Sinkhorn Association (SQSA)**
- Sinkhorn-based soft association between vehicle-side and infrastructure-side Queries
- Confidence-aware residual feature fusion
- Query-level cross-agent fusion optimization
- Complementary infrastructure Query selection
- Auxiliary matching supervision
- Multi-stage SQSA training configurations
- Corresponding evaluation and ablation configurations

The remaining UniV2X framework is retained as the base system unless otherwise
stated.

---

## Documentation

This repository only maintains documentation for the **SQSA-specific training,
evaluation, hyperparameters, experimental results, and checkpoint usage**.

For environment installation, dataset preparation, and the original UniV2X
pipeline, please refer to the official UniV2X repository:

https://github.com/AIR-THU/UniV2X

For the SQSA-specific experimental documentation in **this repository**, see:

### [SQSA Training and Evaluation](https://github.com/links-z/UniV2X_Fusion_SQSA/blob/main/docs/TRAIN_EVAL.md)

The link above points to:

```text
links-z/UniV2X_Fusion_SQSA
└── docs/
    └── TRAIN_EVAL.md
```

---

## Dataset and Evaluation Resources

The experiments use third-party benchmark resources. These resources are not
created by this repository.

### V2X-Seq / Sequential Perception Dataset

The experiments use the Sequential Perception Dataset of **V2X-Seq**.

V2X-Seq is third-party work and is not claimed as a dataset created by this
repository.

- Official Project: https://github.com/AIR-THU/DAIR-V2X-Seq
- CVPR 2023 Paper: https://openaccess.thecvf.com/content/CVPR2023/html/Yu_V2X-Seq_A_Large-Scale_Sequential_Dataset_for_Vehicle-Infrastructure_Cooperative_Perception_and_CVPR_2023_paper.html

### nuScenes Evaluation Protocol

The detection and tracking evaluation metrics used in the experimental pipeline
follow the nuScenes evaluation protocol.

The nuScenes metric definitions and evaluation resources are third-party work
and are not introduced by this repository.

- Official Website: https://www.nuscenes.org/

Detailed attribution and experimental settings are provided in:

### [SQSA Training and Evaluation](https://github.com/links-z/UniV2X_Fusion_SQSA/blob/main/docs/TRAIN_EVAL.md)

---

## Checkpoints

### UniV2X Initialization Checkpoint

SQSA training is initialized from the official UniV2X Cooperation Planning
checkpoint:

```text
univ2x_coop_e2e_stg2.pth
```

This checkpoint is released by the original UniV2X authors and is **not**
a checkpoint originally trained or released by this repository.

Please obtain the original initialization checkpoint from the official UniV2X
project:

https://github.com/AIR-THU/UniV2X

Recommended local location:

```text
ckpts/univ2x_coop_e2e_stg2.pth
```

### SQSA Checkpoint

The final checkpoint produced from the SQSA training described in this
repository is named:

```text
SQSA.pth
```

Recommended local location:

```text
ckpts/SQSA.pth
```

For the exact training stages and evaluation procedure, see:

### [SQSA Training and Evaluation](https://github.com/links-z/UniV2X_Fusion_SQSA/blob/main/docs/TRAIN_EVAL.md)

---

## Experimental Results

The **UniV2X baseline** below refers to the original UniV2X method evaluated
under the same experimental environment and evaluation protocol.

The UniV2X method itself is the work of the original UniV2X authors and is
**not claimed as an original contribution of this repository**.

The **UniV2X + SQSA** row reports the results obtained after applying the
SQSA modification implemented in this repository.

### Overall Results

| Method | mAP ↑ | NDS ↑ | AMOTA ↑ | MOTA ↑ | Recall ↑ | IDS ↓ | FRAG ↓ | FAF ↓ | FP ↓ |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| UniV2X baseline | 0.0526 | 0.0528 | 0.118 | 0.119 | 0.205 | 120 | 80 | 39.9 | 661 |
| **UniV2X + SQSA** | **0.0560** | **0.0545** | **0.158** | **0.160** | **0.242** | **39** | **60** | **38.9** | **636** |
| Improvement | +0.0034 | +0.0017 | +0.040 | +0.041 | +0.037 | -81 | -20 | -1.0 | -25 |

### Car-Class Results

| Method | AP ↑ | AMOTA ↑ | MOTA ↑ | Recall ↑ | IDS ↓ | FRAG ↓ | FAF ↓ | FP ↓ |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| UniV2X baseline | 0.290 | 0.225 | 0.194 | 0.318 | 103 | 70 | 86.1 | 553 |
| **UniV2X + SQSA** | **0.323** | **0.300** | **0.256** | **0.346** | **31** | **48** | **70.0** | **450** |

For detailed experimental settings, training stages, hyperparameters, and
evaluation commands, see:

### [SQSA Training and Evaluation](https://github.com/links-z/UniV2X_Fusion_SQSA/blob/main/docs/TRAIN_EVAL.md)

---

## Attribution Summary

The provenance of the main components used in this repository is summarized
below.

| Item | Source / Attribution |
| --- | --- |
| UniV2X framework | Original work by Haibao Yu et al. |
| UniV2X source code and inherited pipeline | Official AIR-THU/UniV2X repository |
| `univ2x_coop_e2e_stg2.pth` | Official UniV2X pretrained checkpoint |
| V2X-Seq / SPD | Third-party dataset from the V2X-Seq project |
| nuScenes evaluation protocol | Third-party evaluation resource |
| SQSA-related module | SQSA-specific modification implemented in this repository |
| SQSA training configurations | SQSA-specific experimental configurations in this repository |
| `SQSA.pth` | Checkpoint produced from the SQSA training described in this repository |

This repository does not claim authorship of the original UniV2X framework,
inherited UniV2X code, official UniV2X pretrained models, V2X-Seq dataset,
nuScenes evaluation resources, or other third-party components.

Existing copyright notices, author information, and license headers in inherited
source files should be preserved.

---

## Acknowledgement

This repository is built upon the official UniV2X implementation:

> Haibao Yu, Wenxian Yang, Jiaru Zhong, Zhenwei Yang, Siqi Fan, Ping Luo, and Zaiqing Nie.  
> **End-to-End Autonomous Driving through V2X Cooperation.**  
> AAAI Conference on Artificial Intelligence, 2025.

We sincerely thank the UniV2X authors for releasing their source code,
pretrained models, and experimental framework.

We also acknowledge the authors of V2X-Seq for releasing the dataset and
benchmark resources, as well as the nuScenes team for the evaluation protocol
and metric definitions used in the experimental pipeline.

This repository does not claim authorship of these third-party resources.

### Original UniV2X Citation

```bibtex
@inproceedings{yu2024_univ2x,
  title={End-to-End Autonomous Driving through V2X Cooperation},
  author={Haibao Yu and Wenxian Yang and Jiaru Zhong and Zhenwei Yang and Siqi Fan and Ping Luo and Zaiqing Nie},
  booktitle={The 39th Annual AAAI Conference on Artificial Intelligence},
  year={2025}
}
```

Official UniV2X Repository:

https://github.com/AIR-THU/UniV2X

Official AAAI Paper:

https://ojs.aaai.org/index.php/AAAI/article/view/33040

---

## License

This repository contains code inherited from the UniV2X project and retains the
applicable upstream license notices.

The official UniV2X repository states that its assets and code are released
under the **Apache License 2.0**, unless otherwise specified.

Please also follow any separate license or attribution requirements associated
with third-party datasets, pretrained models, evaluation tools, and dependencies
used by this repository.

Existing copyright notices, license headers, and author attributions in
inherited source files should be preserved.
