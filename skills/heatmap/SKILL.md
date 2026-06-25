---
description: >
  단백체 발현 행렬로 hierarchical clustering heatmap 코드를 생성합니다.
  "heatmap", "발현 히트맵", "클러스터링 그려줘", "발현 패턴 시각화" 요청 시 자동 실행.
---

# 단백체 발현 Heatmap 코드 생성

## 사전 확인

코드 작성 전 사용자에게 아래를 확인:
1. 발현 행렬 데이터프레임 변수명 (행: 단백질, 열: 샘플)
2. 표시할 단백질 수 (기본: 상위 50개, FDR 기준)
3. 샘플 그룹 정보 (예: HCC vs Normal) — 컬럼 컬러바용
4. 정규화 방법: z-score (기본) 또는 log2 발현값 그대로
5. 추가 사이드바 필요 여부 (예: FDR 유의성, 그룹 분류)

## 코드 작성 규칙

### 라이브러리
- `seaborn.clustermap` 사용
- z-score 정규화: `z_score=0` (단백질별 행 방향)

### 색상
- 발현: `RdBu_r` colormap (빨강=상향, 파랑=하향)
- vmin/vmax: -2, 2 (z-score 기준)
- 샘플 그룹 컬러바: 상단 (HCC=#E74C3C, Normal=#3498DB 등)

### 클러스터링
- 행(단백질): hierarchical clustering, method='ward', metric='euclidean'
- 열(샘플): hierarchical clustering (그룹 내 패턴 확인)
- dendrogram 표시

### 레이블
- 단백질 이름: 50개 이하면 표시 (fontsize=8), 초과 시 숨김
- 샘플 이름: 표시 (fontsize=9)

### 스타일 (항상 고정)
- 배경: 흰색
- 폰트: Arial
- 그림 크기: (14, 10) 인치
- colorbar 포함
- 저장: `heatmap.png`, 300 DPI, `bbox_inches='tight'`
- 한글 폰트 설정 포함 (NanumGothic)

## 출력 형식

완성된 코드 블록만 출력. 클러스터별 단백질 수 요약 print 포함.
