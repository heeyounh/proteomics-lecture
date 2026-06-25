---
description: >
  단백질 목록으로 STRING 기반 단백질 상호작용 네트워크 시각화 코드를 생성합니다.
  "STRING 네트워크", "단백질 네트워크", "PPI 그려줘", "상호작용 시각화" 요청 시 자동 실행.
---

# STRING 단백질 네트워크 코드 생성

## 사전 확인

코드 작성 전 사용자에게 아래를 확인:
1. 단백질/유전자 목록 (리스트 또는 데이터프레임)
2. STRING edge 데이터 소스:
   - 로컬 parquet 파일 (예: `string_edges.parquet`)
   - 또는 STRING API로 직접 조회 (인터넷 필요)
3. combined_score 필터 기준 (기본: 700)
4. 노드 색상에 쓸 값: log2FC, FDR 등 (없으면 단일 색상)
5. 노드 크기에 쓸 값: -log10(FDR) 등 (없으면 고정 크기)

## 코드 작성 규칙

### 네트워크 구성
- `networkx` 라이브러리 사용
- 입력 단백질 목록 내부 edge만 추출 (서브네트워크)
- combined_score 필터 적용 후 고립 노드 제거 옵션 제공

### 레이아웃
- spring layout, `seed=42` 고정 (재현성 확보)
- `k` 값: 노드 수에 따라 자동 조정 (노드 많을수록 크게)

### 노드 속성
- 크기: -log10(FDR) × 200 (기본 500으로 고정 가능)
- 색상: log2FC → `RdBu_r` colormap (없으면 `#4A90D9` 단색)
- 테두리: 검정 0.5px

### Edge 속성
- 굵기: `combined_score / 1000 × 3` (0.5~3 범위)
- 색상: `#BBBBBB`, alpha=0.6

### 레이블
- 노드 위에 단백질 이름 (fontsize=10, Arial, 검정)
- 겹침 최소화를 위해 위치 offset 적용

### 스타일 (항상 고정)
- 배경: 흰색, 축 없음
- colorbar 포함 (노드 색상이 FC인 경우)
- 그림 크기: (12, 10) 인치
- 저장: `string_network.png`, 300 DPI, `bbox_inches='tight'`
- 한글 폰트 설정 포함 (NanumGothic)

## 출력 형식

완성된 코드 블록만 출력. 네트워크 통계 (노드 수, edge 수, 평균 degree) print 포함.
