# SQSA Training and Evaluation

> **Attribution and scope**
>
> This repository is a derivative research implementation built on the official
> **UniV2X** project. The UniV2X framework, inherited network components,
> original training/evaluation infrastructure, and the official UniV2X
> checkpoint are third-party work and are **not claimed as original work of this
> repository**.
>
> Original UniV2X project:  
> https://github.com/AIR-THU/UniV2X
>
> Original paper:  
> **Haibao Yu, Wenxian Yang, Jiaru Zhong, Zhenwei Yang, Siqi Fan, Ping Luo,
> Zaiqing Nie. "End-to-End Autonomous Driving Through V2X Cooperation."
> AAAI 2025, 39(9): 9598–9606.**  
> https://ojs.aaai.org/index.php/AAAI/article/view/33040
>
> The official UniV2X repository states that its assets and code are released
> under the Apache License 2.0 unless otherwise specified.
>
> This document only describes the **SQSA-specific experimental settings,
> training strategy, results, and checkpoint usage**. For the original
> installation, data preparation, and base UniV2X pipeline, please refer to the
> official UniV2X repository.

---

## 1. Experimental Basis

### 1.1 Base Framework

The experiments are implemented on top of the official **UniV2X** framework.

The following components are inherited from or follow UniV2X and are not claimed
as contributions of this repository:

- the original UniV2X network framework;
- the original data-processing pipeline;
- the original communication mechanism;
- the original training/evaluation infrastructure;
- the official Cooperation Planning checkpoint
  `univ2x_coop_e2e_stg2.pth`.

The SQSA work in this repository modifies the cross-agent Query association and
fusion process in the original UniV2X fusion stage.

### 1.2 Dataset

The experiments use the **Sequential Perception Dataset (SPD)** of **V2X-Seq**.

V2X-Seq is a third-party dataset introduced by:

**Haibao Yu, Wenxian Yang, Hongzhi Ruan, Zhenwei Yang, Yingjuan Tang, Xu Gao,
Xin Hao, Yifeng Shi, Yifeng Pan, Ning Sun, Juan Song, Jirui Yuan, Ping Luo,
Zaiqing Nie. "V2X-Seq: A Large-Scale Sequential Dataset for
Vehicle-Infrastructure Cooperative Perception and Forecasting." CVPR 2023,
pp. 5486–5495.**

- Official project: https://github.com/AIR-THU/DAIR-V2X-Seq
- Paper:
  https://openaccess.thecvf.com/content/CVPR2023/html/Yu_V2X-Seq_A_Large-Scale_Sequential_Dataset_for_Vehicle-Infrastructure_Cooperative_Perception_and_CVPR_2023_paper.html

The dataset, annotations, and task definitions are not created by this
repository.

### 1.3 Initialization Checkpoint

SQSA training starts from the official UniV2X Cooperation Planning checkpoint:

```text
univ2x_coop_e2e_stg2.pth
```

This checkpoint is released by the UniV2X authors and is used only as the
initialization model for SQSA training.

Recommended location:

```text
ckpts/univ2x_coop_e2e_stg2.pth
```

---

## 2. SQSA Training Strategy

The following training strategy describes the SQSA experiments conducted in
this repository.

Except for the **SQSA fusion module** and the **bounding-box detection head
(`bbox head`)**, the other major network modules remain frozen during SQSA
training.

The optimizer is **AdamW** with:

```text
Weight decay: 0.01
```

### 2.1 Stage 1

Training length:

```text
600 iterations
```

Trainable components:

```text
SQSA fusion module
bbox head
```

Learning-rate multipliers:

```text
SQSA      : 1.0
bbox head : 0.1
```

Base learning rate:

```text
5e-5
```

Configuration:

```text
projects/configs_e2e_univ2x/univ2x_SQSA_Stage_1.py
```

The checkpoint obtained from Stage 1 is used to initialize Stage 2.

### 2.2 Stage 2

Training length:

```text
300 iterations
```

Trainable component:

```text
SQSA fusion module
```

Frozen component:

```text
bbox head
```

Learning-rate multiplier:

```text
SQSA : 0.05
```

Base learning rate:

```text
5e-5
```

Configuration:

```text
projects/configs_e2e_univ2x/univ2x_SQSA_Stage_2.py
```

The checkpoint obtained from Stage 2 is used to initialize Stage 3.

### 2.3 Stage 3

Stage 3 jointly optimizes the SQSA fusion module and the bbox head.

Learning-rate multipliers:

```text
SQSA      : 0.05
bbox head : 0.02
```

The base learning rate is reduced during later fine-tuning to:

```text
3e-5
1e-5
```

Stage 3 is divided into three consecutive 300-iteration segments:

```text
projects/configs_e2e_univ2x/univ2x_SQSA_Stage_3-1(300).py
projects/configs_e2e_univ2x/univ2x_SQSA_Stage_3-2(300).py
projects/configs_e2e_univ2x/univ2x_SQSA_Stage_3-3(300).py
```

| Configuration | Stage-3 Iteration Range |
| --- | ---: |
| `univ2x_SQSA_Stage_3-1(300).py` | 0–300 |
| `univ2x_SQSA_Stage_3-2(300).py` | 300–600 |
| `univ2x_SQSA_Stage_3-3(300).py` | 600–900 |

Stage 3 is trained for a total of **900 iterations**.

