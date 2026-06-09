# 🔬 단백체 분석 실습 강의

## 질량분석 데이터로 펩타이드를 찾아내는 과정

---

## 📋 강의 개요

| 항목 | 내용 |
|------|------|
| 소요 시간 | 30분 ~ 1시간 |
| 대상 | Python 입문자 |
| 플랫폼 | GitHub Codespaces |

## 📚 강의 내용

1. **MS1/MS2 스펙트럼** 구조 이해 및 시각화
2. **FASTA In-silico Digestion** - 단백질을 Trypsin으로 자르기
3. **Candidate 필터링** - ppm tolerance로 후보 좁히기
4. **PSM Score 계산** - fragment ion 매칭으로 정답 찾기
5. **실제 데이터 규모 체감** - 왜 고성능 컴퓨팅이 필요한가

---

## 🚀 시작 방법 (참가자 안내)

### 1단계: 이 페이지에서 Codespace 열기

상단의 초록색 **`<> Code`** 버튼 클릭  
→ **`Codespaces`** 탭 클릭  
→ **`Create codespace on main`** 클릭

### 2단계: 환경 자동 설정 기다리기 (약 1~2분)

모든 패키지가 자동으로 설치됩니다.

### 3단계: 노트북 열기

왼쪽 파일 탐색기에서 `proteomics_lecture.ipynb` 클릭

### 4단계: 첫 번째 셀부터 순서대로 실행

각 셀을 클릭 후 **`Shift + Enter`** 로 실행합니다.

---

## 📦 사용 패키지

```
numpy pandas matplotlib biopython pyteomics
```

---

## 💻 강의자 참고사항

- 모든 데이터는 코드로 시뮬레이션 생성 (별도 파일 불필요)
- 각 셀에 한글 주석 상세 포함
- 섹션별로 독립적으로 실행 가능
