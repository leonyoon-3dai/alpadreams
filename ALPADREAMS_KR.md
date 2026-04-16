# NVIDIA AlpaDreams 한국어 정리 (v2)

> **2026년 GTC26 공개. NVIDIA SIL (Spatial Intelligence Lab) 의 자율주행용 action-conditioned 생성형 world model.** 본 문서는 공식 프로젝트 페이지, NVIDIA DRIVE/연구진 공개 발표, 동반 릴리스(Alpamayo · AlpaSim · Cosmos) 를 교차 확인해 작성한 한국어 비공식 요약입니다. 이미지·영상 링크는 모두 NVIDIA 공식 자산입니다.

**문서 기준일**: 2026-04-16 · **v2 변경점**: 공식 페이지 본문(이전 버전에서는 제목만 노출)이 확보되어 릴리스 계획·아키텍처·팀 구성·멀티미디어 자료를 모두 반영했습니다.

---

## 👀 30초 안에 이해하기

<p align="center">
  <a href="https://research.nvidia.com/labs/sil/projects/alpadreams/teaser.mp4">
    <img src="https://img.youtube.com/vi/MI7Fdpty02A/maxresdefault.jpg"
         alt="AlpaDreams Hero — 알파마요 정책이 코스모스가 실시간 생성한 세계 안에서 주행 중"
         width="720">
  </a>
</p>