The best performance is obtained at **Stage-3 iteration 800**, corresponding to:

```text
Stage 1 : 600 iterations
Stage 2 : 300 iterations
Stage 3 : 800 iterations
------------------------------
Total   : 1700 iterations
```

The final model therefore comes from the third Stage-3 segment.

---

## 3. SQSA Hyperparameters

The following hyperparameters belong to the SQSA configuration used in this
repository:

| Hyperparameter | Value |
| --- | ---: |
| Auxiliary matching loss weight (`lambda_aux`) | 0.05 |
| Query complement threshold | 0.3 |
| Sinkhorn temperature | Learnable |

The SQSA implementation is mainly located in:

```text
projects/mmdet3d_plugin/univ2x/fusion_modules/sinkhorn_assoc.py
```

The corresponding training configurations are located in:

```text
projects/configs_e2e_univ2x/
```

---

## 4. Evaluation Metrics

The experiments report detection and tracking metrics used by the inherited
UniV2X evaluation pipeline.

These metric definitions are not claimed as contributions of SQSA.

Detection:

- **mAP**
- **NDS**

Tracking:

- **AMOTA**
- **MOTA**
- **Recall**
- **IDS**
- **FRAG**
- **FAF**
- **FP**

Higher values are better for **mAP, NDS, AMOTA, MOTA, and Recall**.

Lower values are better for **IDS, FRAG, FAF, and FP**.

---

## 5. Experimental Results

The **UniV2X baseline** below is the result obtained by evaluating the original
UniV2X baseline in our experimental environment under the same evaluation
protocol.

The baseline method itself is the work of the UniV2X authors; this repository
does **not** claim UniV2X as its own method.

The **UniV2X + SQSA** row reports the result after applying the SQSA modification
implemented in this repository.

### 5.1 Overall Results

| Method | mAP ↑ | NDS ↑ | AMOTA ↑ | MOTA ↑ | Recall ↑ | IDS ↓ | FRAG ↓ | FAF ↓ | FP ↓ |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| UniV2X baseline | 0.0526 | 0.0528 | 0.118 | 0.119 | 0.205 | 120 | 80 | 39.9 | 661 |
| **UniV2X + SQSA** | **0.0560** | **0.0545** | **0.158** | **0.160** | **0.242** | **39** | **60** | **38.9** | **636** |
| Improvement | +0.0034 | +0.0017 | +0.040 | +0.041 | +0.037 | -81 | -20 | -1.0 | -25 |

### 5.2 Car-Class Results

| Method | AP ↑ | AMOTA ↑ | MOTA ↑ | Recall ↑ | IDS ↓ | FRAG ↓ | FAF ↓ | FP ↓ |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| UniV2X baseline | 0.290 | 0.225 | 0.194 | 0.318 | 103 | 70 | 86.1 | 553 |
| **UniV2X + SQSA** | **0.323** | **0.300** | **0.256** | **0.346** | **31** | **48** | **70.0** | **450** |

These tables distinguish the third-party UniV2X baseline from the SQSA
modification evaluated in this repository.

---

## 6. Final SQSA Checkpoint

The final SQSA checkpoint corresponds to:

```text
Stage-3 iteration: 800
Total cumulative iteration: 1700
```

Checkpoint name:

```text
SQSA.pth
```

Recommended location:

```text
ckpts/SQSA.pth
```

The corresponding configuration is:

```text
projects/configs_e2e_univ2x/univ2x_SQSA_Stage_3-3(300).py
```

Example evaluation command:

```bash
CUDA_VISIBLE_DEVICES=${GPU_IDS} \
./tools/univ2x_dist_eval.sh \
"./projects/configs_e2e_univ2x/univ2x_SQSA_Stage_3-3(300).py" \
./ckpts/SQSA.pth \
${GPU_NUM}
```

---

## 7. Attribution Summary

To avoid ambiguity about authorship:

| Item | Source / ownership |
| --- | --- |
| UniV2X framework | Third-party work by Haibao Yu et al.; AAAI 2025 |
| UniV2X source code and inherited pipeline | Official AIR-THU/UniV2X repository |
| `univ2x_coop_e2e_stg2.pth` | Official UniV2X pretrained checkpoint |
| V2X-Seq / SPD | Third-party dataset by Haibao Yu et al.; CVPR 2023 |
| SQSA module and SQSA-specific training strategy documented here | Modification implemented in this repository |
| `SQSA.pth` | Checkpoint produced from the SQSA training described here |

This repository does not claim authorship of inherited UniV2X code, pretrained
models, the V2X-Seq dataset, or other third-party resources.

Existing copyright notices and license headers in inherited source files should
be preserved.

---

## 8. Original UniV2X Citation

If you use the UniV2X framework, please cite the original UniV2X paper:

```bibtex
@inproceedings{yu2024_univ2x,
  title={End-to-End Autonomous Driving through V2X Cooperation},
  author={Haibao Yu and Wenxian Yang and Jiaru Zhong and Zhenwei Yang and Siqi Fan and Ping Luo and Zaiqing Nie},
  booktitle={The 39th Annual AAAI Conference on Artificial Intelligence},
  year={2025}
}
```

Official repository:

https://github.com/AIR-THU/UniV2X

Official AAAI paper:

https://ojs.aaai.org/index.php/AAAI/article/view/33040
