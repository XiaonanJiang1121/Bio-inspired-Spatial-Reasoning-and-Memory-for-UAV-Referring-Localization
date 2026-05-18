# Research Proposal

## 题目

**Bio-inspired Spatial Reasoning and Memory for UAV Referring Localization**

**面向无人机指代表达定位的类脑空间推理与记忆增强方法**

## 1. Introduction

无人机视觉感知正在从传统的目标检测与场景识别，逐渐走向更高层次的语言引导空间理解。对于巡检、灾害监测、城市管理和低空智能系统而言，无人机不仅需要“看见”目标，还需要根据自然语言描述在大范围、复杂背景和小目标密集的空中视角中精准定位目标。例如，系统需要理解 “the white truck between two buildings near the road” 这类包含目标属性、参照物和空间关系的表达，并输出目标在图像中的精确位置。

现有referring expression comprehension与visual grounding方法在自然图像中已经取得了较好效果，但无人机视角带来了更强的挑战：第一，无人机图像视野范围大，目标尺度小，候选目标之间外观相似；第二，语言描述通常依赖道路、建筑、树木、车辆等地标形成空间约束，仅依赖视觉语义匹配容易产生混淆；第三，粗定位阶段能够召回候选目标，但 fine localization 阶段仍容易出现框偏移、目标边界不准和局部干扰物误包含的问题。

我们的前期工作 ReasonLoc 证明了显式空间关系建模对于语言定位任务的重要性。ReasonLoc 通过 Spatial Information Module (SIM) 建模对象间几何关系，并采用 coarse-to-fine 的定位框架提升 3D 点云中的语言定位性能。然而，ReasonLoc 主要面向 3D point cloud localization，其空间建模和语言监督形式与无人机图像中的 referring localization 仍存在差异。同时，ReasonLoc 中的 fine localization 阶段对 SIM 的利用仍然有限，空间关系对精细定位的提升并不显著。

基于此，ReasonLoc 将扩展为一个面向 UAV referring localization 的类脑空间认知框架。该框架保留 ReasonLoc 的 coarse-to-fine架构，但进一步引入海马体式记忆，使空间关系不仅用于候选重排，也能够参与最终的精细定位过程。我们将视觉感知、顶叶式空间关系建模、海马体式记忆绑定和前额叶式目标导向融合组合起来，构建一个更适合无人机复杂视角的语言定位模型。

## 2. Research Questions and Objectives

### 2.1 Research Questions

本研究主要关注以下问题：

**1: 如何改善 ReasonLoc 中 fine localization 对空间关系利用不足的问题？**
对于ReasonLoc本身来说，原 fine localization 中 SIM 带来的提升有限，说明空间关系可能没有参与 bbox refinement，或者说，原来的将SIM信息通过add形式注入并没有起到很好的效果。本研究希望设计 Memory-Guided Fine Localization，使 HOPM 检索到的记忆上下文和 P-SIM 关系上下文共同指导 bbox refinement。

**2: 海马体式 object-place memory 是否能够提升 coarse candidate reranking 与 fine localization？**
无人机图像中存在大量相似目标，仅依赖视觉匹配或单次空间关系判断容易出错。本研究希望通过 HOPM 将 object semantics、box position 和 spatial relation context 绑定为 object-place-relation memory，并让该记忆同时参与 coarse stage 和 fine stage。

**3: 如何构建一个可解释的类脑 UAV localization 框架？**
本研究希望将模型模块与认知功能形成合理对应：视觉皮层负责视觉感知，顶叶皮层负责空间关系建模，海马体负责 object-place binding，前额叶负责目标导向证据融合。该对应不只是命名，而需要体现在模块功能和信息流中。


### 2.2 Research Objectives

本研究的目标包括：

1. 构建一个面向 UAV referring localization 的 coarse-to-fine 类脑空间定位框架。
2. 将 ReasonLoc 的 SIM 扩展为 P-SIM，使其适配 box-level spatial relation。
3. 设计 HOPM，将视觉语义、空间位置和关系上下文绑定为 object-place-relation memory。
4. 设计 Memory-Guided Fine Localization，使 HOPM 和 P-SIM 同时参与精细定位。
5. 设计 PGR 作为目标导向证据融合头，综合视觉、关系、记忆和精修置信度输出最终定位。
6. 在 SkyFind、AerialVG、RefDrone、RefAerial、RefCOCO 等数据集上验证模型性能，并进行充分消融分析。

