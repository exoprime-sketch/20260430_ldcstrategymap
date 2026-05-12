# src/App.js 인벤토리 (0단계)

생성일: 2026-05-12  
측정: 28,345줄 / 1016 KB (UTF-8)

---

## 1. 핵심 데이터 객체 위치

| 식별자 | 시작줄 | 끝줄(추정) | 설명 |
|---|---|---|---|
| VIETNAM_MEKONG_PILOT_DATA | 65 | 353 | 베트남 메콩 델타 파일럿 전체 데이터 |
| strategyEvidence | 354 | 498 | 베트남 전략 근거 상수 |
| executionFeasibility | 499 | 531 | 베트남 실행 타당성 상수 |
| suitabilityLogic | 532 | 558 | 베트남 적합성 논리 상수 |
| pilotStatus | 559 | 574 | 베트남 파일럿 진행 상태 상수 |
| VIETNAM_PIPELINE_MOCK_DATA | 575 | 675 | 베트남 파이프라인 목 데이터 |
| TECHNOLOGY_TAXONOMY | 2668 | 2935 | 38대 기후기술 분류 체계 |
| RECOMMENDATIONS | 4781 | 5340 | 10개국 협력 대상 추천 배열 |
| PARTNER_DIRECTORY | 5499 | 5865 | 기관별 파트너 디렉터리 |
| COUNTRY_DATA_ENRICHMENTS | 6003 | 6969 | 국가별 데이터 보강 |
| COUNTRY_SUPPLEMENTAL_ENRICHMENTS | 6970 | 7201 | 국가별 보조 데이터 |
| COUNTRY_FRESHNESS_ENRICHMENTS | 7202 | 7733 | 국가별 최신성 데이터 |
| ENHANCED_RECOMMENDATIONS | 8169 | 8190 | 보강된 추천 배열 |
| NORMALIZED_ENHANCED_RECOMMENDATIONS | 8191 | 8194 | 기술명 정규화된 최종 배열 |
| STRATEGY_PRESETS | 8209 | 8284 | 7개 전략 프리셋 정의 |
| CTIS_SITE_LINKS | 26725 | 26733 | CTIS 사이트 링크 |
| CTIS_COUNTRY_CATALOG | 26734 | 26983 | CTIS 국가 카탈로그 |
| CTIS_VISIBLE_SEED_DATA | 26984 | 27208 | CTIS 시드 데이터 |
| OFFICIAL_LINK_WHITELIST | import | - | ../official_link_whitelist_asean.json |

---

## 2. 최상위 함수·컴포넌트 요약 (주요 항목)

### 보안 함수 (절대 보존)
| 함수/상수 | 줄 | 역할 |
|---|---|---|
| UI_ALLOWED_EXTERNAL_HOSTS | 1504 | UI에서 허용된 외부 호스트 Set |
| FETCH_ALLOWED_EXTERNAL_HOSTS | 1571 | 네트워크 요청 허용 호스트 Set |
| MAX_JSON_RESPONSE_BYTES | 1578 | JSON 응답 최대 크기 (1.5 MB) |
| SAFE_STORAGE_KEY_RE | 1579 | localStorage 키 패턴 검증 정규식 |
| ensureExternalUrl | 847 | URL 허용 여부 확인 후 반환 |
| isAllowedExternalUrl | 1601 | 허용 호스트 집합 대조 |
| assertAllowedExternalUrl | 1612 | 위반 시 throw |
| sanitizeSpreadsheetValue | 1635 | formula injection 방지 |
| sanitizeFilenamePart | 1673 | 파일명 특수문자 제거 |
| safeJsonParse | 1398 | JSON.parse 실패 시 fallback |
| escapeHtml | 1357 | HTML 이스케이프 |
| getCspNonce | 1495 | CSP nonce 조회 |

### URL 상태 관리
| 함수 | 줄 | 파싱/생성 항목 |
|---|---|---|
| parsePlatformUrlState | 1128 | view, tech, country, purpose, tab, focus, rec |
| buildPlatformShareUrl | 1153 | 위 항목 → URL 조합 |

> **위험**: `preset`, `demo` 파라미터가 parsePlatformUrlState에 없음 → `?preset=mekong`, `?demo=1` URL 미동작

### 핵심 UI 컴포넌트
| 컴포넌트 | 줄 |
|---|---|
| AppShell | 23662 |
| AppErrorBoundary | 26663 |
| ReviewTabErrorBoundary | 20272 |
| ReviewPanelContent | 20693 |
| ReviewTabGuideCard | 13970 |
| DesktopBrowsePanel | 16109 |
| MapCanvas | 10909 |
| DownloadFallbackModal | 22672 |
| MobileBottomNav | 22417 |
| MobileSheet | 22500 |
| LandingPage | 15037 |

---

## 3. 외부 CDN 의존성

| 라이브러리 | URL | 줄 | 로드 실패 fallback |
|---|---|---|---|
| Tailwind CSS | https://cdn.tailwindcss.com | 791 | useTailwindCDN → tailwindReady false 시 스타일 미적용 |
| MapLibre GL CSS | https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.css | 23665 | 지도 스타일 깨짐 |
| MapLibre GL JS | https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.js | 23669 | 지도 비표시 |
| XLSX | https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js | 23674 | Excel 다운로드 미동작 |

---

## 4. 한글 식별자 잔여 여부

**0건** — 이전 커밋(21d6680)에서 8종 모두 ASCII 변환 완료.  
코드 내 한글은 사용자 노출 문자열에만 존재함.

---

## 5. 알려진 런타임 위험 지점

1. **OFFICIAL_LINK_WHITELIST** (line 61): JSON 파일 부재 시 빌드 실패 → placeholder 주석 추가됨(21d6680), 실제 파일 존재 확인 필요
2. **parsePlatformUrlState** (line 1128): `preset`, `demo` 파라미터 미파싱 → `?preset=mekong`, `?demo=1` URL 비동작
3. **MapLibre CDN 로드 실패**: useExternalScript 오류 시 UI 배너 없음
4. **XLSX CDN 로드 실패**: Excel 다운로드 경로에서 `window.XLSX` 미존재 시 오류 메시지 없음
5. **ENHANCED_RECOMMENDATIONS 정렬**: `sort((a,b) => a.id - b.id)` — id가 숫자임을 가정(베트남 pilot은 id:3001)
6. **rec=1 URL**: id:1 이 RECOMMENDATIONS[0](멕시코·소노라·CCUS) → 통과

---

## 6. URL 파라미터 의미 역추적

`parsePlatformUrlState` 기준:

| 파라미터 | 타입 | 의미 | 예시값 |
|---|---|---|---|
| view | string | 화면 모드 | "landing", "platform" |
| tech | string | 기술 필터 | "탄소 포집 및 저장 (CCUS)" |
| country | string | 국가 필터 | "멕시코" |
| purpose | string | 목적 필터 | "ODA", "R&D 실증" |
| tab | string | 상세 패널 탭 | "overview", "recommendations", "funding", "partners", "sources" |
| focus | string | 지도 포커스 모드 | "region", "country" |
| rec | string(→int) | 추천 항목 id | "1" (멕시코 CCUS) |
| preset | ⚠️ 미파싱 | 전략 프리셋 키 | "mekong", "oda-screening" |
| demo | ⚠️ 미파싱 | 시연 모드 | "1" |

---

## 7. 빌드 상태

```
npx esbuild src/App.js --loader:.js=jsx --bundle=false --outfile=/tmp/check.js
→ 통과 (1.1 MB, ~78 ms, 오류 없음)
```
