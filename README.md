<a id="top"></a>

<p align="center">
  <img src="assets/main.png" width="100%" alt="MiniMax-H3 acceleration research map: fewer evaluations, cheaper computation, and smaller memory footprint across the audio-video generation pipeline." />
</p>

<h1 align="center">Awesome MiniMax-H3 Acceleration</h1>

<p align="center">
  <strong>A curated index of MiniMax-H3 acceleration methods, implementations, and related resources.</strong><br />
  Methods, papers, checkpoints, and systems for researchers and inference engineers.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Focus-Efficient_Inference-0f766e?style=flat-square" alt="Focus: efficient inference" />
  <img src="https://img.shields.io/badge/Scope-Audio%20%2B%20Video-475569?style=flat-square" alt="Scope: audio and video" />
  <img src="https://img.shields.io/badge/Updated-2026--10--07-6366f1?style=flat-square" alt="Updated: 2026-10-07" />
  <a href="#contributing"><img src="https://img.shields.io/badge/Contributions-Welcome-0f766e?style=flat-square" alt="Contributions welcome" /></a>
</p>

<p align="center">
  <a href="#scope">Scope</a> ·
  <a href="#overview">Overview</a> ·
  <a href="#methods">Methods</a> ·
  <a href="#benchmarks">Benchmarks</a> ·
  <a href="#related">Related Resources</a> ·
  <a href="#contributing">Contributing</a>
</p>

---

> [!NOTE]
> This is an independent index of publicly available papers, models, implementations, and evaluations. Inclusion requires an explicit MiniMax-H3 connection in a paper, model card, implementation, or experiment. Entries describe public releases; inclusion does not imply independent reproduction or production readiness.

<a id="scope"></a>
## <img src="https://img.shields.io/badge/01-0f766e?style=flat-square" height="22" alt="" />&nbsp; Scope & Baseline

This collection covers inference acceleration for **MiniMax-H3**, including optimization of the original model, few-step adapters, compressed or architecturally modified variants, and serving systems. It prioritizes the mechanism, release provenance, and validated task scope of each work.

