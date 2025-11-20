# 데이터 정제 파이프라인 기술 설치 가이드

## 목적
자동 정제 파이프라인을 실행하기 위해 필요한 필수 기술 스택과 설치 절차를 정리했습니다. 이전에 정의한 RAG/LoRA 기반 데이터 정제 규칙과 자동화 파이프라인을 실제 환경에서 재현할 때 참고하십시오.

## 기본 환경
- OS: Ubuntu 22.04 LTS 이상 (Debian 계열 호환)
- 권장 사양: 8 vCPU / 16GB RAM 이상, 디스크 여유 50GB 이상
- 네트워크: 공인 인터넷 접근(패키지 설치 및 모델 다운로드 용도)

## 시스템 패키지 설치
```bash
sudo apt update
sudo apt install -y build-essential python3.11 python3.11-venv python3.11-dev \
    git wget curl unzip jq poppler-utils ghostscript tesseract-ocr fonts-nanum \
    pandoc libxml2-dev libxslt1-dev libpq-dev
```
- `poppler-utils`: PDF → text 변환에 필요
- `tesseract-ocr` + `fonts-nanum`: 스캔본 OCR 및 한글 글꼴 처리
- `ghostscript`: HWP → PDF 렌더링, PDF 정규화
- `pandoc`: 다양한 문서 형식 변환

## 파이썬 가상환경 및 패키지 설치
```bash
python3.11 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip wheel
pip install poetry
poetry config virtualenvs.create false
poetry init --no-interaction --name edupplan-cleaning --dependency "pydantic>=2" \
    --dependency "pandas>=2" --dependency "numpy>=1.26" --dependency "fasttext-wheel" \
    --dependency "sentence-transformers>=3" --dependency "unstructured[local-inference]>=0.15" \
    --dependency "pdfplumber>=0.11" --dependency "pymupdf>=1.23" \
    --dependency "tika>=2" --dependency "langchain>=0.2" --dependency "chromadb>=0.5"
poetry install
```
- `unstructured[local-inference]`: 한글 문서 레이아웃 보존 추출
- `pdfplumber`, `pymupdf`: PDF 파서 이중화로 품질 확보
- `fasttext-wheel`: 언어 감지 및 노이즈 필터링
- `sentence-transformers`, `chromadb`: 벡터 임베딩 + 로컬 벡터 DB

## HWP 처리 옵션
- 리눅스 환경에서 한글(HWP) 파일을 처리해야 한다면 다음 중 하나를 선택합니다.
  1. HWP → PDF: `hwpx2pdf` 또는 서버형 한/글 뷰어 컨테이너 활용 후 위 PDF 파이프라인 재사용
  2. HWP → 텍스트: `pyhwp`(라이선스 확인) 또는 외부 변환 API를 호출하여 `.txt/.md`로 수집
- 변환 결과는 `pandoc`으로 Markdown 통합 후 정제 파이프라인에 투입합니다.

## 품질·보안 설정 체크리스트
1. **언어·인코딩 표준화**: UTF-8 유지, 혼합 인코딩 시 `iconv`로 강제 변환[^1]
2. **PII 마스킹**: 정규식 + `presidio`/`re2`를 조합하여 학생·교사 실명을 마스킹[^2]
3. **중복 제거**: `datasketch`의 MinHash LSH로 유사도 0.85 이상인 문단 병합[^3]
4. **저작권 메타데이터**: 출처 URL, 발행년도, 라이선스 유형을 YAML frontmatter에 기록[^4]

## 검증 및 배포
- **단위 검증**: 샘플 50개 문서를 대상으로 추출 품질, OCR 정확도, PII 누락 여부를 수작업으로 확인
- **CI 스크립트**: `make lint`(flake8/ruff), `make test`(pytest)로 파이프라인 함수 품질 보증
- **도커화**: 위 패키지 집합을 Dockerfile로 이미지화해 재현성을 확보하고, Airflow/Prefect에 배포
- **모니터링**: 실패 로그를 S3/GCS에 적재하고, Slack/Webhook 알림으로 파이프라인 상태를 추적

## 트러블슈팅 노트
- OCR 정확도가 낮으면 `tesseract-ocr-script-hangul` 패키지 추가 설치 후 `--oem 1 --psm 6` 옵션을 적용
- PDF가 손상되었을 때는 `qpdf --repair`로 복구 후 재처리
- 메모리 부족 시 `unstructured`를 페이지 단위 스트리밍 모드로 실행

---

[1]: UTF-8 강제 변환은 다국어 혼합 문서에서 발생하는 깨짐 문제를 예방하기 위한 필수 단계입니다.
[2]: 이름·전화번호 등 개인정보 노출을 막아 교육데이터 활용 시 법적 리스크를 줄입니다.
[3]: 유사 문단 병합은 임베딩 DB 용량을 줄이고 검색 품질을 높이는 데 기여합니다.
[4]: 출처 기록은 저작권 준수와 후속 데이터 감사 시 추적 가능성을 확보하기 위해 필요합니다.