## 3. Research Rationale

### 3.1 Why UAV Referring Localization?

无人机巡检任务不仅需要检测目标，还需要根据任务指令和自然语言描述找到目标位置。与普通自然图像不同，无人机图像通常具有大视野、小目标、复杂背景和多目标干扰等特点。一个目标是否被正确定位，往往不仅取决于其外观，还取决于它和周围地标之间的空间关系。

因此，UAV referring localization 是一个适合验证空间推理能力的任务。它能够继承 ReasonLoc 的空间关系建模优势，同时进一步放大 ReasonLoc 在复杂空间关系、小目标定位和视觉-语言消歧方面的价值。

### 3.2 Why Bio-inspired Memory?


现有 localization / UAV REC 方法已经逐渐意识到空间结构的重要性。SkyFind 指出 UAV referring localization 面临大背景、小目标和复杂关系三重挑战，AerialREC 进一步说明 two-step localization 可以缓解背景干扰；VLM-Loc 也指出浅层文本-视觉匹配难以支撑复杂环境中的精细定位，因此引入 BEV 和 scene graph 来增强结构化空间理解。ReasonLoc 则通过 SIM 证明显式几何关系对于语言定位有效。

然而，这些方法大多停留在视觉-语言匹配、候选区域搜索或 pairwise relation modeling 层面。它们能够判断候选目标是否与文本匹配，或者候选与某个地标是否满足局部空间关系，但缺少一个跨阶段的 object-place-relation memory，用于持续保存“目标是什么、目标在哪里、它与哪些地标共同构成当前场景”的上下文。因此，在 coarse stage 和 fine stage 之间，空间关系容易被压缩成一次性分数或附加特征，难以稳定指导最终 bbox refinement。

这正是本研究引入 HOPM 的主要动机：将 P-SIM 产生的局部空间关系进一步绑定为 scene-level object-place-relation memory，使模型在 coarse reranking 中能够利用记忆一致性筛选候选，在 fine localization 中能够利用检索到的目标-地标记忆修正目标框。所以，HPOM 是针对跨阶段空间上下文缺失问题提出的记忆机制。

对于海马体记忆的解释，认知神经科学研究表明，海马体-内嗅皮层系统与空间表征、物体-位置关联和情景记忆密切相关：Moser 等关于 place cells 和 grid cells 的综述说明海马体和内嗅皮层构成了大脑空间表征系统；Knierim 与 Neunuebel 对 LEC / MEC 功能分工的讨论指出，海马体接收来自 MEC 的空间信息和来自 LEC 的非空间对象信息；关于 object-place association 的研究也表明，海马体和相关内嗅皮层通路参与物体与位置之间的关联记忆。近期类脑模型如 Hippoformer 和 Bio-inspired Spatial Reasoning Transformer 进一步说明，将海马体式空间记忆、位置编码和吸引子式空间推理引入 Transformer，有助于提升模型的空间推理能力。



### 3.3 Why Memory-Guided Fine Localization?

ReasonLoc 的 coarse stage 可以通过空间关系提升候选选择，但 fine localization 阶段可能依旧容易受局部视觉噪声、目标尺度小和相似干扰物影响。仅将 SIM 特征简单注入 fine localization 并不足以显著改善精细定位。

因此，本研究将 fine localization 从bbox regression提升为 Memory-Guided Fine Localization。该模块利用 HOPM 检索目标相关的 object-place memory，并结合 P-SIM 提供的目标-地标关系上下文，对候选框进行进一步修正。这样，空间关系和记忆不只影响候选排序，也直接参与最终 bbox 的精细调整。

## 4. Research Method

### 4.1 Problem Formulation

给定一张无人机图像或 aerial image，以及对应的referring expression，模型需要输出目标 bounding box：
模型评价指标与SkyFind进行对齐的：Acc@0.5 Acc@0.7 mIoU

