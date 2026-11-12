---
name: line-art-animation
description: >
  Builds animated line-art SVG components in TypeScript with natural walking gait, spring physics, and day/night cycles. Use when the user asks for line art animation, ink-on-paper characters, walking figures, living street scenes, or animated SVG components.
  키워드: 라인아트 애니메이션, 선화 애니메이션, SVG 애니메이션, 걷는 캐릭터, 잉크 일러스트, 라인아트
version: 1.0.0
status: active
references_path: ./references
---

# ✒️ Line-art Animation

TypeScript와 SVG 기반으로 따뜻한 종이 위에 얇은 먹선으로 그린 듯한 인물, 동물, 풍경 및 낮밤 주기를 연출하는 에이전트 스킬이다. 프레임워크 의존성 없이 순수 DOM/SVG 수학적 연산(`lib/`)으로 구동되며, 60fps 부드러운 애니메이션과 헤드리스 비디오 추출을 지원한다.

---

## Phase 0: 가드레일 및 안티패턴 점검
작업을 시작하기 전 반드시 [references/gotchas.md](./references/gotchas.md)를 먼저 확인한다.
- **발 미끄러짐 방지**: 보행 연산 시 `x += dir * walk.speed * SCALE * dt * go;` 공식을 어기면 발이 땅에서 미끄러진다.
- **기계적 사인파 금지**: 머리나 몸통을 단순 사인파로 흔들지 않고 `walk.pose()`의 `p.bob` 결과를 반영한다.
- **오픈 스트로크 준수**: 사지는 닫힌 파이프가 아니라 끝이 살짝 열린 두 가닥 먹선(`limb()`)으로 표현한다.
- **단독 카드 원칙**: 컴포넌트는 고정된 카드 안에서 완결되어야 하며 외부 웹페이지 배경을 오염시키지 않는다.

---

## Phase 1: 스타일 및 룩앤필 확정
장면을 그리기 전 [references/style.md](./references/style.md)를 참조하여 선 두께와 색상 팔레트를 고정한다.
- **펜 굵기**: 인물 기본 펜은 `1.35` 유닛, 배경 건물 펜은 `pen * scale`, 세부 격자는 `0.4~0.6 * pen`을 사용한다.
- **잉크 및 종이**: 주간 먹선은 완전한 검정이 아닌 `#1c1c1c`, 종이는 따뜻한 미색 `#fdfcf8`을 적용한다.
- **인물 규격**: 표준 인물 높이는 약 95유닛, 발 위치는 `y = 120`, 전방은 `+x` 방향, 기준점은 `x = 40`이다.
- **필터**: `inkDefs()`의 미세 요철(displacement)을 적용하여 기계적 벡터가 아닌 자연스러운 잉크 질감을 낸다.

---

## Phase 2: 프로젝트 세팅 및 라이브러리 탑재
`lib/` 디렉터리의 4가지 의존성 없는 TypeScript 모듈을 프로젝트에 배치한다.
- `lib/ink.ts`: 펜 스트로크, 사지(`limb`), 이음새(`joint`), 단색 채우기(`solid`), 렌더 루프(`run`).
- `lib/gait.ts`: 역운동학 기반 보행/달리기 해석 엔진 (`gait({ kind: 'walk' | 'shuffle' | 'run' })`).
- `lib/motion.ts`: 스프링 물리(`spring`), 진자(`pendulum`), 마우스 커서 반응(`pointerWind`), 시퀀서.
- `lib/daynight.ts`: 시간대별(`τ: 0.0~1.0`) 하늘, 종이, 잉크, 창문 점등 색상 팔레트 관리.

---

## Phase 3: 단일 피규어 및 역학 구현
전체 장면을 한 번에 만들지 않고, 반드시 한 명의 인물 또는 핵심 코너를 먼저 완성하여 승인받는다.
- [references/motion.md](./references/motion.md)의 보행 제어를 적용한다.
- 보행 시 앞뒤 다리의 레이어 순서(`p.front`)를 맞추고, 발목 회전각(`pitch`)을 적용하여 발꿈치가 자연스럽게 떨어지도록 한다.
- 옷자락(코트, 기모노)은 풀이된 무릎과 발목 좌표에 맞춰 걸을 때마다 자연스럽게 A자로 개폐되도록 연동한다.
- 소매, 가방, 늘어진 물품은 `spring()`이나 `pendulum()` 물리로 관성을 부여한다.

---

## Phase 4: 장면 구성 및 낮·밤 인터랙션
인물이 준비되면 [references/scenes.md](./references/scenes.md)를 참조하여 배경과 환경을 결합한다.
- **레이어 구성**: 먼 배경 하늘 → 건물 및 풍경 입면도 → 움직이는 인물/탈것 → 전경 소품 순으로 배치한다.
- **낮·밤 전환**: `applyPalette(tau)`를 호출하여 주간(먹선/미색 종이)에서 야간(인디고 종이/연한 먹선)으로 부드럽게 반전시킨다.
- **야간 발광**: 창문과 가로등에 `mix-blend-mode: screen` 레이어와 따뜻한 방사형 빛을 연출한다.
- **자연스러운 반응**: 마우스 이동을 바람(`pointerWind`)으로 치환하고, 인물이 모자를 만지거나 고양이가 기지개를 켜는 등 세상이 반응하는 인터랙션을 설계한다.

---

## Phase 5: 프레임 단위 정밀 검증
작업 완료를 선언하기 전 [references/verify.md](./references/verify.md) 절차에 따라 철저히 검증한다.
1. **정적 타입 및 빌드 검사**: `npm run typecheck && npm run build` 정상 통과 확인.
2. **결정론적 디버그 모드**: 브라우저 URL에 `?debug`를 붙여 rAF 스로틀링을 우회하고 `window.__step(1/60)`으로 1프레임씩 전진하며 발 미끄러짐, 옷자락 관통 여부를 검사한다.
3. **시간대 스윕**: `τ = 0.3`(낮), `0.72`(골든아워), `0.75`(일몰 반전), `0.85`(심야)를 확인하여 명도 대비를 검증한다.
4. **실제 프레임레이트**: 전면 탭에서 60fps 도달 여부를 확인한다.

---

## Phase 6: 영상 렌더링 및 클립 매칭
- **영상 추출**: [references/video.md](./references/video.md)에 따라 `scripts/capture.ts`로 결정론적 프레임을 캡처한 뒤 `ffmpeg`로 1080p MP4 비디오를 렌더링한다.
- **참조 클립 모사**: 특정 영상의 움직임을 재현할 때는 [references/matching.md](./references/matching.md)에 따라 `scripts/clip.ts`와 `scripts/frames.sh`로 실측하여 오차를 없앤다.

