# 원자력 뉴스 브리핑 (Nuclear News Brief)

매일 아침 국내외 원자력 뉴스를 모아, **원자력공학을 배우는 학생에게** 무슨 일이 있었는지와 공학적으로 왜 중요한지를 쉬운 말로 정리해 게시합니다.
지난 브리핑은 날짜를 골라 다시 볼 수 있습니다. 공개 주소: <https://malmok2.github.io/Nuclear_News/>

`malmok2/PaperCollection`(주간 논문 동향)과 같은 구조입니다: **GitHub Actions에서 실행 → 결과를 저장소에 커밋 → GitHub Pages로 게시 → 연구실 홈페이지(Think_webpage) 도구 목록에서 연결.**

## 1. 동작 방식 (API 키 없이 운영)

```
04:30 KST  GitHub Actions — 수집 (API 키 불필요)
  ├─ RSS 수집: Google 뉴스(국내·해외 검색), World Nuclear News, IAEA, U.S. NRC, ANS
  ├─ 걸러내기: 지난 36시간 · 원자력 키워드 · 북핵·핵무기 등 군사 기사 제외
  ├─ 같은 사건 여러 기사 → 한 묶음, 이미 게시한 기사 제외 (state/seen.json)
  ├─ data/days/날짜.json 저장 (engine = pending) → 제목만 있는 페이지 먼저 게시
  └─ 저장소에 커밋 + GitHub Pages 배포
06:50 KST  Claude Code 예약 작업(Routine) — 해설 작성 (교수님 Claude 구독 사용량에서 차감)
  ├─ python3 -m pipeline.session_brief prompt   → 규칙 + 후보 기사 목록 출력
  ├─ Claude가 해설 JSON 작성 (웹 검색·원문 열람 없음)
  ├─ python3 -m pipeline.session_brief apply    → 검증 + 저장 + docs/ 다시 생성
  └─ push → deploy-pages 워크플로가 사이트 재배포
09:00 KST  수집 재시도 — 04:30 수집이 실패했을 때만
```

해설 한 건은 "무슨 일이 있었나 / 공학도에게 왜 중요한가 / 개념 풀이 / 생각해 볼 질문 / 더 공부하기"로 구성됩니다.
`ANTHROPIC_API_KEY`를 Actions Secret으로 등록하면 04:30 수집 단계에서 API로 바로 해설을 쓰고, 예약 작업은 할 일이 없어 곧바로 끝납니다(선택).

### 분량 조절 (`config/settings.json`)

| 키 | 지금 | 뜻 |
|---|---|---|
| `max_candidates` | 30 | 해설 작성 시 보여 주는 후보 기사 묶음 수 |
| `min_stories` / `max_stories` | 3 / 5 | 해설하는 뉴스 수 |

처음에는 토큰을 아끼려고 작게 잡았습니다. 늘리려면 이 세 값만 바꾸면 됩니다(예: 45 / 4 / 8).

## 2. 저작권 원칙

| 항목 | 처리 |
|---|---|
| 기사 본문·사진 | **싣지 않음.** 수집·저장도 하지 않음 |
| 기사 제목·매체명·링크 | 원문으로 가는 링크로만 게시 |
| 전문 매체 RSS의 한 줄 요지 | AI 입력으로만 쓰고 **디스크에 저장하지 않음** (`run_daily.py`에서 제거) |
| 해설 | Claude가 새로 쓴 글. 기사 문장 복제·직접 인용 금지를 지시 (`pipeline/brief.py`의 `SYSTEM`) |
| 참고 링크 | AI가 URL을 지어내지 못하도록 `config/settings.json`의 `reference_library` 목록에서만 고름 |

AI 해설은 제목·요지만 보고 쓰므로, 제목에 없는 수치·날짜를 만들지 말고 불확실하면 그렇게 밝히도록 지시했습니다. 그래도 틀릴 수 있어 페이지 하단에 원문 확인 안내를 둡니다.

## 3. 최초 설정 (1회)

1. 저장소 공개(Public) 전환 — 무료 계정의 GitHub Pages 조건
2. Settings → Pages → Source: **GitHub Actions**
3. 예약 작업(Routine) 등록 — Claude Code 세션에서 등록함. claude.ai의 Routines 목록에서 확인·중지 가능
4. 예약 실행은 **기본 브랜치**의 워크플로만 동작합니다(현재 기본 브랜치에 있음)

## 4. 사용량

- 수집(GitHub Actions): 무료
- 해설(예약 작업): 하루 1회 세션. 해설용 입력 약 3~4천 토큰 + 출력 3~5천 토큰에, Claude Code 세션 자체의 고정 비용(시스템 지시문·도구 설명, 대부분 캐시)이 더해집니다. 실제 사용량은 첫 실행 뒤 확인이 필요합니다.

## 5. 로컬 실행

```bash
pip install -r requirements.txt
python -m pipeline.run_daily                     # 오늘(KST)
python -m pipeline.run_daily --date 2026-09-28 --force   # 특정 날짜 다시 만들기
python -m pipeline.run_daily --render-only       # data/days로 docs/만 다시 생성
python -m unittest tests.test_offline -v         # 네트워크 없이 전체 흐름 점검
```

## 6. 설정 — `config/settings.json`

| 키 | 내용 |
|---|---|
| `feeds` | RSS 목록. `region`(국내/해외), `needs_keyword`(원자력 키워드 필수 여부), `priority`(1=전문 매체), `enabled:false`로 끄기 |
| `include_keywords` / `exclude_keywords` | 관련성 필터 |
| `categories` | 뉴스 분류 (AI가 이 중에서만 고름) |
| `reference_library` | '더 공부하기' 링크 후보 |
| `max_candidates`, `min_stories`, `max_stories`, `lookback_hours` | 분량·수집 기간 |
| `model`, `effort` | API 키를 쓸 때만 적용 |

## 7. 폴더 구조

```
config/settings.json   설정
pipeline/              collect.py(수집) · brief.py(해설 규칙·API) · session_brief.py(예약 작업용) · build_site.py(페이지) · run_daily.py(수집 진입점)
data/days/             날짜별 브리핑 JSON (누적 아카이브)
state/                 seen.json(게시한 기사) · latest-run.json(마지막 실행 결과, 피드별 성공 여부)
docs/                  공개 사이트: index.html(오늘) · days/날짜.html · archive.html(날짜별 목록)
tests/                 오프라인 점검
```

## 8. 알려진 한계

- 피드 주소는 개발 환경(Claude 원격 세션)의 네트워크 정책상 직접 확인하지 못했습니다. 첫 실행 뒤 `state/latest-run.json`의 `feeds`에서 피드별 성공 여부를 확인하고, 실패한 피드는 주소를 고치거나 `enabled:false`로 끄세요. 전부 실패하면 실행이 실패로 기록됩니다.
- Google 뉴스 링크는 news.google.com을 거쳐 원문으로 이동합니다.
- 해설은 제목과 짧은 요지만 근거로 합니다. 본문을 읽지 않으므로 세부 내용은 원문을 확인해야 합니다.
