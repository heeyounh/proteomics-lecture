---
description: >
  단백체 DE 결과로 publication-quality volcano plot 코드를 생성합니다.
  "volcano plot 그려줘", "화산 그림", "차등 발현 시각화" 등의 요청 시 자동 실행.
---

# Volcano Plot 코드 생성

## 사전 확인

코드 작성 전 사용자에게 아래를 확인:
1. DE 결과 데이터프레임 변수명 (예: `result_df`)
2. log2 fold change 컬럼명 (예: `log2FC`)
3. FDR 또는 p-value 컬럼명 (예: `fdr`, `p_value`)
4. 단백질/유전자 이름 컬럼명 (예: `protein_id`, `gene_norm`)
5. 강조 표시할 단백질 목록 (없으면 자동으로 상위 10개)

## 코드 작성 규칙

### 색상 분류
- FDR < 0.05 AND log2FC > 1 : 빨강 `#E74C3C` → "Up"
- FDR < 0.05 AND log2FC < -1 : 파랑 `#3498DB` → "Down"
- 나머지 : 회색 `#95A5A6` → "ns"

### 기준선
- 수평 점선: FDR = 0.05 (-log10 변환값)
- 수직 점선: log2FC = +1, -1

### 단백질 이름 표시
- adjustText 라이브러리 사용 (겹침 자동 방지)
- 기본 상위 10개 (FDR 기준), 강조 단백질은 ★ 마커

### 스타일 (항상 고정)
- 배경: 흰색
- 폰트: Arial
- 그림 크기: (10, 8) 인치
- 저장: `volcano_plot.png`, 300 DPI, `bbox_inches='tight'`
- 범례: "Up (N개)", "Down (N개)", "ns (N개)" 포함
- 한글 폰트 설정 포함 (NanumGothic)

## 출력 형식

완성된 코드 블록만 출력. 코드 실행 후 Up/Down/ns 단백질 수를 요약 출력하는 print문 포함.
