---
name: vp-archival-and-reenactment
description: 아카이브 푸티지·재현 컷 문법 — 실제 자료와 AI 재현을 섞는 사건·다큐 포맷. "재현", "아카이브", "실제 영상", "사건", "다큐", "옛날 영상", "CCTV", "뉴스 자료", "흑백" 키워드 또는 실화를 다루는 컷에서 로드. 사건기록부·사에키 핵심. 재현이 실제처럼 보이면 안 되고, 실제가 재현처럼 보여도 안 된다. 라우터=video-prompt-router.
user-invocable: true
---

# 아카이브 · 재현 문법

> 🔴 **착수 전 필독 — [실패 정본 `vp-failure-registry`](../vp-failure-registry/SKILL.md)**
> 이 스킬의 규칙은 전부 **실패로 산 것**이다. 반증된 처방 8건·소환 사고 8건·비용 사고 5건이 거기 있다.
> 엔진·코드 실패는 the engine failure log (not included).


> **원칙**: 🔴 **실제 사건 영상은 실제 아카이브를 쓴다.** 재현으로 대체하면 신뢰가 깨진다.
> AI 재현은 **아카이브가 없는 구간을 메우는 용도**이며, **재현임이 화면에서 구분**돼야 한다.

## 1. 실제 vs 재현 — 구분 장치

| 구분 | 화면 처리 |
|---|---|
| 실제 아카이브 | 원본 그대로 (색보정 최소, 원본 노이즈 유지) |
| **AI 재현** | 🔴 **의도적 스타일 차이**를 준다 — 저채도·얕은 심도·부분 실루엣·느린 템포 |
| 자료 사진 | 정지 + 미세 Ken Burns (i2v로 움직이지 마라 — 인물이 살아 움직이면 사기) |

🔴 **재현 컷에 자막·라벨**로 재현임을 밝히는 것이 안전하다(포맷 규칙에 따름).

## 2. 시대별 룩 (재현 컷)

| 시대 | 룩 어휘 |
|---|---|
| 1900~30 | `hand-cranked frame rate, heavy grain, blown highlights, sepia toning, frame edge flicker` |
| 1940~60 | `16mm black and white, high contrast, dust and scratches, slight gate weave` |
| 1970~80 | `faded colour negative, magenta shift, halation around lights, soft focus falloff` |
| 1990~2000 | `VHS chroma bleed, tracking noise band, interlace combing on motion` |
| CCTV | `fixed high angle, low frame rate stutter, timestamp burn area (leave empty), IR wash at night` |
| 캠코더 | `handheld with auto-exposure hunting, zoom rocker jitter` |

⚠️ **타임스탬프·자막을 모델에게 그리게 하지 마라** — 글자가 깨진다. 자리를 비우고 **후반 합성**.

### ✅✅ 실측 검증 골격 (Gen18 · 1970년대 도쿄 골목)

```
스틸: hand-painted shop signs and paper lanterns along both walls, a bicycle leaning against a wall,
      single bare bulb over a doorway. Slight film grain, faded colour, period-accurate.
i2v : The paper lanterns swing slightly and their light wavers on the wet asphalt;
      steam drifts out from a vent low on the left wall; the bare bulb over the doorway flickers once.
```
| 항목 | 결과 |
|---|---|
| zoom | **−0.02%** ✅ 완전 고정 |
| detail / hf | **143** / 8.65% |
| **간판 글자(`やきとん`·`熱燗`)** | ✅ **4프레임 내내 온전** |
| 컷 경계 | 0 |

🔴🔴 **범위 정정: "글자가 깨지니 간판 있는 배경을 피하라"는 과했다.**
**스틸에 이미 있는 글자는 정적 컷에서 유지된다.** 못 하는 것은 **없던 글자를 새로 만드는 것**이다
(`vp-text-and-graphics` §0). → 시대 배경은 **간판을 살린 스틸 + 고정 컷**이 정답이다.

🔴 **시대 컷의 앰비언트 3종**: ①매달린 것의 흔들림 ②그 빛이 젖은 바닥에서 흔들림 ③증기·연기 1줄.
`flickers once` 처럼 **횟수를 지정**하면 발광체가 폭주하지 않는다(§4 발광체 규칙과 양립).

## 3. 재현 컷의 안전한 연출

**얼굴을 정면으로 오래 보여주지 마라** — 정체성·초상 리스크 + 드리프트.
```
✅ seen from behind · hands only · silhouette against the window · out-of-focus in the midground
✅ the object at the centre, the person entering and leaving frame
```
🔴 단, **얼굴을 완전히 가리면 귀신**이 된다 — 부분 노출·측면·실루엣 중 하나를 골라라.

## 4. 🔴 대본에서 넘어오는 사고 2종

1. **가정법이 하류에서 사실이 된다.** "만약 ~였다면"이 요약 단계에서 조건절을 잃고
   **재현 컷까지 제작**된 사고가 있었다 → 대본의 조건절은 **컷 지시로 내려보내지 마라**
2. **널리 퍼진 수치**는 ①1차출처 ②표본수 ③학명을 확인하고 써라 — 대본이 틀리면 컷도 틀린다

## 5. 죽음 · 부상 서술

🔴 **'있는 상태'로 서술하라.** `dead`·`lifeless`는 부재라서 모델이 살아 움직이게 만든다.
```
✅ lying motionless, limbs slack, eyes closed, chest still
✅ (moderation 회피) unconscious → sleeping
```

## 6. 아카이브 검색 주의

- 유명 사건일수록 검색어가 **책 표지·영화 포스터**를 물어온다 → 결과를 눈으로 확인
- 유튜브 자막 IP 차단 시 whisper 폴백
- 실제 푸티지의 픽셀 포맷(`yuvj420p`) 때문에 QA가 막힐 수 있다 → 변환

## 7. 톤

- 조명: 단일 광원 + 큰 미광부 · 시선: 카메라 응시 금지 · 표정: 미소 금지
- 색: `desaturated, crushed shadows` / `cold blue with sodium orange practicals`
- 액션 컷은 **배속 1.5~1.6 전제**로 발주 (달리기 1.6 · 근접 격투 1.5)
- 상세 = `vp-genre-playbooks/references/horror-incident.md`

## 8. 체크리스트

- [ ] 실제 사건에 **실제 아카이브**를 썼는가
- [ ] 재현 컷이 **시각적으로 구분**되는가
- [ ] 자막·타임스탬프를 **모델에게 그리게 하지 않았는가**
- [ ] 얼굴을 **부분 노출/측면/실루엣** 중 하나로 처리했는가 (완전 가림 아님)
- [ ] 대본의 **가정법**이 컷 지시로 넘어오지 않았는가
- [ ] 죽음·의식불명을 **'있는 상태'** 로 서술했는가
- [ ] 아카이브 검색 결과를 **눈으로 확인**했는가