### 4.2 Overall Framework

整体框架保留 ReasonLoc 的 coarse-to-fine 思想，但加入 P-SIM、HOPM、Memory-Guided Fine Localization 和 PGR：

```text
Image I + Text Q
  ↓
Visual Perception
  ↓
object / region / landmark candidates
  ↓
P-SIM: box-level spatial relation graph
  ↓
HOPM: object-place-relation memory
  ↓
Coarse Stage:
  visual score + relation score + memory consistency score
  → Top-K candidate reranking
  ↓
Fine Stage:
  local visual feature + P-SIM relation context + HOPM memory context
  → memory-guided bbox refinement
  ↓
PGR:
  goal-directed evidence fusion
  ↓
Final bbox
```

### 4.3 Visual Perception

ReasonLoc中的Visual Encoder在人脑中对应为Visual Perception 对应视觉皮层，负责提取 UAV 图像中的目标、区域和地标候选。对于 SkyFind / AerialVG / RefDrone 这类UAV REC任务，它需要处理：UAV视角下的小目标；大量背景干扰；同类目标密集出现；车辆、建筑、道路、树木、船只等目标与地标信息。该模块为后续 P-SIM 空间关系建模和 HOPM 记忆绑定提供基础视觉表示。

### 4.4 P-SIM: Parietal Box-level Spatial Information Module

P-SIM对应的是顶叶皮层的空间关系建模功能。它根据候选目标和地标之间的 bbox 几何关系构建空间关系图。顶叶皮层在空间认知中主要负责：1: 物体位置关系；2: 左右、上下、远近等空间判断；3: egocentric spatial representation，即以当前观察视角为中心的空间关系；4: 视觉信息与空间结构整合。这个模块其实就是ReasonLoc中SIM的扩展，但ReasonLoc面向3D点云和地图坐标，而SkyFind是面向UAV图像，因此需要将SIM改成box框间的关系

例如表达：“the white truck between two buildings near the road”P-SIM要进一步判断我们找到的truck和周边物体的空间关系。
P-SIM 在两个阶段发挥作用：
1. **Coarse stage**：提供 relation-aware reranking feature，判断候选是否满足语言中的空间约束。
2. **Fine stage**：提供 relation-guided refinement context，帮助 bbox refinement 判断目标框应如何修正。

### 4.5 HOPM: Hippocampal Object-Place Memory

HOPM 是本研究的核心新增模块，对应海马体式 object-place memory。它的核心是将视觉语义、空间位置和关系上下文绑定为 memory slots，在检索的时候可以在memory中进行检索。也就是说海马体会将视觉皮层提取的信息和顶叶提取的几何信息进行绑定，它把这些零散的碎片整合在一起。

在认知神经科学中，关于海马体记忆有一个模型“What-Where-Binding”模型。是说哺乳动物的大脑中，外部世界的视觉信息在进入海马体形成记忆之前，必须先经过内嗅皮层 (Entorhinal Cortex, EC)，其中又分成了LEC 和 MEC。这个模型中具体的含义是
1. LEC (Lateral Entorhinal Cortex) 外侧内嗅皮层：生物学功能中负责接收来自腹侧视觉流的信息，专门处理物体的非空间属性，比如物体的身份、颜色、类别、气味等
2. MEC (Medial Entorhinal Cortex) 内侧内嗅皮层：负责接收来自顶叶背侧流的信息，专门处理纯粹的空间和几何结构，这里也是网格细胞 (Grid Cells)所在的地方，它在大脑里铺设了一张不可见的坐标网格，负责计算距离、方向和相对位置。
3. Hippocampus：负责将LEC传来的物体特征和MEC传来的空间网格在海马体融合绑定。海马体利用“位置细胞 (Place Cells)”，将是什么和在哪里进行绑定，形成了一个情景记忆块。

HOPM 参与两个阶段：

1. **Coarse stage**：为候选目标提供 memory consistency score，辅助候选重排。
2. **Fine stage**：根据 query 和 Top-K candidates 检索相关 memory context，为 bbox refinement 提供 object-place-relation 记忆支持。

### 4.6 Coarse Stage: Memory-aware Candidate Reranking