> 🎬 **Hero Teaser** (클릭 시 재생, 약 30초): NVIDIA [Alpamayo 1](https://research.nvidia.com/publication/2025-10_alpamayo-r1) 정책 모델이 **AlpaDreams 가 실시간으로 생성한 세계** 안에서 폐루프(closed-loop)로 주행하는 장면. 화면 속 모든 도로·차량·조명은 **재구성이 아니라 Cosmos 가 그 순간 합성** 한 것입니다.
> 원본 MP4 → [research.nvidia.com/.../alpadreams/teaser.mp4](https://research.nvidia.com/labs/sil/projects/alpadreams/teaser.mp4)

한 문장 정리:

> **"기록된 과거를 재생하는 시뮬레이터" 가 아니라, "다음 순간을 생성하는 시뮬레이터".**

---

## 목차
1. [핵심 요약 카드](#핵심-요약-카드)
2. [AlpaDreams 가 무엇인가 (영상 비교)](#1-alpadreams-가-무엇인가-영상-비교)
3. [릴리스 상태 — 드디어 공개 예정](#2-릴리스-상태--드디어-공개-예정)
4. [동작 원리 (공식 도식 3장)](#3-동작-원리-공식-도식-3장)
5. [기술 스펙 — 실시간 성능](#4-기술-스펙--실시간-성능)
6. [생성형이어서 할 수 있는 것 (클립 모음)](#5-생성형이어서-할-수-있는-것-클립-모음)
7. [Alpamayo / AlpaSim / AlpaDreams 삼각 구도](#6-alpamayo--alpasim--alpadreams-삼각-구도)
8. [선행 연구 Cosmos-Drive-Dreams](#7-선행-연구-cosmos-drive-dreams)
9. [기반 플랫폼 NVIDIA Cosmos](#8-기반-플랫폼-nvidia-cosmos)
10. [알아두면 좋은 관련 라이브러리·논문](#9-알아두면-좋은-관련-라이브러리논문)
11. [기여자 목록 (공식 크레딧)](#10-기여자-목록-공식-크레딧)
12. [참고 자료 (출처 링크)](#11-참고-자료-출처-링크)

---

## 핵심 요약 카드

| 항목 | 내용 |
|------|------|
| **정식 명칭** | AlpaDreams — Real-Time Generative Closed-Loop Autonomous Vehicle Simulation Built on NVIDIA Cosmos |
| **개발 주체** | NVIDIA Spatial Intelligence Lab (SIL) · NVIDIA DRIVE |
| **최초 공개** | NVIDIA GTC26 (2026년 봄, San Jose) — 부스 데모 + 공식 블로그 |
| **대표 연구자** | Sanja Fidler, Xuanchi Ren, Zan Gojcic, Jun Gao, Amlan Kar 외 총 29명 |
| **모델 유형** | Action-conditioned **transformer 기반 autoregressive** 생성형 world model (KV cache) |
| **백본** | NVIDIA **Cosmos** World Foundation Model (post-trained) |
| **입력 조건** | 월드 시나리오(레인/바운딩박스) + 텍스트 프롬프트 + 메모리 캐시 |
| **출력** | 다음 스텝 카메라 프레임 (gRPC 로 AlpaSim 에 전달) |
| **해상도/FPS** | **4 카메라 × 105 FPS × 704×1280** (16× GB300 NVL72 기준) |
| **형제 프로젝트** | **Alpamayo** (VLA 정책) + **AlpaSim** (시뮬레이터 런타임) |
| **선행 연구** | **Cosmos-Drive-Dreams** (arXiv 2506.09042) |
| **코드 공개 일정** | ✅ GTC26 이후 **AlpaSim 에 리서치 통합** 형태로 공개 예정 (공식 명시) |
| **공식 페이지** | https://research.nvidia.com/labs/sil/projects/alpadreams/ |

---

## 1. AlpaDreams 가 무엇인가 (영상 비교)

[![Comparison of generative and reconstruction-based closed-loop simulation](https://img.youtube.com/vi/MI7Fdpty02A/maxresdefault.jpg)](https://www.youtube.com/watch?v=MI7Fdpty02A)

> 🎬 **Video 1 — 생성형 vs 재구성 기반 폐루프 시뮬레이션 비교** (왼쪽 위부터 시계방향: World Scenario / AlpaDreams / Neural Reconstruction / Policy). 재구성은 "기록된 골목" 을 벗어나지 못하지만, AlpaDreams 는 새로운 장면을 생성해 정책을 노출시킵니다.

### 왜 생성형이 필요한가

기존 자율주행 시뮬레이터 계보는 다음 3단계입니다.

```
① Log-replay        : 녹화된 센서 로그를 그대로 재생 → 정책이 달라져도 월드는 불변
② Neural Reconstruction : 3DGS/NeRF 로 씬을 재구성 → 시점 자유도↑, 그러나 captured corridor 이탈 불가
③ Generative World Model (← AlpaDreams): 거대 비디오 prior 에 기반해 새 장면 자체를 합성
```

AlpaDreams 의 공식 설명을 그대로 옮기면:

> "Reconstruction-based workflows remain fundamentally anchored to the data that was originally observed … **generative world models trained on massive amounts of visual data learn rich priors** about how the world behaves and visually evolves. These priors allow such models to synthesize very challenging phenomena such as **highly dynamic weather conditions (rain, storm, snow, wind)** or **deformable objects** (such as a mattress on top of a car)."

---

## 2. 릴리스 상태 — 드디어 공개 예정

**✅ 이전 버전 문서에서 '미공개' 로 적었던 부분이 공식 페이지에서 명시적으로 바뀌었습니다**:

> "**After GTC we will release a research integration of AlpaDreams into AlpaSim** as a reference workflow for generative closed-loop simulation, enabling the community to build on it and adapt it to their own needs. We aim to empower developers to use Cosmos to create their own AlpaDreams-style world models, extend sensor suites, and add scenario controls."

| 리소스 | 현재 상태 (2026-04-16) |
|--------|----------------------|
| 공식 프로젝트 페이지 | ✅ 공개 (본문 + 영상 6개 + 이미지 3장) |
| Alpamayo 1 논문 | ✅ [publication/2025-10_alpamayo-r1](https://research.nvidia.com/publication/2025-10_alpamayo-r1) |
| AlpaSim 런타임 | ✅ 오픈소스 [NVlabs/alpasim](https://github.com/NVlabs/alpasim) |
| Alpamayo 모델 | ✅ [NVlabs/alpamayo](https://github.com/NVlabs/alpamayo) · HF [nvidia/Alpamayo-R1-10B](https://huggingface.co/nvidia/Alpamayo-R1-10B) |
| Cosmos Transfer 2.5 | ✅ [nvidia-cosmos/cosmos-transfer2.5](https://github.com/nvidia-cosmos/cosmos-transfer2.5) |
| **AlpaDreams 자체 코드** | ⏳ **GTC26 이후 AlpaSim 리서치 통합 예정** |
| arXiv 논문 | ❌ 아직 등재되지 않음 (2026-04-16) |

> 💡 즉, "실물 코드는 아직이지만 오픈 계획이 명확히 약속된 상태" 입니다. 주변 구성요소(AlpaSim/Alpamayo/Cosmos)가 먼저 오픈되어 있으므로 파이프라인 학습 용도로 조합해 볼 수 있습니다.

---

## 3. 동작 원리 (공식 도식 3장)

### 3.1 전체 폐루프 워크플로우

<p align="center">
  <img src="https://research.nvidia.com/labs/sil/projects/alpadreams/images/fig1.jpg"
       alt="AlpaDreams closed-loop block diagram"
       width="760">
</p>

> 📘 **Figure 1 — 폐루프 워크플로우**: 정책 모델(Alpamayo 1) 또는 사용자가 액션을 AlpaSim 으로 보냄 → AlpaSim 이 시뮬레이션 상태를 갱신하고 컨텍스트를 AlpaDreams 로 전달 → AlpaDreams 가 다음 카메라 프레임 생성 → 정책으로 반환. 이 닫힌 루프가 바로 "closed-loop" 의 정의입니다.

```
     ┌─────────────┐  action    ┌─────────────┐  context    ┌─────────────┐
     │  Alpamayo 1 │ ─────────▶ │   AlpaSim   │ ─────────▶ │ AlpaDreams  │
     │ (policy/VLA)│            │  (runtime)  │            │ (world model│
     │             │            │             │            │  on Cosmos) │
     │             │ ◀───────── │             │ ◀───────── │             │
     └─────────────┘ sensor obs └─────────────┘  frames    └─────────────┘
                              (gRPC 인터페이스로 통신)
```

### 3.2 입력 조건 3종 세트

<p align="center">
  <img src="https://research.nvidia.com/labs/sil/projects/alpadreams/images/fig2.jpg"
       alt="AlpaDreams input conditioning diagram"
       width="760">
</p>

> 📘 **Figure 2 — 3개의 컨트롤 입력**
> 1. **World scenario**: 시뮬레이터 다음 상태를 차선/바운딩박스로 추상화 (Cosmos Transfer 2.5 멀티뷰 체크포인트와 동일 표현)
> 2. **Text prompt**: 조명·날씨·시간대 등 환경 변주를 자연어로 지시
> 3. **Memory cache**: 과거 프레임 컨텍스트 (KV cache) — 시간적 일관성 유지

### 3.3 Autoregressive 생성 vs 일괄 생성

<p align="center">
  <img src="https://research.nvidia.com/labs/sil/projects/alpadreams/images/fig3.jpg"
       alt="Autoregressive video generation diagram"
       width="760">
</p>

> 📘 **Figure 3 — 양방향 image-to-video 디노이징(왼쪽)** vs **causal KV-cache 기반 autoregressive 생성(오른쪽)**. offline video 생성 모델은 한 번에 긴 비디오를 내지만, 폐루프에 쓰려면 "정책이 조향을 꺾자마자 월드가 반응" 해야 하므로 AlpaDreams 는 **짧은 다음 프레임 묶음을 여러 번** 생성합니다.

---

## 4. 기술 스펙 — 실시간 성능

| 항목 | 값 |
|------|-----|
| **아키텍처** | Transformer (Cosmos 기반) + KV Cache |
| **생성 방식** | Autoregressive step-wise (few-step denoising) |
| **스테이트 렌더러** | 프레임당 < 1 ms |
| **디노이징 가속** | Few-step schedule → 단일 뷰 대비 **최대 35× 속도 향상** |
| **인퍼런스 최적화** | CUDA Graphs, context parallelism (멀티 GPU) |
| **실측 처리량** | **4 카메라 × 105 FPS × 704×1280** @ 16× GB300 NVL72 |
| **카메라 일관성** | cross-view attention 으로 3D consistency 확보 |
| **롱테일 커버** | 프롬프트 기반 counterfactual (야간/눈/비/안개 등) |
| **롱 롤아웃 안정성** | 수 분 단위 rollout 에서 시각 붕괴 없음 (recondition + memory cache) |

> 📎 성능 원문: *"AlpaDreams generates four camera views at 105 FPS each with 704×1280 resolution on 16 NVIDIA GB300 NVL72 GPUs."*

---

## 5. 생성형이어서 할 수 있는 것 (클립 모음)

### 5.1 극단적 기상·변형 물체·장면 편집

<table>
<tr>
<td width="33%" align="center">
  <a href="https://research.nvidia.com/labs/sil/projects/alpadreams/wind.mp4">
    <img src="https://img.youtube.com/vi/AAsM2RcZtxo/hqdefault.jpg" width="100%" alt="windy drive"><br>
    🌬️ <b>Windy drive</b>
  </a>
</td>
<td width="33%" align="center">
  <a href="https://research.nvidia.com/labs/sil/projects/alpadreams/mattress.mp4">
    <img src="https://img.youtube.com/vi/MI7Fdpty02A/hqdefault.jpg" width="100%" alt="mattress on highway"><br>
    🛏️ <b>하이웨이 위 매트리스</b>
  </a>
</td>
<td width="33%" align="center">
  <a href="https://research.nvidia.com/labs/sil/projects/alpadreams/editing.mp4">
    <img src="https://img.youtube.com/vi/CAZVe0NTuN8/hqdefault.jpg" width="100%" alt="scene editing"><br>
    ✂️ <b>씬 편집</b>
  </a>
</td>
</tr>
</table>

> 📎 원본 MP4 링크 (직접 재생 가능):
> [wind.mp4](https://research.nvidia.com/labs/sil/projects/alpadreams/wind.mp4) ·
> [mattress.mp4](https://research.nvidia.com/labs/sil/projects/alpadreams/mattress.mp4) ·
> [editing.mp4](https://research.nvidia.com/labs/sil/projects/alpadreams/editing.mp4)

### 5.2 사람이 직접 주행 (Human-in-the-loop)

[![Human driving in generative closed-loop simulation](https://img.youtube.com/vi/WjyQ5PuYLa8/maxresdefault.jpg)](https://www.youtube.com/watch?v=WjyQ5PuYLa8)

> 🎬 **Video 2** — 사람이 핸들로 AlpaDreams 안에서 운전. 같은 gRPC 인터페이스로 policy 자리에 human 이 들어갈 수 있어, **디버깅/시연/전문가 데모 수집** 용도로 유용.

### 5.3 보행자·자전거 등 어려운 long-tail 시나리오

[![Pedestrians and cyclists](https://img.youtube.com/vi/AAsM2RcZtxo/maxresdefault.jpg)](https://www.youtube.com/watch?v=AAsM2RcZtxo)

> 🎬 **Video 3** — Vulnerable Road Users (VRU) 시나리오. 신경 재구성이 약한 **관절 동작**(사람 걷기, 자전거 타기, 와이퍼) 을 Cosmos prior 로 잘 합성. 신호등·차선·표지판 같은 의미적 단서도 보존.

### 5.4 다중 카메라 동시 생성

[![Four-camera generation](https://img.youtube.com/vi/CAZVe0NTuN8/maxresdefault.jpg)](https://www.youtube.com/watch?v=CAZVe0NTuN8)

> 🎬 **Video 5** — 4 카메라를 **동기화** 해 서라운드 뷰 생성. cross-view attention 으로 동적 객체·조명이 카메라 간 일관되게 유지.

### 5.5 Counterfactual: 낮 → 밤 → 눈

[![Counterfactual variations](https://img.youtube.com/vi/MoNZwSaqzp0/maxresdefault.jpg)](https://www.youtube.com/watch?v=MoNZwSaqzp0)

> 🎬 **Video 6** — 같은 씬을 (상) 원본, (중) 야간, (하) 눈 내리는 날씨 로 프롬프트만 바꿔 변주. 단일 프롬프트로 **균형 잡힌 학습·평가 데이터 엔진** 역할 가능.

### 5.6 긴 롤아웃에서도 드리프트 없음

<p align="center">
  <a href="https://research.nvidia.com/labs/sil/projects/alpadreams/recording_4x.mp4">
    <img src="https://img.youtube.com/vi/MI7Fdpty02A/hqdefault.jpg" alt="Long rollout 4x speed" width="640">
  </a>
</p>

> 🎬 **Video 4** — 긴 비디오 시퀀스를 4배속으로 재생. 반복해서 상태와 최근 히스토리로 **recondition** 하기 때문에 multi-minute rollout 에서도 외관 붕괴가 관찰되지 않음.
> 원본 MP4 → [recording_4x.mp4](https://research.nvidia.com/labs/sil/projects/alpadreams/recording_4x.mp4)

---

## 6. Alpamayo / AlpaSim / AlpaDreams 삼각 구도

```
┌───────────────────────────────────────────────────────────┐
│                      Alpamayo 1                           │
│   Vision-Language-Action 정책 (Level-4 지향, 10B 파라미터) │
│   ▶ "차가 어떻게 사고하고 움직일지"                        │
└───────────────────────────────────────────────────────────┘
                          │ action
                          ▼
┌───────────────────────────────────────────────────────────┐
│                       AlpaSim                             │
│   오픈소스 closed-loop 시뮬레이션 런타임                   │
│   gRPC 마이크로서비스, 시나리오 상태 관리                  │
│   ▶ "전체 파이프라인을 어떻게 실행/평가할지"               │
└───────────────────────────────────────────────────────────┘
                          │ context
                          ▼
┌───────────────────────────────────────────────────────────┐
│                    AlpaDreams  (★ 이 문서)                │
│   Action-conditioned generative world model (Cosmos 기반) │
│   ▶ "월드가 그 액션에 어떻게 반응해서 어떻게 보일지"       │
└───────────────────────────────────────────────────────────┘
```

| | Alpamayo 1 | AlpaSim | AlpaDreams |
|---|----------|---------|------------|
| 역할 | 정책(Policy) | 시뮬레이터 쉘 | World model |
| 입력 | 멀티뷰 비디오 + ego history | 센서·시나리오 스펙 | 초기 state + action + 텍스트 |
| 출력 | 추론 trace + 6.4s trajectory | 물리·gRPC 오케스트레이션 | 다음 RGB 프레임 (멀티뷰) |
| 오픈소스 | ✅ Apache 2.0 (inference) / 모델은 비상업 | ✅ Apache 2.0 | ⏳ 연구 통합 예정 |
| 코어 모델 | Cosmos-Reason | N/A | Cosmos Transfer 계열 post-train |

---

## 7. 선행 연구: Cosmos-Drive-Dreams

| 항목 | Cosmos-Drive-Dreams | AlpaDreams |
|------|---------------------|------------|
| 릴리스 | ✅ 모델·파이프라인·데이터 오픈 (2025) | ⏳ AlpaSim 통합 예정 |
| 목적 | **오프라인** 합성 데이터 생성 | **온라인** closed-loop 시뮬 |
| 조건 입력 | HDMap, 3D cuboid, 텍스트, LiDAR depth | **Action + 월드 스테이트 + 텍스트 + 메모리** |
| 출력 | 121 프레임 / 10초 클립 (사전 생성) | **스텝 단위 짧은 프레임** (실시간) |
| 상호작용 | ❌ 단방향 생성 | ✅ 정책-월드 피드백 |
| 논문 | [arXiv 2506.09042](https://arxiv.org/abs/2506.09042) | — |
| GitHub | [nv-tlabs/Cosmos-Drive-Dreams](https://github.com/nv-tlabs/Cosmos-Drive-Dreams) | — |
| 데이터셋 | HF [PhysicalAI-Autonomous-Vehicle-Cosmos-Drive-Dreams](https://huggingface.co/datasets/nvidia/PhysicalAI-Autonomous-Vehicle-Cosmos-Drive-Dreams) (5,843 클립 + 81,802 합성 비디오) | — |

두 프로젝트의 **리드 연구자(Xuanchi Ren) 가 명시적으로 후속작이라 소개**했으며, 컨트롤 표현도 Cosmos Transfer 2.5 와 호환됩니다.

---

## 8. 기반 플랫폼: NVIDIA Cosmos

| 구성요소 | 역할 |
|----------|------|
| Cosmos Tokenizer | 비디오 → 잠재 공간 토큰화 |
| Cosmos Predict / Transfer | Diffusion·Autoregressive WFM, 미래 프레임 생성 (**AlpaDreams 가 여기서 post-train**) |
| Cosmos Reason | VLM 추론 백본 (Alpamayo 가 사용) |
| Cosmos Guardrails | 안전성 필터링 |
| Cosmos-RL | Post-training 용 RL 프레임워크 |

📎 관련 링크:
[Cosmos 공식](https://www.nvidia.com/en-us/ai/cosmos/) ·
[Cosmos Cookbook (nvda.ws)](https://nvda.ws/4qevli8) ·
[HF collections (cosmos)](https://huggingface.co/nvidia/collections?search=cosmos) ·
[build.nvidia.com 데모](https://nvda.ws/3Yg0Dcx)

---

## 9. 알아두면 좋은 관련 라이브러리·논문

### A. NVIDIA 자체 생태계
| 프로젝트 | 한 줄 요약 | 링크 |
|----------|-----------|------|
| NVIDIA Cosmos | World Foundation Model 플랫폼 | [nvidia.com/.../cosmos](https://www.nvidia.com/en-us/ai/cosmos/) |
| Cosmos-Drive-Dreams | WFM 기반 합성 주행 데이터 생성 | [arXiv](https://arxiv.org/abs/2506.09042) · [GitHub](https://github.com/nv-tlabs/Cosmos-Drive-Dreams) |
| Cosmos Transfer 2.5 | 멀티뷰 컨트롤-to-비디오 모델 (AlpaDreams 컨트롤 표현의 원형) | [github.com/nvidia-cosmos/cosmos-transfer2.5](https://github.com/nvidia-cosmos/cosmos-transfer2.5) |
| Alpamayo 1 / 1.5 | VLA 자율주행 정책 | [GitHub](https://github.com/NVlabs/alpamayo) · [HF](https://huggingface.co/nvidia/Alpamayo-R1-10B) · [publication](https://research.nvidia.com/publication/2025-10_alpamayo-r1) |
| AlpaSim | 오픈소스 closed-loop 시뮬레이터 | [GitHub](https://github.com/NVlabs/alpasim) |
| NVIDIA DRIVE | 차량 소프트웨어 스택 | [nvidia.com/drive](https://developer.nvidia.com/drive/simulation) |
| GB300 NVL72 | Blackwell Ultra 랙 (AlpaDreams 레퍼런스 HW) | [nvidia.com/.../gb300-nvl72](https://www.nvidia.com/en-us/data-center/gb300-nvl72/) |

### B. 경쟁·비교 가능한 AV world model
| 프로젝트 | 제공 주체 | 특징 |
|----------|-----------|------|
| Waymo World Model | Waymo | 2026-02 공개, closed-loop AV 시뮬레이션 |
| GAIA-1 / GAIA-2 | Wayve | 생성형 driving world model |
| DriveDreamer 시리즈 | 학계 컨소시엄 | Diffusion 기반 driving WM |
| GenAD | 학계 | 일반화된 video generation for AD |
| Vista (Drive-WM) | 학계 | 고충실도 driving scene 생성 |
| ADriver-I | 학계 | 언어 조건 액션-컨디셔닝 |

📎 메타 목록: [Awesome-World-Model (AV)](https://github.com/HaoranZhuExplorer/World-Models-Autonomous-Driving-Latest-Survey)

### C. 일반 world model / 비디오 생성
| 기법 | 요점 |
|------|------|
| DreamerV3 (Hafner+ 2023) | 모델 기반 RL 의 정석 |
| Genie / Genie 2 (DeepMind) | foundation world model, action-conditioning |
| Sora, Stable Video Diffusion | 장기 비디오 생성 트렌드 |
| Diffusion Forcing / Diffusion Policy | autoregressive 확산 |

### D. 폐루프와 엮이는 기하 백엔드 (AlpaDreams 의 "비교군")
| 기법 | 요점 |
|------|------|
| 3D Gaussian Splatting (3DGS) | 실시간 고품질 신경 렌더링 |
| 3DGRT / 3DGUT | 왜곡 카메라·롤링 셔터 지원 ray tracing/UT |
| Neural Harmonic Textures | 고주파 텍스처 재현용 NVIDIA 연구 |
| NeRF · Instant-NGP | implicit field 원조 |

---

## 10. 기여자 목록 (공식 크레딧)

공식 페이지에 명시된 총 **29인** (알파벳 순 원문 그대로):

> Aarti Basant · Amlan Kar · Despoina Paschalidou · Guillermo Garcia Cobo · Haithem Turki · Huan Ling · Jaewoo Seo · Jialiang Wang · James Lucas · Jay Wu · Jonathan Lorraine · Jun Gao · Kai He · Katarina Tothova · Kevin Xie · Michal Tyszkiewicz · Qi Wu · Riccardo de Lutio · Ruilong Li · Sanja Fidler · Seung Wook Kim · Tianchang Shen · Tianshi Cao · Tobias Pfaff · William Lew · Xuanchi Ren · Yifan Lu · Zan Gojcic · Zian Wang

팀 리드로 추정되는 **Sanja Fidler (NVIDIA VP, SIL)** 는 GTC26 에서 ["Advancing Autonomous Vehicles with World Models"](https://www.nvidia.com/gtc/session-catalog/sessions/gtc26-s82446/) 세션을 발표했습니다.

---

## 11. 참고 자료 (출처 링크)

### 일차 자료 (공식)
- 🏠 **공식 프로젝트 페이지**: https://research.nvidia.com/labs/sil/projects/alpadreams/
- 🐦 NVIDIA DRIVE 공식 소개: [x.com/NVIDIADRIVE/.../2034622341578133539](https://x.com/NVIDIADRIVE/status/2034622341578133539)
- 🐦 리드 연구자 Xuanchi Ren: [x.com/xuanchi13/.../2033702698248274036](https://x.com/xuanchi13/status/2033702698248274036)
- 🎤 Sanja Fidler GTC26 세션: [gtc26-s82446](https://www.nvidia.com/gtc/session-catalog/sessions/gtc26-s82446/)
- 👤 Xuanchi Ren 홈페이지: [xuanchiren.com](https://xuanchiren.com/)

### 일차 자료 (멀티미디어, 공식 CDN)

| 유형 | 파일 |
|------|------|
| 🎞️ Teaser 비디오 | [teaser.mp4](https://research.nvidia.com/labs/sil/projects/alpadreams/teaser.mp4) |
| 🎞️ Wind drive | [wind.mp4](https://research.nvidia.com/labs/sil/projects/alpadreams/wind.mp4) |
| 🎞️ Mattress on highway | [mattress.mp4](https://research.nvidia.com/labs/sil/projects/alpadreams/mattress.mp4) |
| 🎞️ Scene editing | [editing.mp4](https://research.nvidia.com/labs/sil/projects/alpadreams/editing.mp4) |
| 🎞️ Long rollout (4x) | [recording_4x.mp4](https://research.nvidia.com/labs/sil/projects/alpadreams/recording_4x.mp4) |
| 🖼️ Figure 1 (폐루프 블록도) | [images/fig1.jpg](https://research.nvidia.com/labs/sil/projects/alpadreams/images/fig1.jpg) |
| 🖼️ Figure 2 (입력 컨디셔닝) | [images/fig2.jpg](https://research.nvidia.com/labs/sil/projects/alpadreams/images/fig2.jpg) |
| 🖼️ Figure 3 (autoregressive) | [images/fig3.jpg](https://research.nvidia.com/labs/sil/projects/alpadreams/images/fig3.jpg) |
| ▶️ YouTube Video 1 (비교) | [watch?v=MI7Fdpty02A](https://www.youtube.com/watch?v=MI7Fdpty02A) |
| ▶️ YouTube Video 2 (인간 주행) | [watch?v=WjyQ5PuYLa8](https://www.youtube.com/watch?v=WjyQ5PuYLa8) |
| ▶️ YouTube Video 3 (보행자) | [watch?v=AAsM2RcZtxo](https://www.youtube.com/watch?v=AAsM2RcZtxo) |
| ▶️ YouTube Video 5 (4카메라) | [watch?v=CAZVe0NTuN8](https://www.youtube.com/watch?v=CAZVe0NTuN8) |
| ▶️ YouTube Video 6 (counterfactual) | [watch?v=MoNZwSaqzp0](https://www.youtube.com/watch?v=MoNZwSaqzp0) |

### 이차 자료 (기술 블로그·뉴스)
- [Building Autonomous Vehicles That Reason (Alpamayo 기술 블로그)](https://developer.nvidia.com/blog/building-autonomous-vehicles-that-reason-with-nvidia-alpamayo/)
- [Accelerating AV Simulation with Neural Reconstruction + WFM](https://developer.nvidia.com/blog/accelerating-av-simulation-with-neural-reconstruction-and-world-foundation-models/)
- [NVIDIA Alpamayo 발표 뉴스룸](https://nvidianews.nvidia.com/news/alpamayo-autonomous-vehicle-development)
- [GTC26 Virtual Worlds 기사](https://blogs.nvidia.com/blog/gtc-2026-virtual-worlds-physical-ai/)
- [Waymo World Model (참고 비교)](https://waymo.com/blog/2026/02/the-waymo-world-model-a-new-frontier-for-autonomous-driving-simulation/)

### 관련 오픈소스 (조합해 AlpaDreams 유사 파이프라인 구성 가능)
- [NVlabs/alpamayo](https://github.com/NVlabs/alpamayo)
- [NVlabs/alpasim](https://github.com/NVlabs/alpasim)
- [nv-tlabs/Cosmos-Drive-Dreams](https://github.com/nv-tlabs/Cosmos-Drive-Dreams)
- [nvidia-cosmos/cosmos-transfer2.5](https://github.com/nvidia-cosmos/cosmos-transfer2.5)
- [github.com/nvidia-cosmos](https://github.com/nvidia-cosmos)

---

### 변경 로그

- **2026-04-17 v2.1**: 크레딧 수 정정(27명 → 29명, 공식 페이지 목록과 대조), "다분 단위" 오타 수정("수 분 단위"), 섹션 10 정렬 설명 간소화.
- **2026-04-16 v2**: 공식 프로젝트 페이지 본문(영상 6편, 도식 3장, 기술 스펙, 기여자 목록, 릴리스 계획) 확보 후 대폭 보강. 이전 버전의 "코드 미공개" 기술을 "**GTC26 이후 AlpaSim 리서치 통합 형태로 공개 예정**" 으로 수정.
- **2026-04-16 v1**: 초기 정리 — 공식 페이지 본문이 잡히지 않아 SNS/주변 릴리스 교차 참조로 작성.
