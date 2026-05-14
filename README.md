# AlpaDreams — 한국어 정리

NVIDIA **AlpaDreams** (GTC26 공개, 자율주행용 action-conditioned generative world model) 에 대한 한국어 기술 요약 리포지토리입니다.

<p align="center">
  <a href="ALPADREAMS_KR.md">
    <img src="https://research.nvidia.com/labs/sil/projects/alpadreams/images/fig1.jpg" alt="AlpaDreams closed-loop block diagram" width="720">
  </a>
</p>

- 📖 본문: **[ALPADREAMS_KR.md](ALPADREAMS_KR.md)** — 30초 요약, 공식 영상·도식 포함 기술 분석, 릴리스 계획, Alpamayo/AlpaSim/Cosmos 관계, 관련 논문·라이브러리
- 🏠 공식 프로젝트 페이지: https://research.nvidia.com/labs/sil/projects/alpadreams/
- 📅 문서 기준일: **2026-04-16** (v2 · 공식 본문 반영)

## 2026-05-14 Codex 점검 메모

- 2026년 5월 기준으로 이 repo는 AlpaDreams를 "공식 코드 실행 repo"가 아니라 공식 페이지/영상/논문 흐름을 한국어로 읽기 위한 비공식 기술 노트로 보는 것이 맞습니다.
- 핵심 개념은 action-conditioned world model, closed-loop evaluation, autonomous driving simulation, Cosmos/Alpamayo/AlpaSim 계열과의 관계입니다. 용어가 빠르게 바뀌는 영역이므로 release plan과 공개 artifact 여부는 공식 프로젝트 페이지를 기준으로 재확인하세요.
- Isaac Sim/ROS 2 학습과 연결할 때는 "로봇/차량의 행동 action이 다음 scene state 분포를 어떻게 바꾸는가"를 중심축으로 두면 이해가 쉽습니다.

> ⚠️ 본 리포지토리는 공식 자료가 아닌 **비공식 한국어 요약**입니다. 모든 이미지·영상 링크는 NVIDIA 공식 CDN/유튜브 원본을 가리킵니다. 잘못된 부분은 이슈/PR 로 알려주세요.