Coarse stage 的目标是从大范围 UAV 图像中快速召回可能的目标候选，并将明显不符合语言描述的候选排除。这个阶段不追求最终框的精确边界，而是尽可能保证目标被包含在 Top-K candidates 中，为后续 fine localization 提供可靠搜索空间。

在 ReasonLoc 中，coarse stage 主要依赖文本与候选区域之间的语义匹配，并通过 SIM 引入空间关系重排。在本研究中，我们将这一阶段升级为 memory-aware candidate reranking。具体而言，候选目标首先由 Visual Perception 模块生成，并获得基础视觉-语言匹配结果；随后，P-SIM 检查候选目标与周围地标之间是否满足语言中的空间约束；最后，HOPM 判断该候选是否能够与当前场景中的 object-place-relation memory 保持一致。

因此，coarse stage 的重排依据主要包括三类证据：

1. **Visual matching evidence**：候选目标在外观、类别或属性上是否与文本描述一致。
2. **Spatial relation evidence**：候选目标与参照地标之间的方向、距离、包含、邻近等关系是否符合语言约束。
3. **Memory consistency evidence**：候选目标是否能够被合理地绑定到当前场景的 object-place memory 中。

该阶段的输出不是最终定位结果，而是 Top-K candidates 以及与这些候选相关的空间关系上下文和记忆检索上下文。这些上下文会被传递到 fine stage，用于进一步精修目标框。换言之，coarse stage 负责“找出可能目标”，而不是“决定最终目标”。

### 4.7 Memory-Guided Fine Localization

Memory-Guided Fine Localization 的目标是在 coarse stage 选出的候选基础上，进一步利用局部视觉特征、空间关系上下文和记忆上下文进行 bbox refinement。

ReasonLoc 原本的 fine localization 更接近于在 coarse candidate 内部做局部位置回归。即使加入 SIM，空间关系也主要以附加特征的形式进入 fine stage，对最终精度的提升帮助有限。我们的改进重点是让空间关系和记忆真正参与精修过程，而不只是作为额外信息被动相加。

对于每个 Top-K candidate，fine stage 会重新关注候选目标附近的局部区域，并结合三个来源的信息进行精修：

1. **Local visual evidence**：候选框内部及其周围区域的局部视觉特征，用于判断目标边界、尺度和局部外观。
2. **Relation-guided spatial Info**：P-SIM 提供的目标-地标关系信息，用于判断候选框中心是否偏离了合理空间位置，以及候选是否满足语言中的空间关系。
3. **Memory-guided object-place context**：HOPM 检索出的目标相关记忆和地标相关记忆，用于提供“目标应该处在什么位置、与哪些地标共同出现、在当前场景中应形成怎样的 object-place relation”。

在这一阶段，模型不只是判断哪个候选更好，而是对候选框进行具体修正。例如，当粗框只覆盖了目标的一部分时，局部视觉特征可以帮助扩展边界；当粗框偏向相邻干扰物时，P-SIM 的关系上下文可以帮助判断目标应向哪个方向移动；当多个相似目标同时出现时，HOPM 的 object-place memory 可以帮助模型选择与语言描述和场景记忆最一致的目标区域。

因此，Memory-Guided Fine Localization 的输出包括 refined bbox、fine-stage visual feature、relation-guided refinement feature、memory-guided refinement feature 和 refinement confidence。这些结果随后交给 PGR 做最终目标导向融合。

### 4.8 PGR: Prefrontal Goal-directed Reasoning Head

PGR 对应前额叶式目标导向证据融合，作为最终 localization decision head。在人类大脑中，前额叶皮层位于额头正后方。它不直接负责底层视觉编码、几何关系构建或记忆存储，而负责对这些证据进行目标导向融合。它的核心功能是执行控制、目标导向行为和工作记忆检索。

因此，PGR 的输入来自 fine stage 的结果和空间记忆上下文，主要包括：

