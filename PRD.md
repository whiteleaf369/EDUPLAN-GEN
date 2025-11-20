# LLM 기반 학습지도안 자동 생성 시스템 (RAG + LoRA)

## 1. 개요
- **목표:** RAG와 LoRA를 결합해 교과 학습지도안을 자동으로 생성하는 시스템을 설계한다. GPT 기반 코드 생성만으로 데이터 정제, 모델 학습, 검색 파이프라인, UI까지 구축 가능함을 전제로 한다.
- **대상:** 프로그래밍 경험이 적은 교사·연구자도 프롬프트 중심 워크플로로 손쉽게 활용할 수 있는 노코드/로우코드 구현.
- **기대효과:** 지도안 작성 시간 90% 이상 단축, 교사별 문체·양식 반영, 교육 연구용 데이터 축적.

## 2. 핵심 기술
### 2.1 RAG (Retrieval-Augmented Generation)
- **정의:** LLM 생성 전에 외부 지식을 검색·주입하여 응답 정확도와 근거성을 높이는 방식.
- **용도:** 교과서, 교육과정, 기존 지도안 등에서 관련 문단을 검색해 생성 프롬프트에 포함.
- **예시:** “고2 생명과학1 광합성” 요청 시 성취기준·실험 절차 등 관련 문서를 자동 검색 후 LLM에 전달.

### 2.2 LoRA (Low-Rank Adaptation)
- **정의:** 대형 언어모델의 일부 가중치만 저랭크로 미세 조정해 개인화 스타일을 반영하는 방법.
- **용도:** 교육청 양식, 학교별 서식, 교사 고유 문체, 평가 방식 등을 반영한 맞춤형 모델 구축.
- **예시:** “서울시교육청 양식 지도안 50개”로 LoRA 학습 후 동일한 문체·서식을 자동 생성.

### 2.3 LLM 코딩 기반 실현성
- **근거:** ChatGPT/Claude/Gemini 등의 LLM이 CSV 변환, HuggingFace API 호출, Flowise 워크플로 JSON, Streamlit UI 코드 등을 자동 생성할 수준에 도달.
- **결론:** 전문 개발자 없이 프롬프트 엔지니어링만으로 전체 파이프라인을 구축할 수 있음.[^llm]

## 3. 시스템 구성 요소
| 구성요소 | 기술/도구 | 설명 | 구현 난이도 |
| --- | --- | --- | --- |
| 데이터 수집 | Google Drive, Notion | 학습지도안·교육과정 문서 업로드 | 매우 쉬움 (노코드) |
| 검색 모듈 (RAG) | FlowiseAI / Dust.tt / ChatGPT 생성 코드 | 문서 로더→벡터스토어→리트리버→LLM | 쉬움 |
| 텍스트 생성 (LLM) | GPT-4 / Claude Sonnet | 지도안 본문 생성 | 매우 쉬움 |
| 모델 튜닝 (LoRA) | HuggingFace AutoTrain | CSV 업로드 후 클릭형 LoRA 학습 | 쉬움 |
| 인터페이스 | Streamlit / Replit / ChatGPT Code Interpreter | 웹 UI 또는 챗봇 UI | 쉬움 |

## 4. 구현 단계
### 4.1 데이터 준비
1. 학습지도안 샘플 30~100개 수집(과목, 학년, 주제 다양화).
2. 교과목·학년·주제·세부 활동·평가 요소를 열로 정리한 CSV 생성.
3. LLM에게 “폴더 내 텍스트를 단일 CSV로 정리” 코드를 요청해 자동 변환.

### 4.2 LoRA 학습 (HuggingFace AutoTrain)
1. AutoTrain 프로젝트 생성 후 **Text to Text** 선택.
2. **Mistral-7B-Instruct** 또는 **Llama-3-8B** 모델 지정.
3. CSV 업로드 → **LoRA** 옵션 선택 → 학습 실행.
4. 완료된 모델 링크를 기록해 추론 엔드포인트 또는 HuggingFace Hub에서 사용.

### 4.3 RAG 구축 (Flowise 또는 LangChain)
1. 워크플로: **User Input → Document Loader (PDF/Text) → Vector Store (Chroma) → Retriever → LLM Node**.
2. 프롬프트 템플릿 예시:
   ```
   다음 문서를 참고하여 아래 조건에 맞는 학습지도안을 작성하시오.
   조건: {user_input}
   ```
