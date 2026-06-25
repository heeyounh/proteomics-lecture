---
description: >
  생존 분석 데이터로 Kaplan-Meier 생존 곡선 코드를 생성합니다. lifelines 라이브러리 사용.
  "KM curve", "생존 분석", "Kaplan-Meier", "생존 곡선 그려줘" 요청 시 자동 실행.
---

# Kaplan-Meier 생존 곡선 코드 생성

## 사전 확인

코드 작성 전 사용자에게 아래를 확인:
1. 메타데이터 데이터프레임 변수명 (예: `meta_df`)
2. 생존 기간 컬럼명 (예: `OS_month`, `survival_months`)
3. 이벤트 발생 컬럼명 (예: `OS_event`, `status`) — 1=사망/이벤트, 0=censored
4. 그룹 컬럼명 (예: `cluster`, `group`) 및 그룹 값 목록
5. 분석 종류: Overall Survival (OS) / Disease-Free Survival (DFS)

## 코드 작성 규칙

### 라이브러리
- `lifelines` 라이브러리 사용 (KaplanMeierFitter)
- log-rank test: `lifelines.statistics.multivariate_logrank_test`

### 그룹 색상 (그룹 수에 따라)
- 2그룹: 파랑 `#2196F3`, 주황 `#FF9800`
- 3그룹: 파랑 `#2196F3`, 주황 `#FF9800`, 초록 `#4CAF50`
- 4그룹 이상: matplotlib tab10 colormap

### 그림 구성
- 주 패널: KM 곡선 + 95% 신뢰구간 (alpha=0.15)
- 그림 안 텍스트: log-rank p-value (소수점 3자리)
- 하단 패널: at-risk table (각 시점별 남은 환자 수)
- 범례: "그룹명 (n=N, events=E)" 형식

### 스타일 (항상 고정)
- 배경: 흰색
- 폰트: Arial
- y축: Survival Probability (0~1), x축: 시간 단위 (월 또는 일)
- 그림 크기: (10, 7) 인치
- 저장: `km_curve.png`, 300 DPI, `bbox_inches='tight'`
- 한글 폰트 설정 포함 (NanumGothic)

## 출력 형식

완성된 코드 블록만 출력. 각 그룹의 중앙 생존 기간을 print하는 코드 포함.