1. **Refined candidate representation**：经过 fine localization 精修后的候选目标表示。
2. **Refined bbox**：fine stage 输出的目标框。
3. **Relation-guided refinement feature**：P-SIM 在 fine stage 中提供的空间关系验证与修正信息。
4. **Memory-guided refinement feature**：HOPM 在 fine stage 中检索到的 object-place-relation 记忆上下文。
5. **Refinement confidence**：fine stage 对当前精修结果可靠性的估计。

PGR 的任务不是重新执行 coarse reranking，而是判断 fine stage 的定位结果是否在视觉证据、空间关系和记忆上下文之间达到一致。最终，PGR 根据这些 fine-stage evidence 输出最终 bbox 和 final confidence。

### 4.9 Training Strategy and Implementation Notes

本研究首先需要在 SkyFind 上跑通一个 ReasonLoc-style baseline，确认数据层、候选生成、评价指标和 coarse-to-fine 基础流程能够正常work。随后再依次加入 P-SIM、HOPM、Memory-Guided Fine Localization 和 PGR，以便清楚判断每个模块是否真正带来性能提升。

在训练流程上，coarse stage 主要学习候选目标的召回与初步重排，使模型能够从大范围 UAV 图像中找到可能目标。P-SIM 在这一阶段学习候选目标与周围地标之间的空间关系一致性，并在后续 fine stage 中提供 relation-guided refinement context。HOPM 不作为单独的后处理模块，而是在训练中通过 coarse reranking 和 fine refinement 两个阶段共同学习 object-place-relation memory。

Fine stage 是本研究最关键的实验改进部分。它需要在 Top-K candidates 的基础上进一步学习 bbox refinement，并验证 P-SIM relation context 与 HOPM retrieved memory context 是否能够真正改善目标边界和中心位置。PGR 则放在 fine stage 之后训练，输入来自 refined candidate representation、relation-guided refinement feature、memory-guided refinement feature 和 refinement confidence，用于输出最终 bbox 与 final confidence。

因此，完整训练路线可以分为四步：第一，训练并评估 ReasonLoc-style baseline；第二，加入 P-SIM 并验证 relation-aware reranking；第三，加入 HOPM 并验证 memory consistency 与 memory-guided refinement；第四，加入 PGR 并完成完整模型训练与消融分析。

### 4.10 Experiments

本研究暂定使用以下数据集：

| Dataset | Source | Task Type | Metrics |
| --- | --- | --- | --- |
| SkyFind | TPAMI 2026| UAV referring localization | Acc@0.5, Acc@0.7, mIoU |
| AerialVG | ICCV 2025 | Aerial visual grounding | Acc@0.5, Acc@0.7, mIoU |
| RefDrone | arXiv 2025 WHU | UAV referring localization | Acc@0.5, Acc@0.7, mIoU |
| RefAerial | Shanghai AI Lab | Aerial referring localization | Acc@0.5, Acc@0.7, mIoU |
| RefCOCO / RefCOCO+ / RefCOCOg | General REC benchmark | General referring localization | Acc@0.5, Acc@0.7, mIoU |

主要实验包括：
1. 与 existing REC / visual grounding methods 比较。
2. 与 ReasonLoc baseline 比较。

拟重点比较的方法包括：

| Method | Reason for Comparison |
| --- | --- |
| TransVG | 传统 one-stage Transformer visual grounding 方法，将 grounding 建模为直接 bbox regression，可作为通用 REC 基础对照。 |
| SeqTR | SkyFind 中 sequence-to-sequence REC 方法，将 bbox 预测为坐标 token 序列，适合作为序列式定位 baseline。 |
| PolyFormer | SkyFind 中常见 REC 方法里表现最强的baseline之一。 |
| GroundingDINO / NGDINO | 在 武汉大学 RefDrone 数据集上评测，可进一步比较其针对 multi-target / no-target 场景提出的 NGDINO 变体。 |
| AerialREC | SkyFind 提出的 UAV REC baseline，通过先搜索潜在目标区域再精定位来缓解大视野背景干扰，和我们的 coarse-to-fine 框架最直接相关。 |
| AerialVG | AerialVG 数据集提出的 relation-aware aerial visual grounding 方法 |


### 4.11 Ablation Studies

为了验证各模块有效性，计划进行以下消融：

