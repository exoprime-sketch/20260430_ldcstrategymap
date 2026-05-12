# 데이터 흐름 매핑 (1단계)

생성일: 2026-05-12

---

## 1. 추천 레코드 생성 흐름

```
RECOMMENDATIONS[0..9] (makeRec, id 1~10)
  └─ applyCountryDataEnrichment
       ├─ buildStrategyEvidence   → rec.strategyEvidence
       ├─ buildExecutionFeasibility → rec.executionFeasibility
       ├─ buildLocalPartners      → rec.localPartners
       ├─ buildSuitabilityLogic   → rec.suitabilityLogic
       └─ getPilotStatus          → rec.pilotStatus

VIETNAM_MEKONG_PILOT_DATA
  └─ buildRecFromVietnamPilot → VIETNAM_PILOT_REC (id: 3001)

ENHANCED_RECOMMENDATIONS = [...filtered, VIETNAM_PILOT_REC].sort(id)

NORMALIZED_ENHANCED_RECOMMENDATIONS = map(normalizeRecommendationTech)
  └─ LEGACY_TO_UPDATED_TECH_MAP 역매핑 적용
```

---

## 2. 화면 흐름 ↔ 데이터 소비 매핑

### 상세 패널 탭 구조 (ReviewPanelContent, L20693)

| 탭 key | 탭 라벨 | 소비 컴포넌트 | 소비 데이터 |
|---|---|---|---|
| overview | 핵심 요약 | ReviewTabGuideCard | rec.strategyEvidence.summary, rec.reasons, rec.scores |
| recommendations | 근거·전략 | StrategyEvidenceCard, ExecutionFeasibilityCard, SuitabilityLogicCard, VietnamPilotCard | rec.strategyEvidence.drivers, rec.strategyEvidence.sourceData, rec.executionFeasibility |
| funding | 재원·실행 | FundingExecutionPanel | rec.executionFeasibility.financeChannels, pipelineData, PARTNER_DIRECTORY |
| partners | 파트너 | LocalPartnersCard, InternationalCooperationReadinessCard | rec.localPartners, buildInternationalCooperationAssessment() |
| sources | 출처·링크 | OfficialDocumentShelfCard, EvidenceCoverageSummaryCard | collectEvidenceLinks(), OFFICIAL_LINK_WHITELIST |

---

## 3. 점수 항목 정의·산출식·코드 일치 여부

### 3-1. rec.scores 직접 저장 항목

| 항목 | 필드 | 저장 위치 | 산출 방법 |
|---|---|---|---|
| 데이터 충족률 | scores.coverage | makeRec (RECOMMENDATIONS 각 항목) | 직접 정의 (0~100) |
| 근거 신뢰도 | scores.reliability | makeRec | 직접 정의 (0~100) |
| 결측 복원력 | scores.resilience | makeRec | 직접 정의 (0~100) |
| 종합 목적 적합도 | scores.feasibility | 미저장 또는 직접 정의 | 직접 또는 0.35×cov+0.35×rel+0.30×res |

**일치 여부**: SCORE_METHOD(L3401) 텍스트 설명과 buildEvidenceMetrics(L18176) 산출식 일치.  
단, feasibility가 재계산인 경우 가중치(35%/35%/30%)는 코드에 명시되지 않고 SCORE_METHOD 설명에만 있음 → buildPracticalMetrics에서 실제 산출 확인 필요.

### 3-2. buildEvidenceMetrics 산출 (scores가 없을 때 fallback)

```
coverage  = scores.coverage OR (52 + data_items×8 + region_rows×6 + pipeline×5 - missing×5)
feasibility = scores.feasibility OR (46 + finance_channels×9 + pipeline×7 + partner_bonus)
```
→ 직접 scores 필드가 있으면 그것을 우선 사용. 없으면 데이터 보유량 기반 추정.  
**한계**: 추정값과 직접값이 혼용될 수 있어 점수 기준이 레코드마다 다름.

### 3-3. buildPracticalMetrics 산출 (추가 4개 지표)

```
documentReadiness = rel×0.45 + directLinks×8 + policyDocs×5 + detailLinks×4
financeReadiness  = rel×0.30 + pipeline×9 + financeChannels×7 + partnerLinks×6
partnerAccess     = resilience×0.40 + partners×10 + partnerLinks×8
policyAlignment   = coverage×0.40 + policyDocs×10 + resilience×0.20 + pipeline×4
```

→ METRIC_FRAMEWORK(L3409) 정의와 코드 연산이 **대응됨**. 다만 문서상 정의는 정성적이고  
  코드는 계량적이므로 tooltip에서 "추정 산출값" 명시 보강 권장.

---

## 4. strategyEvidence / executionFeasibility / suitabilityLogic / pilotStatus 소비처

| 상수명 | 소비 위치 | 용도 |
|---|---|---|
| strategyEvidence | sanitizeStrategyEvidence(L3329), buildStrategyEvidence 기본값 | 베트남 파일럿의 근거 데이터 구조 원형 |
| executionFeasibility | sanitizeExecutionFeasibility(L3377), buildExecutionFeasibility 기본값 | 베트남 실행 타당성 구조 원형 |
| suitabilityLogic | buildSuitabilityLogic 내부 (L5947) | 기술·지역 적합성 논리 구조 원형 |
| pilotStatus | getPilotStatus 내부 (L5984) | 파일럿 진행 상태 구조 원형 |

---

## 5. VIETNAM_MEKONG_PILOT_DATA → 화면 흐름 대응

```
VIETNAM_MEKONG_PILOT_DATA
  ├─ .evidence → strategyEvidence.drivers 로 매핑
  ├─ .funding  → executionFeasibility.financeChannels 로 매핑
  ├─ .partners → localPartners 로 매핑
  ├─ .risks    → suitabilityLogic.risks 로 매핑
  ├─ .followUp → pilotStatus.nextActions 로 매핑
  └─ .sources  → strategyEvidence.sourceData 로 매핑
```

화면 흐름(목적→대상지→기술→근거→재원·파트너→실행)과 1:1 대응 **확인됨**.
