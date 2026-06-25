---
description: >
  GO enrichment 분석 결과로 Metascape 스타일 bubble chart 코드를 생성합니다.
  "GO 분석", "GO bubble", "enrichment 시각화", "경로 분석 그려줘" 요청 시 자동 실행.
---

# GO Enrichment Bubble Chart 코드 생성

## 사전 확인

코드 작성 전 사용자에게 아래를 확인:
1. GO enrichment 결과 데이터프레임 변수명 (예: `go_results`)
2. GO term 이름 컬럼 (예: `name`, `term_name`)
3. p-value 컬럼 (예: `p_value`)
4. gene count 컬럼 (예: `intersection_size`, `gene_count`)
5. gene ratio 컬럼 (없으면 `intersection_size / term_size`로 계산)
6. 표시할 term 수 (기본 20개)

gprofiler 결과가 아닌 경우 컬럼명을 맞게 매핑해서 코드 작성.

## 코드 작성 규칙

### 데이터 전처리
- p-value 오름차순 정렬 후 상위 N개 선택
- GO term 이름이 너무 길면 40자로 truncate
- y축: term 이름 (p-value 낮은 것이 위로)

### 버블 속성
- x축: Gene Ratio
- y축: GO term 이름
- 버블 크기: gene count에 비례 (기본 배율 30)
- 버블 색상: -log10(p_value), `Reds` colormap (값 높을수록 진한 빨강)
- 버블 테두리: 회색 0.3px, alpha=0.85

### 범례 / 컬러바
- 오른쪽에 colorbar 추가, 라벨: `-log10(p-value)`
- 버블 크기 범례: 최소/중간/최대 gene count 3개

### 스타일 (항상 고정)
- 배경: 흰색
- 폰트: Arial 10pt
- 격자선: x축 방향 연한 회색 점선
- 그림 크기: (11, 8) 인치
- 저장: `go_bubble.png`, 300 DPI, `bbox_inches='tight'`
- 한글 폰트 설정 포함 (NanumGothic)

## 출력 형식

완성된 코드 블록만 출력.
