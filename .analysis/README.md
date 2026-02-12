# LangExtract 분석

> LLM 기반 비정형 텍스트→구조화 정보 추출 라이브러리. Source Grounding으로 모든 추출의 원문 위치를 정확히 추적

| 항목 | 내용 |
|------|------|
| 저장소 | [google/langextract](https://github.com/google/langextract) |
| 언어 | Python |
| 라이선스 | Apache-2.0 (상업적 사용/수정/배포 자유, 특허 보호) |
| 분석일 | 2026-02-12 |

## 핵심 인사이트

1. **Source Grounding이 핵심 차별점** — 모든 추출에 원문의 정확한 char/token 위치를 매핑. difflib 기반 정확 매칭 → fuzzy 매칭 → 부분 매칭 3단계 전략으로, LLM이 원문을 약간 변형해도 정렬 가능
2. **조건부 프롬프트 검증(Prompt Validation)** — 실행 전에 few-shot example이 실제 텍스트와 정렬되는지 자동 검증. OFF/WARNING/ERROR 3단계로 사용자 제어. "Fail fast, fail loudly" 철학
3. **Plugin 기반 Provider 시스템** — Entry Points + Registry Pattern + Lazy Loading으로 코어 수정 없이 새 LLM 제공자 추가. PEP 562 lazy module loading으로 import 비용 최소화
4. **Example-Driven Schema 자동 생성** — few-shot example에서 JSON Schema를 자동 추론하여 Gemini의 controlled generation에 적용. 구조화된 출력을 강제하여 파싱 실패율 극감
5. **다중 패스 추출로 Recall 향상** — 같은 텍스트를 N번 분석하여 누락된 엔티티를 보충. 첫 패스 우선 + non-overlapping 병합 전략. 비용-Recall 트레이드오프를 사용자에게 투명하게 공개

## 문서

- [overview.md](./overview.md) — 프로젝트 철학, 가치, 기술 스택
- [core-logic.md](./core-logic.md) — 핵심 로직 흐름, 알고리즘/패턴
- [architecture.md](./architecture.md) — 아키텍처, 모듈 구조, 설계 패턴