**Baseline resources:** [official repository](https://github.com/MiniMax-AI/MiniMax-H3) · [original checkpoints](https://huggingface.co/MiniMaxAI/MiniMax-H3) · [ComfyUI checkpoint distribution](https://huggingface.co/Comfy-Org/MiniMax-H3).

H3 processes multimodal inputs in a shared token sequence and jointly denoises video and audio. Its released base checkpoints already incorporate CFG distillation. The original Omni-Transformer includes AdaLN branches whose modulation outputs can be precomputed for inference. The methods below target attention computation, denoiser evaluations, modulation computation, and inference scheduling. See the [official architecture description](https://github.com/MiniMax-AI/MiniMax-H3#model-architecture).

| Term | Meaning in this index |
| :--- | :--- |
| **T2VA** | Text-to-video generation with audio. |
| **FL2VA** | First-/last-frame-conditioned audio-video generation; the base FL2VA checkpoint also supports text-only input. |
| **Ref2VA** | Generation conditioned on multimodal references. |
| **NFE** | Number of denoiser function evaluations; distinguish this from scheduler grid points and per-chunk step counts. |
| **H3-derived model** | A model trained from or built on H3 with altered weights, architecture, or generation protocol. |

**Curation boundary.** General diffusion methods appear here only when H3-specific evidence is available. Format conversions and UI integrations are attributed to their upstream method. Generic deployment interfaces, aesthetic LoRAs, and unrelated video models are outside the main taxonomy.

<a id="overview"></a>
## <img src="https://img.shields.io/badge/02-6366f1?style=flat-square" height="22" alt="" />&nbsp; Research Overview

| Research direction | Optimization target | Representative work |
| :--- | :--- | :--- |
| [Few-step generation & distillation](#distillation) | Denoiser evaluation count and learned sampling trajectories | LightX2V Turbo, FastH3, PDD, HyperFlow, DMAD, LynnReal-Omni |
| [Efficient attention & architecture](#attention) | Attention computation and model structure | Sol-Attn, Turbo-SLA, Veda, VDN-H3, MC-Sparse, VC-Attention |
| [Caching & feature prediction](#caching) | Feature reuse and prediction across denoising steps | Spectrum, FirstBlockCache, MotionCache, TE-Speed, AdaTaylorCache |
| [Quantization & compression](#compression) | Weight/activation precision, modulation parameters, memory traffic | ConvRot, NF4, GGUF, OrbitQuant, SVDQuant, AdaLN precomputation |
| [Kernels, runtimes & distributed inference](#systems) | Execution efficiency, communication, and component placement/offloading | SGLang, vLLM-Omni, LightX2V, H3-specific runtimes |
| [VAE & decoder acceleration](#vae) | VAE decode cost and reconstruction quality versus decoding speed | Light VAE, H3-TAE, MotionCache Fast VAE Decode |
| [Autoregressive diffusion acceleration](#ar-diffusion) | Chunked autoregressive generation and cross-chunk memory reuse for streaming | TaoMate-H3 |

Entries are organized by technical contribution. A project may appear in multiple categories when it contributes more than one acceleration mechanism; ports and quantized exports retain their upstream attribution.

<a id="methods"></a>
## <img src="https://img.shields.io/badge/03-0f766e?style=flat-square" height="22" alt="" />&nbsp; Methods & Implementations

<a id="distillation"></a>
### 3.1 Few-Step Generation & Distillation

Adapters and H3-derived models trained for generation with fewer denoiser evaluations.

| Work | Technical contribution | Task coverage / notes | Resources |
| :--- | :--- | :--- | :--- |
| **LightX2V / MiniMax-H3-Turbo** | Few-step Turbo adapters with checkpoint-specific sampling schedules. | FL2VA/T2VA and Ref2VA releases; task coverage, shifts, and resolution differ by checkpoint. | [Code](https://github.com/ModelTC/Minimax-H3-Turbo) · [Weights](https://huggingface.co/lightx2v/Minimax-h3-Turbo) |
| **LarryVRH H3 Turbo** | Community few-step LoRA with a dedicated H3 sampler and loader. | T2V/I2V workflows; runtime handling for unpruned and AdaLN-pruned base checkpoints. Keep this family separate from LightX2V Turbo. | [Code](https://github.com/larryvrh/ComfyUI-MiniMax-H3-Turbo) · [Weights](https://huggingface.co/larryvrh/MiniMax-H3-Turbo-Lora) |
| **FastH3 / FastVideo** | Data-free distillation combined with video sparse attention. | Multiple releases; the cited 8-Step V2 checkpoint targets T2VA and requires VSA-H3. FL2VA and Ref2VA were not distilled in that release. | [Code](https://github.com/hao-ai-lab/FastVideo) · [8-Step V2](https://huggingface.co/FastVideo/FastVideo-FastH3-8-Step-V2) · [Collection](https://huggingface.co/collections/FastVideo/fastvideo-fasth3) |
| **PDD H3 Acc-LoRAs / Alibaba PAI** | Application of Parallel Decoding Distillation to H3, with a backbone adapter and interval-specific output heads. | Separate FL2VA and Ref2VA checkpoints. Requires PDD-aware loading rather than ordinary LoRA loading alone. | [PDD paper](https://arxiv.org/abs/2607.26004) · [Code](https://github.com/aigc-apps/VideoX-Fun) · [H3 weights](https://huggingface.co/alibaba-pai/MiniMax-H3-Acc-LoRAs) |
| **HyperFlow / Video Rebirth** | Data-free flow self-distillation released as an eight-step adapter. | Official Diffusers Modular Pipeline; T2VA, FL2VA, and Ref2VA workflows retain joint audio-video generation. | [Code](https://github.com/Video-Rebirth/hyperflow) · [Weights](https://huggingface.co/videorebirth/hyperflow) |
| **DMAD** | Distribution matching formulated as adversarial distillation. | Four-step H3 audio-video student; distinguish author-supported inference from downstream conversions. | [Paper](https://arxiv.org/abs/2610.02188) · [Project / resources](https://yzmblog.github.io/projects/DMAD/) |
| **PDMD** | Public two- and four-NFE H3 LoRA releases. | Weight releases indexed; training provenance, full task coverage, and reproducibility evidence need further review. | [2-NFE weights](https://huggingface.co/pdmd2026/pdmd_2NFE_lora) · [4-NFE weights](https://huggingface.co/pdmd2026/pdmd_4NFE_lora) |
| **LynnReal-Omni few-step generation** | Four-step Standard and three-step Flash generation. | H3-derived checkpoints with variant-specific task coverage and sampling schedules. | [Paper](https://arxiv.org/abs/2609.15863) · [Code / model links](https://github.com/LynnReal-AI/LynnReal-Omni) |

<a id="attention"></a>
### 3.2 Efficient Attention & Architecture Modification

These works reduce attention computation or change its structure. Training-free approximations, trained sparse models, and hybrid architectures represent different experimental settings.

| Work | Technical contribution | H3-specific evidence / qualification | Resources |
| :--- | :--- | :--- | :--- |
| **Sol Engine / Sol-Attn** | Training-free sparse attention integrated into Sol Engine. | H3 deployment reports cover attention, runtime, and pipeline optimizations; end-to-end speedups should not be attributed to attention alone. | [H3 system report](https://nvlabs.github.io/Sana/Sol-Engine/H3-DataCenter/) · [Triton port](https://github.com/kijai/ComfyUI-SolAttn_triton) · [CuTe DSL port](https://github.com/quzopl/ComfyUI-SolAttn-H3) |
| **LightX2V Turbo-SLA** | Joint few-step distillation and Sparse–Linear Attention adaptation. | A dedicated sparse FL2V adapter, not merely a generic attention switch applied to any Turbo checkpoint. | [H3 weights / recipe](https://huggingface.co/lightx2v/Minimax-h3-Turbo-SLA) · [SLA](https://github.com/thu-ml/SLA) |
| **Veda / Miowtion** | Learned block selection for video sparse attention, with predictor training and LoRA quality recovery. | H3 implementation supports FL2VA and Ref2VA paths; released predictor scope must be checked separately. | [Code](https://github.com/veda-sparse/Miowtion) · [T2VA preview](https://huggingface.co/Veda-Sparse/Minimax-H3-T2VA-Veda-8NFE-600Step-Preview) |
| **Video DeltaNet / VDN-H3** | Video-native hybrid attention, retaining Softmax for interactions involving text or audio. | Architectural adaptation combined with few-step distillation and a serving stack with additional runtime optimizations. | [Paper](https://arxiv.org/abs/2609.20744) · [Project / resources](https://openvdn.github.io/) |
| **LynnReal-Omni-Flash architecture** | A smaller, 42-block transformer with spatial token selection. | Structural and token-level computation reduction in the Flash variant. | [Paper](https://arxiv.org/abs/2609.15863) · [Code](https://github.com/LynnReal-AI/LynnReal-Omni) |
| **MC-Sparse** | Training-free sparse attention with cached query groups, KV selections, and dense–sparse residuals. | Paper includes MiniMax-H3-Base evaluation; research-only entry with no public implementation listed. | [Paper](https://arxiv.org/abs/2610.06801) |
| **VC-Attention** | Value smoothing and fused probability casting for low-bit attention. | H3 is included among the evaluated video models; kernel-level and complete-generation effects should be distinguished. | [Paper](https://arxiv.org/abs/2609.15810) |
| **H3-Optimizations** | H3-specific sparse scheduling, token ordering, and memory optimization. | Source implementation with configurable backends; includes an experimental AMD path. | [Code](https://github.com/Zironic/H3-Optimizations) |
| **X-MinimaxH3** | Single-GPU H3 execution with configurable sparse acceleration across sampling stages. | Single-GPU implementation for consumer hardware, covering joint attention and pipeline scheduling. | [Code](https://github.com/PullMyBoots/X-MinimaxH3) |

**Backend implementations.** SageAttention and Comfy Kitchen also appear in H3 execution paths. Track the actual H3 integration and hardware requirements rather than treating every generic attention library as an independent H3 method; see the [SGLang H3 cookbook](https://docs.sglang.io/cookbook/diffusion/MiniMax/MiniMax-H3) and [H3 Colab comparison harness](https://github.com/soren-labs/minimax-h3-colab).

<a id="caching"></a>
### 3.3 Caching & Feature Prediction

Training-free caching and feature prediction reduce denoiser computation by reusing or approximating intermediate features or model outputs across sampling steps. Block-level reuse does not necessarily reduce NFE. These approximations can change the sampling trajectory, so both video and audio quality require evaluation.

| Work | Reuse / prediction mechanism | Implementation note | Source |
| :--- | :--- | :--- | :--- |
| **Spectrum H3** | Spectral prediction of post-transformer hidden features. | H3-specific implementation with audio-aware replay handling; approximate execution. | [Code](https://github.com/xmarre/ComfyUI-Spectrum-MiniMax-H3) |
| **FirstBlockCache H3** | First-block residual change determines reuse of later computation. | Dedicated ComfyUI node for H3. | [Code](https://github.com/duckyshell/ComfyUI-MiniMaxH3-FirstBlockCache) |
| **TE-Speed** | Block-cache acceleration. | The original release and OSS reimplementation are separate distributions with different version and compatibility requirements. | [Original](https://github.com/tl2012tl/TE-Speed-MiniMaxH3) · [OSS implementation](https://github.com/HELPMEEADICE/TE-Speed-MiniMaxH3-OSS) |
| **MotionCache** | Reuse of joint video/audio residuals based on motion-weighted change. | Also includes an experimental batched video VAE decoder. | [Code](https://github.com/starsFriday/ComfyUI-MiniMax-H3-MotionCache) |
| **TeaCache H3** | Threshold-based selection between cache reuse and recomputation. | H3-specific node; evaluation coverage has not been verified for this index. | [Code](https://github.com/Icyoung/ComfyUI-MiniMaxH3-TeaCache) |
| **TeleFuser / AdaTaylorCache** | Feature prediction calibrated for the joint DiT stack. | H3 calibration uses audio-token residuals to avoid masking audio error with the larger video sequence. | [H3 cookbook](https://tele-ai.github.io/TeleFuser/cookbook/minimax-h3/) |
| **Cache-DiT and velocity-cache integrations** | Block caching or reuse and extrapolation of predicted velocities across sampling steps. | Framework integrations with calibrated profiles or manual cache settings; their supported configurations differ. | [SGLang](https://docs.sglang.io/cookbook/diffusion/MiniMax/MiniMax-H3) · [RunningHub implementation](https://github.com/HM-RunningHub/ComfyUI_RH_MinMaxH3/blob/main/docs/sampling.md) |

<a id="compression"></a>
### 3.4 Quantization & Model Compression

This category includes both arithmetic optimization and memory-oriented representations. Checkpoint size, runtime memory, and generation latency are different quantities.

| Work / release family | Optimization | Interpretation and provenance | Source |
| :--- | :--- | :--- | :--- |
| **AdaLN precomputation / pruned H3** | Precompute time-dependent modulation and omit the corresponding parameter branches during inference. | Architectural opportunity described by H3; distinguish this from generic Transformer layer pruning. | [Architecture](https://github.com/MiniMax-AI/MiniMax-H3#model-architecture) · [Comfy-Org releases](https://huggingface.co/Comfy-Org/MiniMax-H3) |
| **ConvRot / mixed low-bit exports** | Quantized H3 checkpoints with different precision and representation choices. | Release families rather than a single uniform recipe; verify unpruned/pruned variants and task compatibility per file. | [Comfy-Org](https://huggingface.co/Comfy-Org/MiniMax-H3) · [Kijai experiments](https://huggingface.co/Kijai/MiniMax-H3-experimental) |
| **DiffSynth-Studio H3-NF4** | NF4 weights for memory-constrained execution. | Primarily a low-memory deployment path; latency depends on kernels and offloading. | [Weights / documentation](https://huggingface.co/DiffSynth-Studio/MiniMax-H3-NF4) |
| **OrbitQuant W4A4** | Low-bit weights and activations. | H3-specific quantized release with its own execution requirements. | [Model card](https://huggingface.co/WaveCut/MiniMax-H3-OrbitQuant-W4A4) |
| **SVDQuant FP4 + PDD8** | Quantization integrated with an accelerated H3 adapter. | Community H3/Nunchaku fork; do not assume upstream Nunchaku support. | [Artifacts / fork instructions](https://huggingface.co/1ronman1993/MiniMax-H3-SVDQuant-fp4-pdd8) |
| **GGUF H3** | Quantized formats and associated runtime support. | Group conversions by provenance; different exporters and unpruned/pruned variants are not interchangeable. | [leejet](https://huggingface.co/leejet/MiniMax-H3-GGUF) · [joeygambino](https://huggingface.co/joeygambino/MiniMax-H3-GGUF) |
| **LynnReal-Omni INT8** | INT8 projection weights and activations. | Flash uses a trained W8A8 export; Standard also provides an INT8 execution path. | [Implementation / exports](https://github.com/LynnReal-AI/LynnReal-Omni/blob/main/comfyui/README.md) |
| **LynnReal-Omni Lite precomputation** | Schedule-specific time-modulation tables reduce the checkpoint footprint. | Standard Lite and Flash Lite preserve their corresponding variants' tested outputs at the shipped schedules. | [Documentation](https://github.com/LynnReal-AI/LynnReal-Omni/blob/main/comfyui/README.md) |

<a id="systems"></a>
### 3.5 Kernels, Runtimes & Distributed Inference

System entries identify H3-specific execution work. Their performance depends on the selected model, attention backend, precision, parallel layout, and component placement or offloading policy.

| System | H3-specific contribution | Source |
| :--- | :--- | :--- |
| **SGLang** | Distributed execution, encoder scheduling, attention backends, quantization, and caching recipes. | [H3 cookbook](https://docs.sglang.io/cookbook/diffusion/MiniMax/MiniMax-H3) |
| **vLLM-Omni** | Sequence parallelism, text-encoder tensor parallelism, VAE patch parallelism, and offloading. | [H3 recipe](https://recipes.vllm.ai/MiniMaxAI/MiniMax-H3) |
| **LightX2V** | H3-specific combinations of few-step inference, quantization, optimized operators, AdaLN caching, and offloading. | [H3 configurations](https://github.com/ModelTC/LightX2V/tree/main/configs/minimax_h3) |
| **h3.c** | Native H3 inference engine for Apple Silicon. | [Code](https://github.com/antirez/h3.c) |
| **MiniMax-H3 MLX / PipeNetwork** | MLX port with AdaLN precomputation and quantized checkpoints. | [Code](https://github.com/PipeNetwork/minimax-h3-mlx) |
| **MiniMax-H3 Swift** | Swift/MLX implementation and documented investigations of caching and kernel tradeoffs. | [Code](https://github.com/loading-awesome/MiniMax-H3-Swift) · [Engineering notes](https://github.com/loading-awesome/MiniMax-H3-Swift/blob/main/docs/PERFORMANCE_GUIDE.md) |
| **H3 ComfyUI Acceleration Pack / AMD ROCm** | Integration and reproducible comparison tooling for Turbo, Spectrum, and FirstBlockCache on ROCm. | [Code](https://github.com/JH427/minimax-h3-comfyui-acceleration) |

<a id="vae"></a>
### 3.6 VAE & Decoder Acceleration

These methods reduce VAE decoding cost or trade reconstruction quality for decoding speed; they do not reduce H3 denoiser cost.

| Work | Technical contribution | Scope / notes | Resources |
| :--- | :--- | :--- | :--- |
| **LynnReal Light VAE** | Lightweight video decoder used in the Flash pipeline. | Decoder optimization for H3-derived audio-video generation. | [Code](https://github.com/LynnReal-AI/LynnReal-Omni) · [Weights](https://huggingface.co/stdstu123/LynnReal-Onmi-light-vae) |
| **H3-TAE** | Lightweight approximate decoding. | Evaluate reconstruction and temporal fidelity separately from denoiser behavior. | [Weights](https://huggingface.co/Kijai/MiniMax-H3-TAE) |
| **MotionCache Fast VAE Decode** | Experimental batched decoding in an H3-specific workflow. | Batched VAE decoding; separate from MotionCache's denoiser cache. | [Code](https://github.com/starsFriday/ComfyUI-MiniMax-H3-MotionCache) |

<a id="ar-diffusion"></a>
### 3.7 Autoregressive Diffusion Acceleration

Chunked autoregressive generation reuses cross-chunk state to produce continuous audio-video streams. This setting differs from full-sequence denoising and VAE-side acceleration.

| Work | Technical contribution | Scope / notes | Resources |
| :--- | :--- | :--- | :--- |
| **TaoMate-H3** | Few-step chunked autoregressive generation with cross-chunk memory reuse for continuous audio-video streaming. | Released T2VA path; do not infer release of FL2VA or Ref2VA from roadmap entries. | [Code / weights](https://github.com/TaoLiveAIGC/TaoMate-H3) |

<a id="benchmarks"></a>
## <img src="https://img.shields.io/badge/04-6366f1?style=flat-square" height="22" alt="" />&nbsp; Benchmark Resources

External evaluations are linked with brief descriptions only. Their protocols, measurements, rankings, and sample outputs remain at the original sources.

| Resource | What it provides |
| :--- | :--- |
| [**MiniMax H3 speed-ups, measured — plox-1**](https://plox-1.github.io/h3-speedups/#rank=quality) | Interactive comparison of acceleration methods and combinations, with generated clips and baseline comparisons. Still-frame quality ratings should be distinguished from motion and audio evaluation. |
| [**H3 Turbo Evaluation — A/B viewer**](https://jo-nike.github.io/h3-turbo-eval/ab.html) | A paired visual comparison interface for inspecting H3 outputs under alternative configurations. |
| [**MiniMax H3 Colab comparison harness**](https://github.com/soren-labs/minimax-h3-colab) | Reproducible Native, Spectrum, and TE-Speed experiments with pinned revisions, sample outputs, and compatibility records. |
| [**AMD ROCm reference campaign**](https://github.com/JH427/minimax-h3-comfyui-acceleration) | Reference workflows and benchmark tooling for H3 acceleration on an AMD system. |

Author-run evaluation and reproduction instructions are also available in the linked method repositories. These resources use different workloads and protocols and are not combined into a unified leaderboard here.

<a id="related"></a>
## <img src="https://img.shields.io/badge/05-0f766e?style=flat-square" height="22" alt="" />&nbsp; Related Resources

| Resource | Relation to this index |
| :--- | :--- |
| [MiniMax-H3](https://github.com/MiniMax-AI/MiniMax-H3) | Original model architecture and deployment references. |
| [MiniMax H3 Integrations](https://github.com/MiniMax-AI/awesome-minimax-h3-integration) | Collection of H3 checkpoints, tools, and integrations. |
| [Awesome MiniMax H3 / AtlasCloudAI](https://github.com/AtlasCloudAI/awesome-minimax-h3) | Community models, workflows, and related resources. |
| [Awesome Efficient Diffusion](https://github.com/AI-Efficiency/Awesome-Efficient-Diffusion) | General literature on efficient diffusion and flow-matching models. |
| [Awesome Video Diffusion](https://github.com/showlab/Awesome-Video-Diffusion) | Broader video diffusion research. |

<a id="contributing"></a>
## <img src="https://img.shields.io/badge/06-6366f1?style=flat-square" height="22" alt="" />&nbsp; Contributing

To propose a paper, project, benchmark, or correction, open an issue or pull request.

**Acknowledgments.** Credit belongs to the authors of the linked papers, models, implementations, and evaluations. This index is not affiliated with MiniMax or any listed project. Linked code and model weights are subject to their respective licenses.

<p align="center"><a href="#top">Back to top ↑</a></p>