| Ablation | Purpose |
| --- | --- |
| w/o P-SIM | 验证空间关系建模是否有效 |
| w/o HOPM | 验证 object-place memory 是否有效 |
| w/o HOPM in coarse stage | 验证 memory consistency reranking 的作用 |
| w/o HOPM in fine stage | 验证 memory-guided refinement 的作用 |
| w/o Memory-Guided Fine Localization | 验证 fine localization 升级是否有效 |
| w/o PGR | 验证目标导向证据融合是否有效 |
| P-SIM only in coarse stage | 验证 P-SIM 是否需要参与 fine localization |
| P-SIM only in fine stage | 验证 P-SIM 对不同阶段的贡献 |

### 4.12 Expected Contributions

本研究预期贡献如下：

1. 提出一个面向 UAV referring localization 的类脑 coarse-to-fine 空间定位框架。
2. 将 ReasonLoc 的 SIM 扩展为 P-SIM，使其支持 UAV 图像中的 box-level / region-level spatial relation。
3. 引入 HOPM，将视觉语义、空间位置和关系上下文绑定为 object-place-relation memory，并跨 coarse stage 与 fine stage 使用。
4. 提出 Memory-Guided Fine Localization，使记忆和空间关系直接参与 bbox refinement。
5. 设计 PGR 作为前额叶式目标导向融合头，综合视觉、关系、记忆和精修证据输出最终定位。
6. 在 SkyFind 等 UAV/aerial referring localization 数据集上系统验证模型的空间推理和精细定位能力。

## 5. Timeline and Next Steps

### Stage 1: 数据层适配与 SkyFind pipeline 跑通

首先需要完成 SkyFind 数据集到现有 ReasonLoc-style pipeline 的转移。由于 ReasonLoc 原始代码主要面向 3D point cloud，而 SkyFind 是 UAV image-level referring localization，因此我的计划是先完成数据层和任务形式的转换。

这一阶段主要包括：

1. 得到 SkyFind 数据集。
2. 统一数据输出格式，使其能够提供 image、query、candidate boxes、ground-truth bbox 等信息。
3. 将原 ReasonLoc 的 coarse-to-fine 思想迁移到 2D bbox localization 设置中。
4. 实现与 SkyFind 对齐的评价指标，包括 Acc@0.5、Acc@0.7 和 mIoU。
5. 跑通一个不包含类脑模块的 ReasonLoc baseline，得到一个初步的baseline结果。

### Stage 2: P-SIM 的 box-level 空间关系适配

在 baseline 跑通后，第二阶段加入 P-SIM。该阶段将 ReasonLoc 原始 SIM 从 3D object relation 扩展为 UAV image 中的 box-level spatial relation。

这一阶段主要包括：

1. 构建候选目标与周围地标之间的 box-level relation graph。
2. 建模相对方向、距离、尺度差异、包含关系、邻近关系等空间信息。
3. 将 P-SIM 用于 coarse stage 的 relation-aware candidate reranking。
4. 对 P-SIM 在 fine stage 中提供 relation-guided refinement context 的方式进行初步实验。

### Stage 3: HOPM 类脑记忆模块实现

第三阶段开始实现核心类脑仿生模块 HOPM。HOPM 将同时参与 coarse stage 和 fine stage。

这一阶段主要包括：

1. 构建 object-place-relation memory slot，将视觉语义、bbox 位置和 P-SIM 关系上下文进行绑定。
2. 在 coarse stage 中引入 memory consistency，用于辅助 candidate reranking。
3. 在 fine stage 中根据 query 和 Top-K candidates 检索相关 memory context。
4. 设计 w/o HOPM、w/o HOPM in coarse、w/o HOPM in fine 等消融实验。

### Stage 4: Memory-Guided Fine Localization

第四阶段重点解决 ReasonLoc fine localization 提升有限的问题。该阶段将 fine localization 从普通 bbox regression 转变为 HOPM-guided refinement。

这一阶段主要包括：

1. 利用 Top-K candidates 提取局部视觉特征。
2. 将 P-SIM relation context 和 HOPM retrieved memory context 输入 fine localization。
3. 预测 refined bbox 和 refinement confidence。
4. 对比普通 fine localization、P-SIM-only refinement、HOPM-only refinement 和完整 memory-guided refinement。

