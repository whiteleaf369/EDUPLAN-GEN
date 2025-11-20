# LLM 기반 학습지도안 자동 생성 시스템 (RAG + LoRA)

## 1. 개요
본 문서는 RAG와 LoRA를 결합해 교과 학습지도안을 자동으로 작성하는 시스템 설계를 정리하고, LLM 중심 프롬프트 개발만으로 구현하는 절차를 설명한다. 전문 프로그래밍 지식이 없어도 단계별 프롬프트 지침을 따라 전체 파이프라인을 구축할 수 있다.

## 2. 핵심 기술
### 2.1 RAG (Retrieval-Augmented Generation)
- **정의:** 문서·데이터베이스 등 외부 지식을 검색해 생성 전에 LLM에 제공하는 방식.
- **목표:** 교과서, 교육과정, 기존 지도안에서 성취기준·활동 예시를 검색해 반영.
- **예시:** "고2 생명과학1 광합성 수업지도안"을 요청하면 광합성 성취기준 문서를 검색 후 생성.

### 2.2 LoRA (Low-Rank Adaptation)
- **정의:** 대형 언어모델의 일부 가중치만 추가 학습해 개인화 스타일을 반영하는 경량 튜닝 기법.
- **목표:** 교육청 양식·교사별 문체를 반영한 맞춤형 생성.
- **예시:** 서울시교육청 양식 지도안 50개로 LoRA 학습 후 동일 문체로 출력.

### 2.3 LLM 코딩 기반 구현 가능성
- CSV 변환, HuggingFace API 호출, Flowise 워크플로 JSON, Streamlit UI 등을 LLM이 자동 생성 가능.
- 프롬프트 중심으로 코드와 설정을 생성해 노코드·로우코드 수준에서 구축할 수 있음.

## 3. 시스템 구성요소
| 구성요소 | 기술/도구 | 설명 | 구현 난이도 |
| --- | --- | --- | --- |
| 데이터 수집 | Google Drive, Notion | 학습지도안·교육과정 문서 업로드 | 매우 쉬움 (노코드) |
| 검색 모듈 (RAG) | FlowiseAI / Dust.tt / ChatGPT 코드 생성 | LLM 프롬프트로 자동 생성 가능 | 쉬움 |
| 텍스트 생성 (LLM) | GPT-4 / Claude Sonnet | 직접 텍스트 생성 | 매우 쉬움 |
| 모델 튜닝 (LoRA) | HuggingFace AutoTrain | CSV 업로드 후 클릭형 실행 | 쉬움 |
| 인터페이스 | Streamlit / Replit / ChatGPT Code Interpreter | ChatGPT가 UI 코드 자동 생성 | 쉬움 |

## 4. 구현 단계별 세부 계획
### 4.1 데이터 준비
1. 학습지도안 30~100개 수집 후 과목, 학년, 수업주제, 활동, 목표 등을 추출.
2. CSV로 정리: `subject, level, text` 형식 권장.
3. 프롬프트 예시: "아래 폴더 내 텍스트 파일을 하나의 CSV로 정리하는 Python 코드를 작성해줘."

### 4.2 LoRA 학습 (HuggingFace AutoTrain)
1. HuggingFace 가입 후 AutoTrain 접속 → 프로젝트 "Text to Text" 생성.
2. 베이스 모델: Mistral-7B-Instruct 또는 Llama-3-8B 선택.
3. CSV 업로드 → LoRA 옵션 선택 → 학습 시작.
4. ChatGPT에게 AutoTrain 업로드용 CSV 포맷 변환 코드 생성 요청.
5. 학습 완료 후 모델 링크 발급 및 메타데이터 기록.

### 4.3 RAG 구축 (Flowise 또는 LangChain)
1. Flowise에서 워크플로 생성: **User Input → Document Loader(PDF/Text) → Vector Store(Chroma) → Retriever → LLM Node**.
2. 프롬프트 예시: `다음 문서를 참고하여 조건에 맞는 학습지도안을 작성하시오. 조건: {user_input}`.
3. ChatGPT에 Flowise 워크플로 JSON 생성을 요청하거나 LangChain 파이썬 코드를 생성하도록 지시.

### 4.4 RAG + LoRA 결합
1. Flowise LLM 노드에 HuggingFace LoRA 엔드포인트 연결 또는 LangChain에서 `HuggingFaceEndpoint`와 `Chroma`를 결합.
2. 검색 결과를 LoRA 모델 입력 프롬프트에 주입해 개인화된 생성 결과 확보.
3. 응답 형식 템플릿(예: 학습목표, 준비물, 활동, 평가)을 명시해 일관성 유지.

### 4.5 인터페이스 개발
1. Streamlit 또는 Replit에서 단일 페이지 UI 생성: 학년·과목 선택, 주제 입력, 생성 버튼, 미리보기 영역.
2. ChatGPT 코드 생성 기능을 활용해 UI 컴포넌트와 API 호출 코드를 자동 작성.
3. 필요 시 인증/저장을 위해 Google Drive 또는 Notion API 연동.

## 5. 운영 및 배포 지침
- **데이터 버전 관리:** 업로드 데이터와 학습 로그를 폴더 단위로 보관하고 버전 태깅.
- **평가:** 생성된 지도안을 샘플링해 전문가 검토 후 피드백을 LoRA 재학습에 반영.
- **배포:** Replit/Netlify로 프론트 배포, HuggingFace Space 또는 자체 서버로 모델 엔드포인트 제공.
- **보안:** 학생 개인정보가 포함된 자료는 마스킹 후 사용하고, 저장소 접근 권한을 최소화.

## 6. 예상 효과
- 지도안 작성 시간 약 90% 절감.
- 프롬프트 중심 개발로 코딩 부담 최소화.
- 교사별 스타일 반영 및 연구용 데이터 축적 용이.

## 7. 추가 참고 도구 및 다음 단계
- **추가 도구:** Perplexity.ai(자료 검색), Colab + PEFT(LoRA 실험), Netlify/Replit(배포).
- **학습용 튜토리얼:** "ChatGPT만으로 RAG+LoRA 워크플로 완성" 가이드 작성 및 공유.
- **비교 분석:** ChatGPT 기반 vs Flowise 노코드 기반 워크플로 효율성 비교.
- **확장 아이디어:** 논문 주제 제안, 초등학생도 이해할 수 있는 자동화 설명 자료 제작.

---

[^1]: 모든 각주는 한글로 작성되었습니다.