3. Flowise에서는 위 노드를 UI로 연결하고, ChatGPT에게 JSON 구성을 요청해 자동 세팅.
4. LangChain 사용 시, LoRA 모델을 LLM Wrapper로 설정하고 ChromaDB Retriever와 연결.

### 4.4 RAG + LoRA 결합
1. Flowise LLM 노드에 HuggingFace LoRA 엔드포인트 연결(토큰/엔드포인트 URL 설정).
2. LangChain 파이프라인 예시: `HuggingFacePipeline`(LoRA) → `RetrievalQA` 체인.
3. 평가: RAG On/Off 및 LoRA On/Off 조합으로 BLEU/ROUGE 또는 인간 평가를 수행해 효과 검증.

### 4.5 배포 및 운영
1. **UI**: Streamlit/Replit로 간단한 입력 폼과 결과 미리보기 제공.
2. **호스팅**: Netlify(정적) 또는 Replit/Render(동적) 사용, 학습된 LoRA 모델은 HuggingFace Inference API로 호출.
3. **모니터링**: 사용자 피드백 수집 폼, 프롬프트 개선 히스토리, 로그(쿼리·검색문서·응답) 저장.
4. **보안/저작권**: 교육자료 라이선스 확인, 학생 개인정보 비포함 여부 검증.

## 5. 상세 실행 계획 (주차별)
| 주차 | 목표 | 산출물 |
| --- | --- | --- |
| 1주차 | 데이터 수집·정제, CSV 자동화 스크립트 확보 | 정제된 CSV, 변환 스크립트, 데이터 관리 가이드 |
| 2주차 | LoRA 학습 및 모델 검증 | 학습된 LoRA 모델 링크, 예시 출력, 평가 리포트 초안 |
| 3주차 | RAG 워크플로 구축 | Flowise/LangChain 설정 파일, 검색 품질 샘플 로그 |
| 4주차 | RAG+LoRA 통합 및 UI 구현 | 통합 파이프라인, Streamlit/Replit UI, 데모 영상/스크린샷 |
| 5주차 | 평가·배포·운영 가이드 작성 | 평가 결과 리포트, 운영/모니터링 체크리스트 |

## 6. 기술 스택 및 설정 세부사항
- **모델**: LoRA 대상은 Mistral-7B-Instruct 또는 Llama-3-8B; 프롬프트 길이를 고려해 4k~8k 컨텍스트 지원 모델 권장.
- **벡터스토어**: Chroma DB(내장), 대규모 데이터 시 Weaviate/PGVector 대체 가능.
- **임베딩**: InstructorXL 또는 OpenAI text-embedding-3-large(비공개 환경이면 로컬 임베딩 모델 사용).
- **하이퍼파라미터(LoRA 예시)**: r=16, alpha=32, dropout=0.1, 학습률 1e-4, epoch 3~5.
- **평가**: ROUGE-L/ChrF 등 자동 지표 + 교사 도메인 전문가 휴리스틱 평가 병행.

## 7. 리스크 및 완화
- **데이터 편향**: 특정 학교·교사 문체에 치우칠 위험 → 다양한 샘플 수집, 데이터 균형 검토.
- **저작권/보안**: 교육 자료의 라이선스 확인, 학생 개인정보 포함 여부 점검 → 배포 전 수동 검열 체크리스트 운영.
- **모델 품질 저하**: 프롬프트 길이 과다 또는 검색 품질 저하 → 검색 상위 k 튜닝, 프롬프트에 근거 문단 강조.
- **비용 관리**: Inference API 호출 비용 증가 → 캐싱, 오프라인 배치 생성, 경량 모델 우선 사용.

## 8. 다음 단계
- Flowise 워크플로 JSON과 LangChain 샘플 스크립트를 LLM에 요청해 확보.
- Streamlit UI 초안 생성 후 학습된 LoRA 엔드포인트와 연동.
- 테스트 데이터셋으로 RAG On/Off, LoRA On/Off ablation을 진행해 품질 차이를 보고.

[^llm]: LLM이 생성하는 코드를 검토할 때, 입력 데이터의 개인정보 및 저작권 위반 여부를 사람이 확인해야 한다.