### Stage 5: PGR 目标导向融合头与完整模型验证

第五阶段实现 PGR。在 fine stage 之后融合 refined candidate representation、P-SIM relation-guided refinement feature、HOPM memory-guided refinement feature 和 refinement confidence，输出最终 bbox 与 final confidence。

这一阶段主要包括：

1. 设计 PGR 作为 fine-stage evidence fusion head。
2. 验证 PGR 是否能够提升最终 bbox 选择和定位置信度。
3. 完成完整模型与 TransVG、SeqTR、PolyFormer、GroundingDINO / NGDINO、AerialREC、AerialVG 等方法的比较。
4. 完成模块消融和 subset analysis，例如 small-object subset、relation-heavy subset、multi-distractor subset。

### Tentative Timeline

| Time | Main Tasks | Expected Output |
| --- | --- | --- |
| Week 1 | ReasonLoc-style baseline 迁移到 SkyFind | 初始 baseline 结果，定位主要瓶颈 |
| Week 2-3 | 实现 P-SIM box-level relation graph 与 coarse reranking | P-SIM 初步实验与消融 |
| Week 4-5 | 实现 HOPM object-place-relation memory | HOPM 在 coarse / fine 两阶段的初步结果 |
| Week 6-7 | 实现 Memory-Guided Fine Localization | fine localization 对比实验 |
| Week 7 | 实现 PGR 并完成完整模型评测 | 完整模型主结果与消融结果 |
| Week 8 | 扩展到其他数据集，整理图表和论文内容 | proposal 修订、实验补充和论文初稿 |


## References

### Datasets

