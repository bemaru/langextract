# 프로젝트 철학, 가치, 기술 스택

## 프로젝트 철학

LangExtract의 핵심 철학은 **"LLM 출력을 신뢰할 수 있게 만드는 구조화"**. 단순히 텍스트를 추출하는 게 아니라, **어디서 왔는지 증명하고, 형식을 보장하고, 품질을 사전 검증**하는 것.

### 추구하는 가치

| 가치 | 구현 방식 |
|------|----------|
| **Source Grounding** | 모든 추출에 char_interval + alignment_status 부여. 원문 위치 정확 추적 |
| **DX 우선** | `lx.extract()` 3줄로 완전한 파이프라인. 점진적 복잡성 증가 |
| **신뢰성** | Schema constraints(Gemini controlled generation) + prompt validation |
| **확장성** | Entry point 기반 plugin 시스템. 코어 수정 없이 새 provider 추가 |
| **투명성** | AlignmentStatus enum으로 매칭 신뢰도 표시. Interactive HTML 시각화 |

### 설계 원칙

```
"Many examples better than long instructions"
  → Few-shot learning 중심. 좋은 예제가 긴 프롬프트보다 낫다

"Transparency over black boxes"
  → 모든 추출에 신뢰도(EXACT/FUZZY/LESSER) 첨부. 사용자가 판단

"Cost-aware by default"
  → extraction_passes=3은 3배 비용. 명시적 경고
```

## 해결하는 문제

### 기존 LLM 추출의 한계

| 문제 | 상세 |
|------|------|
| **출처 추적 불가** | 추출된 정보가 원문 어디서 왔는지 알 수 없음 |
| **장문서 누락** | Long context에서도 "Needle in Haystack" 문제 발생 |
| **구조 불일관** | LLM 출력 형식이 일관성 없어 파싱 실패 빈번 |
| **도메인 적응 어려움** | 각 도메인별 모델 재학습 불가능 |

### LangExtract의 차별점

| 기존 대안 | LangExtract |
|----------|------------|
| 단순 LLM 호출 + JSON 파싱 | Source grounding + schema 강제 + prompt validation |
| Context window 초과 시 정보 손실 | 자동 청킹 + 다중 패스 + 병렬 처리 |
| 모델별 코드 수정 필요 | Plugin 시스템으로 model_id만 변경 |
| 결과 검증 수동 | Interactive HTML 시각화 |

### 대상 사용자

- 의료/법률 정보 처리 (정확한 출처 추적 필수)
- 대규모 문서 자동 구조화 (엔터프라이즈)
- ML 엔지니어/데이터 과학자 (few-shot으로 빠른 파이프라인 구축)

## 기술 스택과 선택 이유

### 핵심 의존성

```toml
google-genai>=1.39.0        # Gemini API (주 제공자, controlled generation 지원)
absl-py>=1.0.0              # Google 표준 로깅/플래깅
aiohttp>=3.8.0              # 비동기 HTTP (배치 API)
pydantic>=1.8.0             # 데이터 검증
regex>=2023.0.0             # 유니코드 지원 정규식
google-cloud-storage>=2.14  # GCS (배치 API 캐싱)
```

### 선택적 의존성

```toml
openai = ["openai>=1.50.0"]  # pip install langextract[openai]
```

### 기술 선택의 트레이드오프

| 기술 | 선택 이유 | 트레이드오프 |
|------|----------|------------|
| **Gemini 기본** | Google 프로젝트. controlled generation으로 스키마 강제 가능 | Google 종속성 인상 (but OpenAI/Ollama도 지원) |
| **difflib** | Python 표준 라이브러리. LCS 기반 정확한 시퀀스 매칭 | 대규모 텍스트에서 O(n·m) 비용 |
| **regex** (not re) | `\p{L}` 유니코드 패턴으로 다국어 지원 | 표준 re보다 무거움 |
| **absl-py** | Google 내부 표준 로깅. 관찰성 우수 | 비Google 개발자에게 진입장벽 |
| **Entry Points** | setuptools 표준. 패키지 설치만으로 플러그인 등록 | 발견 메커니즘 이해 필요 |
| **OpenAI 선택적** | 필요 없으면 설치 안 해도 됨 | 설치 단계 추가 |

## 배울 점

- **"신뢰할 수 있는 AI 출력"이라는 프레이밍**: 단순 추출이 아니라 "출처 증명 + 구조 보장 + 품질 검증"으로 가치를 정의
- **Few-shot > 긴 프롬프트**: 좋은 예제 3-5개가 장황한 지시보다 효과적이라는 설계 원칙
- **비용 투명성**: extraction_passes, max_workers 등 비용에 영향을 미치는 파라미터를 명시적으로 경고

## 적용 아이디어

- **EDR AI에서 Source Grounding 적용**: 보안 이벤트 분석 시 AI 판단의 근거를 원본 로그의 정확한 위치로 추적. 분석관이 "왜 이 판단?"을 클릭 한 번으로 확인
- **Prompt Validation 패턴**: AI 기능의 프롬프트 품질을 사전 검증하는 메커니즘. 잘못된 예제가 프로덕션에 배포되기 전에 차단
- **비용 투명성 제공**: EDR AI의 LLM 호출에도 "이 분석은 토큰 N개 사용, 예상 비용 $X" 표시
