# NVIDIA AlpaDreams 한국어 정리

> 2026년 GTC26 에서 처음 공개된 NVIDIA의 자율주행용 **action-conditioned 생성형 world model** 프로젝트. 본 문서는 공식 프로젝트 페이지, NVIDIA DRIVE / 연구진 SNS, 동반 릴리스(Alpamayo, AlpaSim) 의 공개 정보만을 토대로 작성했습니다. 코드/논문이 공개되기 전까지는 **정보가 계속 보강되어야 하는 문서** 이므로, 잘못되었거나 업데이트가 필요한 항목은 이슈/PR 로 알려주세요.

문서 기준일: **2026-04-16**

---

## 목차
1. [한눈 요약](#한눈-요약)
2. [AlpaDreams 는 무엇인가](#alpadreams-는-무엇인가)
3. [릴리스 상태 (코드/논문/데이터)](#릴리스-상태-코드논문데이터)
4. [기술적 포지셔닝](#기술적-포지셔닝)
5. [Alpamayo / AlpaSim / AlpaDreams 삼각 구도](#alpamayo--alpasim--alpadreams-삼각-구도)
6. [직접적인 선행 연구: Cosmos-Drive-Dreams](#직접적인-선행-연구-cosmos-drive-dreams)
7. [기반 플랫폼: NVIDIA Cosmos](#기반-플랫폼-nvidia-cosmos)
8. [알아두면 좋은 관련 라이브러리 · 논문](#알아두면-좋은-관련-라이브러리--논문)
9. [실무자가 지금 당장 할 수 있는 것](#실무자가-지금-당장-할-수-있는-것)
10. [참고 자료 (출처)](#참고-자료-출처)

---

## 한눈 요약

| 항목 | 내용 |
|------|------|
| **정식 명칭** | AlpaDreams |
| **개발 주체** | NVIDIA SIL (Scalable Intelligent Learning) / NVIDIA DRIVE · Toronto AI Lab |
| **최초 공개** | NVIDIA GTC26 (2026년 봄) — 부스 데모 형태 |
| **한 줄 설명** | "Action-conditioned generative world model for closed-loop AV simulation" (NVIDIA DRIVE 공식 설명) |
| **기반** | NVIDIA Cosmos (World Foundation Model) |
| **형제 프로젝트** | **Alpamayo** (VLA 정책 모델), **AlpaSim** (오픈소스 시뮬레이터) |
| **선행 연구** | **Cosmos-Drive-Dreams** (arXiv 2506.09042, CVPR 2025 계열) |
| **코드 공개** | ❌ 2026-04-16 기준 **미공개** (프로젝트 페이지/데모만 존재) |
| **논문 공개** | ❌ 확인 불가 (arXiv 미등재, 공식 기술 블로그에 본문 기술 상세 없음) |
| **대표 연구자** | Xuanchi Ren (NVIDIA) — Cosmos-Drive-Dreams 리드, 본인 X 계정에서 AlpaDreams 소개 |
| **공식 페이지** | https://research.nvidia.com/labs/sil/projects/alpadreams/ |

> ✨ **핵심 차별점**: 기존 AV 시뮬레이터가 과거를 "재생/재구성" 하는 데 초점이 있다면, AlpaDreams 는 **"다음에 무엇이 일어날지" 를 action-conditioned 로 생성** 하여 정책(policy)과 닫힌 루프(closed-loop) 로 상호작용할 수 있게 합니다.

---

## AlpaDreams 는 무엇인가

### 공식 설명 원문 인용

NVIDIA DRIVE 공식 채널의 소개 문장:

> **"What if AV simulation didn't just replay or reconstruct the past—but could generate what happens next? Introducing NVIDIA AlpaDreams, our latest research: an action-conditioned generative world model for closed-loop AV simulation. Combined with Alpamayo and AlpaSim, it enables ..."**
> — [NVIDIA DRIVE @ X](https://x.com/NVIDIADRIVE/status/2034622341578133539)

연구자 본인(Xuanchi Ren) 소개:

> **"Last GTC we presented Cosmos-Drive-Dreams. At #GTC26, we're introducing AlpaDreams — an interactive closed-loop world model for autonomous driving. Come try it at the booth!"**
> — [Xuanchi Ren @ X](https://x.com/xuanchi13/status/2033702698248274036)

### 알려진 동작 원리 (공개된 수준에서)

1. **World Model 이 환경을 실시간 생성** — NVIDIA Cosmos 기반 video/world foundation model 을 백본으로 사용해 도로·차량·날씨를 토큰 단위로 예측.
2. **Policy 또는 운전자가 상호작용** — 정책이 던진 액션(조향·가속·감속)을 조건(condition)으로 받아 다음 프레임 world state 가 갱신.
3. **시뮬레이터가 월드를 업데이트** — 일반적인 replay 기반 시뮬레이터는 "녹화된 과거" 만 재생하지만, AlpaDreams 는 **generative** 라서 새로운 시나리오를 만들어낼 수 있음.
4. **Closed-loop 평가** — 정책의 반응이 다음 state 분포를 바꾸고, 바뀐 state 가 다시 정책을 자극하는 피드백 루프 완성.

이 흐름은 Waymo World Model, Wayve GAIA-2 등 동시대 다른 AV world model 들과 철학이 유사하나, **NVIDIA Cosmos** 와의 통합과 Alpamayo/AlpaSim 생태계 묶음 공급이라는 포인트가 차별점입니다.

---

## 릴리스 상태 (코드/논문/데이터)

### 2026-04-16 시점에서 **확인된 사실만** 정리하면:

| 아이템 | 상태 | 확인 출처 |
|--------|------|-----------|
| 공식 프로젝트 페이지 | ✅ 존재 (제목만 확인, 본문 확장 중) | research.nvidia.com/labs/sil/projects/alpadreams |
| 데모 | ✅ GTC26 부스에서 실시간 인터랙션 데모 | Xuanchi Ren / NVIDIA DRIVE |
| 백서/기술보고서 | ❌ 공개 버전 없음 | NVIDIA 공식 블로그/뉴스룸 전수 확인 |
| arXiv 논문 | ❌ 등재 미확인 | arxiv.org 검색 결과 없음 |
| GitHub 저장소 | ❌ NVlabs/NVIDIA 계열에 별도 repo 없음 | github.com/NVlabs, github.com/orgs/NVIDIA |
| 모델 weights | ❌ Hugging Face 미등재 | huggingface.co/NVIDIA 정책 가중치는 Alpamayo 에만 존재 |
| 오픈소스 라이선스 | ❌ 알 수 없음 | — |

### 관련 릴리스는 **주변 프로젝트**만 오픈:

- **Alpamayo 1 / 1.5** (VLA policy): [github.com/NVlabs/alpamayo](https://github.com/NVlabs/alpamayo), HF `nvidia/Alpamayo-R1-10B`
- **AlpaSim** (시뮬레이션 플랫폼): [github.com/NVlabs/alpasim](https://github.com/NVlabs/alpasim), Apache 2.0
- **Physical AI AV Dataset**: 25개국, 1,727시간 주행 데이터 (Hugging Face)
- **Cosmos-Drive-Dreams** (AlpaDreams 의 논문판 이전작): [github.com/nv-tlabs/Cosmos-Drive-Dreams](https://github.com/nv-tlabs/Cosmos-Drive-Dreams), [arXiv 2506.09042](https://arxiv.org/abs/2506.09042)

> 📌 **요약**: "AlpaDreams 자체" 는 **리서치 데모 단계** 이며, 실체로 쓸 수 있는 코드는 **아직 없습니다**. 관련 오픈소스를 조합하려면 AlpaSim + Alpamayo + Cosmos-Drive-Dreams 세 저장소를 기반으로 삼는 것이 현재 가능한 최선의 근사치입니다.

---

## 기술적 포지셔닝

### "재생형" 시뮬레이터 vs "생성형" 시뮬레이터

```
[ 기존 AV 시뮬레이션 ]
  녹화된 센서 로그   ──▶   재생    ──▶   정책 평가
  (log-replay)             (deterministic)
  ❌ 정책이 다르게 반응해도 월드는 안 바뀜 → 오프-폴리시 왜곡

[ Neural Reconstruction 시뮬레이터 (NeRF/3DGS 기반) ]
  실제 씬 재구성   ──▶   카메라 이동 렌더링   ──▶   정책 평가
  ✅ 시점은 자유롭지만
  ❌ "있었던 사건" 외 새로운 시나리오 생성 불가

[ AlpaDreams — Generative World Model ]
  (초기 상태) + action  ──▶   WFM이 다음 상태 생성   ──▶   정책
                                    ▲                          │
                                    └──────────────────────────┘
                                       (closed-loop)
  ✅ 정책이 달라지면 월드도 달라짐 → 진짜 closed-loop 평가
  ✅ 희귀 시나리오(눈, 안개, 보행자 난입)를 새로 생성 가능
```

### 결과적으로 노리는 효과

1. **Sim-to-Real 갭 감소** — 실제 로그 기반 photorealism 을 유지하면서 시나리오 다양성도 확보.
2. **Long-tail 커버** — Cosmos 계열 world model 은 이미 희귀 시나리오 생성에 강점이 있음 (Cosmos-Drive-Dreams 에서 증명).
3. **정책 RL/강화학습 환경** — deterministic replay 와 달리 확률적 world model 은 on-policy 강화학습 환경으로 직접 활용 가능.

---

## Alpamayo / AlpaSim / AlpaDreams 삼각 구도

NVIDIA 는 CES26 · GTC26 에서 자율주행 AI 를 **세 개의 축** 으로 묶어 공개했습니다.

```
┌─────────────────────────────────────────────────────────┐
│                      Alpamayo                           │
│   Vision-Language-Action 정책 모델 (Level 4 지향)        │
│   chain-of-causation reasoning, 6.4s 미래 궤적 예측       │
│   ▶ "차가 어떻게 사고하고 움직일지"                     │
└─────────────────────────────────────────────────────────┘
                          │ action (조향·가속)
                          ▼
┌─────────────────────────────────────────────────────────┐
│                    AlpaDreams  (★ 이 문서)              │
│   Action-conditioned generative world model             │
│   ▶ "월드가 그 액션에 어떻게 반응할지"                   │
└─────────────────────────────────────────────────────────┘
                          │ next state
                          ▼
┌─────────────────────────────────────────────────────────┐
│                       AlpaSim                           │
│   오픈소스 closed-loop 시뮬레이션 플랫폼                 │
│   gRPC 마이크로서비스, 센서 모델링, 교통 시나리오         │
│   ▶ "전체 파이프라인을 어떻게 실행/평가할지"             │
└─────────────────────────────────────────────────────────┘
```

| | Alpamayo | AlpaSim | AlpaDreams |
|---|----------|---------|------------|
| 역할 | 정책(Policy) | 시뮬레이터 쉘 | World model |
| 입력 | 멀티뷰 비디오 + ego history | 센서 · 시나리오 스펙 | 초기 state + action |
| 출력 | 추론 trace + 6.4s trajectory | 물리·센서 시뮬레이션 결과 | 생성된 다음 state (비디오/월드) |
| 오픈소스 | ✅ Apache 2.0 (inference code) / 모델은 비상업 | ✅ Apache 2.0 | ❌ 미공개 |
| 규모 | 10B 파라미터 | N/A | 미공개 |
| 핵심 베이스 | Cosmos-Reason | 자체 + 외부 policy 연동 | NVIDIA Cosmos WFM |

---

## 직접적인 선행 연구: Cosmos-Drive-Dreams

AlpaDreams 는 이름부터 **Cosmos-Drive-Dreams** 의 직계 후속작임을 드러냅니다. 두 프로젝트의 같은 리드 연구자(Xuanchi Ren) 가 "지난 GTC 에는 Cosmos-Drive-Dreams, 이번 GTC 에는 AlpaDreams" 라고 명시적으로 비교한 사실이 이를 뒷받침합니다.

| 항목 | Cosmos-Drive-Dreams | AlpaDreams |
|------|---------------------|------------|
| 목적 | **오프라인** synthetic 데이터 생성 | **온라인** closed-loop 시뮬레이션 |
| 조건 입력 | HDMap, 3D cuboid, 텍스트, LiDAR depth | Action (정책의 조향·가속) |
| 출력 | 긴 합성 비디오 클립 (121 frame × 10초) | 실시간으로 갱신되는 world state |
| 상호작용 | ❌ 단방향 생성 | ✅ 정책-월드 피드백 |
| 릴리스 | ✅ 모델, 파이프라인, 데이터 오픈 | ❌ 아직 비공개 |
| arXiv | [2506.09042](https://arxiv.org/abs/2506.09042) | — |
| GitHub | [nv-tlabs/Cosmos-Drive-Dreams](https://github.com/nv-tlabs/Cosmos-Drive-Dreams) | — |

### Cosmos-Drive-Dreams 핵심 성과 (참고)
- HDMap/BBox/LiDAR 레이블과 함께 **5,843 × 10초 클립, 81,802 합성 비디오** 를 생성.
- 악천후(눈/비/안개) 등 **long-tail 시나리오** 를 만들어 3D lane detection, 3D object detection, 정책 학습의 일반화 성능 향상.
- 단일 시점에서 **6개 view consistent 비디오** 생성 지원.

AlpaDreams 가 공식 논문을 내면 이 베이스에서 "(1) 시간적 일관성 확장, (2) action 조건화, (3) 실시간화" 세 축으로 확장된 형태일 가능성이 높습니다 (공개 정보 기반 추론).

---

## 기반 플랫폼: NVIDIA Cosmos

AlpaDreams 의 백본이 되는 **NVIDIA Cosmos** 는 "Physical AI" 를 위한 World Foundation Model (WFM) 스위트입니다.

| 구성요소 | 역할 |
|----------|------|
| **Cosmos Tokenizer** | 비디오 → 연속 잠재 공간으로 효율 압축 |
| **Cosmos Predict / Transfer** | Diffusion/Autoregressive 기반 WFM, 미래 프레임 생성 |
| **Cosmos Reason** | VLM 기반 추론 백본 (Alpamayo 의 상위 모델) |
| **Cosmos Guardrails** | 안전성 필터링 |
| **Cosmos-RL** | Post-training 용 강화학습 프레임워크 (Alpamayo 에 사용) |

2026 초 기준 Cosmos 3 가 공개되었으며, Alpamayo/AlpaSim/AlpaDreams 모두 Cosmos 스택 위에서 설계됩니다.

> 📎 참고 페이지: [NVIDIA Cosmos 공식](https://www.nvidia.com/en-us/ai/cosmos/), [Cosmos 기반 AV 개발 기술 블로그](https://developer.nvidia.com/blog/simplify-end-to-end-autonomous-vehicle-development-with-new-nvidia-cosmos-world-foundation-models/)

---

## 알아두면 좋은 관련 라이브러리 · 논문

AlpaDreams 를 이해하거나 비슷한 시스템을 자체 구현해 보려 할 때 참고할 만한 **외부/내부 레퍼런스** 모음입니다.

### A. NVIDIA 자체 생태계

| 프로젝트 | 한 줄 요약 | 링크 |
|----------|-----------|------|
| NVIDIA Cosmos | World Foundation Model 플랫폼 | [cosmos page](https://www.nvidia.com/en-us/ai/cosmos/) |
| Cosmos-Drive-Dreams | WFM 기반 합성 주행 데이터 생성 | [arXiv](https://arxiv.org/abs/2506.09042) · [GitHub](https://github.com/nv-tlabs/Cosmos-Drive-Dreams) |
| Alpamayo 1 / 1.5 | VLA 자율주행 정책 모델 | [GitHub](https://github.com/NVlabs/alpamayo) · [HF](https://huggingface.co/nvidia/Alpamayo-R1-10B) |
| AlpaSim | 오픈소스 closed-loop 시뮬레이터 | [GitHub](https://github.com/NVlabs/alpasim) |
| NVIDIA DRIVE Simulation | Omniverse 기반 센서 · 차량 시뮬 | [developer.nvidia.com/drive/simulation](https://developer.nvidia.com/drive/simulation) |
| 3D-GRUT / 3DGUT | Ray tracing · UT 기반 Gaussian 렌더러 | nv-tlabs/3dgrut |
| Neural Reconstruction for AV | 신경 재구성 + WFM 결합 기술 블로그 | [NVIDIA Blog](https://developer.nvidia.com/blog/accelerating-av-simulation-with-neural-reconstruction-and-world-foundation-models/) |

### B. 경쟁 · 비교가능한 world model (AV 도메인)

| 프로젝트 | 제공 주체 | 특징 |
|----------|-----------|------|
| **Waymo World Model** | Waymo | 2026-02 공개, closed-loop AV 시뮬레이션 |
| **GAIA-1 / GAIA-2** | Wayve | 생성형 driving world model 계열 |
| **DriveDreamer / DriveDreamer-2** | 학계 (연구 컨소시엄) | Diffusion 기반 driving world model |
| **GenAD** | 학계 | Generalized video generation for autonomous driving |
| **Vista (Drive-WM)** | 학계 | High-fidelity driving scene generation |
| **ADriver-I** | 학계 | Language-guided action-conditioned generation |

보다 망라된 목록: [Awesome-World-Model (Autonomous Driving)](https://github.com/HaoranZhuExplorer/World-Models-Autonomous-Driving-Latest-Survey)

### C. 일반 world model / 비디오 생성 기반

| 기법 | 요점 |
|------|------|
| **DreamerV3** (Hafner+ 2023) | 모델 기반 강화학습의 정석 |
| **Genie / Genie 2** (Google DeepMind) | foundation world model, 액션-컨디셔닝 |
| **VideoPoet · Sora** | 장기 비디오 생성 트렌드 |
| **Stable Video Diffusion** | 오픈소스 비디오 확산 모델 기준점 |

### D. Neural Reconstruction 계열 (AV 시뮬의 기하 백엔드)

| 기법 | 요점 |
|------|------|
| **3D Gaussian Splatting (3DGS)** | 실시간 고품질 신경 렌더링 |
| **3DGRT / 3DGUT** | 왜곡 카메라·롤링 셔터 지원, ray tracing 결합 |
| **NeRF · Instant-NGP** | implicit field 기반 재구성의 원조 |
| **Neural Harmonic Textures (NHT)** | 고주파 텍스처 재현용 NVIDIA 연구 |

---

## 실무자가 지금 당장 할 수 있는 것

AlpaDreams 자체 코드는 나오지 않았지만, 공개된 구성 요소를 조합해 "프리뷰 수준" 으로 실험해 볼 수 있습니다.

1. **AlpaSim 을 내려받아 closed-loop 골격 이해** — gRPC 마이크로서비스 구조, 정책 Slot, 센서 Slot 의 인터페이스 확인.
2. **Alpamayo 1.5 를 policy 로 플러그인** — HF 가중치를 로컬 GPU 에 올려 AlpaSim 안에서 돌려보기.
3. **Cosmos-Drive-Dreams** 를 world backbone 대용으로 오프라인 데이터 증강.
4. NVIDIA DRIVE Hyperion / Omniverse 기반 **데이터 캡처 파이프라인** 리뷰.
5. Cosmos-RL 의 RL post-training 절차를 읽어 둠 → AlpaDreams 가 공개되면 곧바로 on-policy 학습 파이프라인에 연결 가능.

**주의**: Alpamayo 모델 가중치는 **비상업 라이선스** 이므로 상용 제품에는 그대로 넣을 수 없습니다. 연구·평가용으로만 사용하세요.

---

## 참고 자료 (출처)

- **공식 프로젝트 페이지**: [AlpaDreams — NVIDIA SIL](https://research.nvidia.com/labs/sil/projects/alpadreams/)
- **NVIDIA DRIVE 공식 소개 트윗**: [x.com/NVIDIADRIVE/.../2034622341578133539](https://x.com/NVIDIADRIVE/status/2034622341578133539)
- **리드 연구자 Xuanchi Ren 트윗**: [x.com/xuanchi13/.../2033702698248274036](https://x.com/xuanchi13/status/2033702698248274036)
- **Xuanchi Ren 홈페이지**: [xuanchiren.com](https://xuanchiren.com/)
- **Alpamayo GitHub**: [github.com/NVlabs/alpamayo](https://github.com/NVlabs/alpamayo)
- **AlpaSim GitHub**: [github.com/NVlabs/alpasim](https://github.com/NVlabs/alpasim)
- **Cosmos-Drive-Dreams arXiv**: [arxiv.org/abs/2506.09042](https://arxiv.org/abs/2506.09042)
- **Cosmos-Drive-Dreams GitHub**: [github.com/nv-tlabs/Cosmos-Drive-Dreams](https://github.com/nv-tlabs/Cosmos-Drive-Dreams)
- **Cosmos-Drive-Dreams 데이터셋 (HF)**: [huggingface.co/datasets/nvidia/PhysicalAI-Autonomous-Vehicle-Cosmos-Drive-Dreams](https://huggingface.co/datasets/nvidia/PhysicalAI-Autonomous-Vehicle-Cosmos-Drive-Dreams)
- **NVIDIA Cosmos 공식**: [nvidia.com/en-us/ai/cosmos](https://www.nvidia.com/en-us/ai/cosmos/)
- **NVIDIA Alpamayo 발표 뉴스룸**: [nvidianews.nvidia.com/news/alpamayo-autonomous-vehicle-development](https://nvidianews.nvidia.com/news/alpamayo-autonomous-vehicle-development)
- **Alpamayo 기술 블로그**: [developer.nvidia.com/blog/building-autonomous-vehicles-that-reason-with-nvidia-alpamayo](https://developer.nvidia.com/blog/building-autonomous-vehicles-that-reason-with-nvidia-alpamayo/)
- **Neural Reconstruction + WFM 블로그**: [developer.nvidia.com/blog/accelerating-av-simulation-with-neural-reconstruction-and-world-foundation-models](https://developer.nvidia.com/blog/accelerating-av-simulation-with-neural-reconstruction-and-world-foundation-models/)
- **NVIDIA GTC 2026 Virtual Worlds 종합 기사**: [blogs.nvidia.com/blog/gtc-2026-virtual-worlds-physical-ai](https://blogs.nvidia.com/blog/gtc-2026-virtual-worlds-physical-ai/)
- **Waymo World Model**: [waymo.com/blog/2026/02/the-waymo-world-model-a-new-frontier-for-autonomous-driving-simulation](https://waymo.com/blog/2026/02/the-waymo-world-model-a-new-frontier-for-autonomous-driving-simulation/)
- **Awesome World Model for AV**: [github.com/HaoranZhuExplorer/World-Models-Autonomous-Driving-Latest-Survey](https://github.com/HaoranZhuExplorer/World-Models-Autonomous-Driving-Latest-Survey)

---

### 변경 로그

- **2026-04-16**: 초기 정리본 작성 — 공식 페이지 본문이 아직 제목 수준이어서 NVIDIA DRIVE / Xuanchi Ren 공식 발표, Cosmos-Drive-Dreams · AlpaSim · Alpamayo 오픈 자료를 교차 참조해 구성. 코드/논문 공개 시 상단 표의 "릴리스 상태" 갱신 예정.