- [SkyFind: A Large-Scale Benchmark Unveiling Referring Expression Comprehension for UAV](https://github.com/wangkunyu241/SkyFind)
- [AerialVG: Relation-aware Aerial Visual Grounding](https://arxiv.org/abs/2504.07836)
- [RefDrone: A Referring Expression Comprehension Dataset for Drone Scene Understanding](https://arxiv.org/abs/2502.00392)
- [RefAerial: A Benchmark and Approach for Referring Detection in Aerial Images](https://www.researchgate.net/publication/404102315_RefAerial_A_Benchmark_and_Approach_for_Referring_Detection_in_Aerial_Images)
- [RefCOCO / RefCOCO+ / RefCOCOg: Generation and Comprehension of Unambiguous Object Descriptions](https://arxiv.org/abs/1511.02283)

### Brain-inspired Spatial Reasoning and Memory

- [Place Cells, Grid Cells, and the Brain's Spatial Representation System](https://www.annualreviews.org/content/journals/10.1146/annurev.neuro.31.061307.090723)
- [Tracking the Flow of Hippocampal Computation: Pattern Separation, Pattern Completion, and Attractor Dynamics](https://www.nature.com/articles/nrn3671)
- [What and Where: A Context-based Framework for Understanding Object–Place Associations in Mammalian Memory](https://pmc.ncbi.nlm.nih.gov/articles/PMC3791368/)
- [The Hippocampus and Episodic Memory: Place Cells, Time Cells and Memory Binding](https://www.nature.com/articles/nrn.2017.69)
- [Hippoformer: Integrating Hippocampus-inspired Spatial Memory with Transformers](https://openreview.net/forum?id=hxwV5EubAw)
- [Bio-Inspired Spatial Reasoning Transformer: Grid Cells, Place Cells, and Attractor Dynamics for Text-Based Spatial Understanding](https://openreview.net/forum?id=B48jHaYIx1)
- [Endowing Embodied Agents with Spatial Reasoning Capabilities for Vision-and-Language Navigation / BrainNav](https://arxiv.org/abs/2504.08806)
- [RoboMemory: A Brain-inspired Multi-memory Agentic Framework for Lifelong Learning in Physical Embodied Systems](https://arxiv.org/abs/2508.01415)
- [CLEA: Closed-Loop Embodied Agent for Enhancing Task Execution in Dynamic Environments](https://arxiv.org/abs/2503.00729)
- [From Reactive to Cognitive: Brain-inspired Spatial Intelligence for Embodied Agents / BSC-Nav](https://arxiv.org/abs/2508.17198)

### Visual Grounding and REC Baselines

- [TransVG: End-to-End Visual Grounding with Transformers](https://arxiv.org/abs/2104.08541)
- [SeqTR: A Simple yet Universal Network for Visual Grounding](https://arxiv.org/abs/2203.16265)
- [PolyFormer: Referring Image Segmentation as Sequential Polygon Generation](https://arxiv.org/abs/2302.07387)
- [Grounding DINO: Marrying DINO with Grounded Pre-Training for Open-Set Object Detection](https://arxiv.org/abs/2303.05499)
- [AerialREC: UAV REC Baseline Framework in SkyFind](https://github.com/wangkunyu241/SkyFind)
- [QRNet: Query Regression Network for Visual Grounding](https://ieeexplore.ieee.org/document/10113813)
- [SimREC: A Simple Framework for Referring Expression Comprehension](https://arxiv.org/abs/2301.09271)

### Remote Sensing and Aerial Grounding

- [SATGround: A Spatially-Aware Approach for Visual Grounding in Remote Sensing](https://arxiv.org/abs/2512.08881)
- [MB-ORES: Multi-Branch Object Reasoner for Remote Sensing Visual Grounding](https://arxiv.org/abs/2503.24219)
- [GroundingSuite: Measuring Complex Multi-Granular Pixel Grounding](https://arxiv.org/abs/2503.10596)
- [DViN: Dynamic Visual Routing Network for Weakly Supervised Referring Expression Comprehension](https://cvpr.thecvf.com/virtual/2025/poster/32531)
- [TCRT: Task-aware Cross-modal Feature Refinement Transformer with LLMs for Visual Grounding](https://cvpr.thecvf.com/virtual/2025/poster/33672)
- [Ground-V: Teaching VLMs to Ground Complex Instructions in Pixels](https://arxiv.org/abs/2505.13788)
- [OmniEarth: A Benchmark for Evaluating VLMs in Geospatial Tasks](https://arxiv.org/abs/2603.09471)
- [AeroReformer: Aerial Referring Transformer for UAV-based Referring Image Segmentation](https://lironui.github.io/Files/AeroReformer.pdf)

### Localization, Scene Graphs, and Memory Representation

- [VLM-Loc: Localization in Point Cloud Maps via Vision-Language Models](https://arxiv.org/abs/2603.09826)
- [SFCo-Nav: Efficient Zero-Shot Visual Language Navigation via Collaboration of Slow LLM and Fast Attributed Graph Alignment](https://arxiv.org/abs/2603.01477)
- [MapNav: A Novel Memory Representation via Annotated Semantic Maps for VLM-based Vision-and-Language Navigation](https://arxiv.org/abs/2502.13451)
- [Mem2Ego: Empowering Vision-Language Models with Global-to-Ego Memory for Long-Horizon Embodied Navigation](https://arxiv.org/abs/2502.14254)
- [CityNavAgent: Aerial Vision-and-Language Navigation with Hierarchical Semantic Planning and Global Memory](https://aclanthology.org/2025.acl-long.1511/)



### Embodied Agent and PGR-related Works Discussed

- [FlightGPT: Towards Generalizable and Interpretable UAV Vision-and-Language Navigation with Vision-Language Models](https://aclanthology.org/2025.emnlp-main.338.pdf)
- [OpenVLN: Open-world Aerial Vision-Language Navigation](https://arxiv.org/abs/2511.06182)
- [AerialVLA: A Vision-Language-Action Model for UAV Navigation via Minimalist End-to-End Control](https://arxiv.org/abs/2603.14363)
- [OpenFly: A Versatile Toolchain and Large-scale Benchmark for Aerial Vision-Language Navigation](https://arxiv.org/abs/2502.18041)
- [AdaDrive: Self-Adaptive Slow-Fast System for Language-Grounded Autonomous Driving](https://arxiv.org/abs/2511.06253)
- [Critic-V: VLM Critics Help Catch VLM Errors in Multimodal Reasoning](https://arxiv.org/abs/2411.18203)
