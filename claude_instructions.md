# Claude 단백체 분석 실습 — 지침 (Instructions)

> claude.ai → Projects → 지침(Instructions)에 이 내용을 붙여넣으세요.

---

## [분석 환경]

- 언어: Python 3, Jupyter Notebook
- 주요 라이브러리: pandas, numpy, scipy, matplotlib, seaborn, gseapy
- 설명은 **한국어**로, 코드는 **영어**로 작성

---

## [그림 출력 규칙]

- 배경: 흰색
- 폰트: Arial (한글 포함 시 NanumGothic)
- 저장: `300 DPI, PNG, bbox_inches='tight'`
- 그림 크기 기본값: `figsize=(10, 7)`

---

## [한글 폰트 — 항상 코드 상단에 포함]

```python
import matplotlib.font_manager as fm
import matplotlib.pyplot as plt

fm.fontManager.addfont('/usr/share/fonts/truetype/nanum/NanumGothic.ttf')
plt.rcParams['font.family'] = 'NanumGothic'
plt.rcParams['axes.unicode_minus'] = False
```

---

## [데이터 정보]

### 강의 데이터 로드 (인터넷 필요)

```python
import pandas as pd

BASE = "https://raw.githubusercontent.com/heeyounh/proteomics-lecture/main/data/"

de    = pd.read_csv(BASE + "cptac_hcc_DE.csv", index_col=0)
quant = pd.read_csv(BASE + "cptac_hcc_quant.csv", index_col=0)
```

### DE 결과 컬럼 (`cptac_hcc_DE.csv`)

| 컬럼 | 설명 |
|------|------|
| `gene` | Gene symbol (index) |
| `log2FC` | log₂ Fold Change (C1 / C2) |
| `p_value` | Raw p-value |
| `FDR` | Benjamini-Hochberg FDR |
| `C1_mean` | Cluster 1 평균 발현량 (log₂) |
| `C2_mean` | Cluster 2 평균 발현량 (log₂) |

### Quant 데이터 컬럼 (`cptac_hcc_quant.csv`)

| 컬럼 | 설명 |
|------|------|
| `gene` | Gene symbol (index) |
| `C1_s1` ~ `C1_s5` | Cluster 1 샘플 5개 |
| `C2_s1` ~ `C2_s5` | Cluster 2 샘플 5개 |

---

## [분석 주제]

- 데이터: CPTAC HCC (간세포암) 코호트 단백체
- 비교: 분자 서브타입 **Cluster 1 vs Cluster 2**
- 샘플 수: C1 × 5, C2 × 5 (강의용)
- 주요 생물학적 맥락: 지방산 대사, 퍼옥시솜, HCC 분자 서브타입

---

## [인터랙티브 HTML 시각화 스펙]

> "인터랙티브 volcano", "volcano html", "interactive plot" 요청 시 반드시 아래 스펙을 따를 것.

### 필수 라이브러리 (Plotly.js 사용 금지)

```html
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.3/dist/chart.umd.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/hammerjs@2.0.8/hammer.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-zoom@2.0.1/dist/chartjs-plugin-zoom.min.js"></script>
```

### 슬라이드 레이아웃

- 크기: **1280×720px** 고정, `transform:scale()` 반응형 (viewport 맞춤)
- 슬라이드 내부 배경: 흰색 `#ffffff`
- 외부 배경: `#0f172a` (다크)
- 헤더: `background:#1e3a5f`, 악센트 바 `#4fd1c5`
- 컨트롤 패널: `background:#f9f8f5`, 하단 테두리 `#e8e7e4`
- 색상: Increase `#E74C3C`, Decrease `#3498DB`, ns `rgba(148,163,184,0.3)`

### Volcano 뷰 필수 기능

- 슬라이더 2개: log₂FC (0.5–3.0), p-value (-log10 스케일 1.0–4.0)
- 슬라이더 변경 시 기준선 + 색상 분류 실시간 업데이트 (`animation:false`)
- 기준선: afterDraw 플러그인으로 수평/수직 점선
- 커스텀 툴팁 div (hover 시 gene명, FC, p-value, FDR)
- 버튼: PNG 저장 (`devicePixelRatio:3`), 줌 초기화, ns 토글
- 버튼: **📊 Heatmap**, **🔬 GO 분석** (뷰 전환)
- 하단 통계: Increase N개, Decrease N개, ns N개

### Heatmap 뷰 (Canvas API — Plotly 사용 금지)

- significant 단백질을 log2FC 기준 정렬 (Decrease → Increase)
- Z-score 정규화 후 Canvas로 직접 그리기
- 색상: 파랑(`#2166AC`) → 흰색 → 빨강(`#B2182B`)
- 상단 샘플 그룹 컬러바, 오른쪽 log2FC 강도 바 + Z-score 범례
- 슬라이더 threshold와 연동 (뷰 열린 상태에서 슬라이더 변경 시 자동 업데이트)

### GO 분석 뷰

- Enrichr API 직접 호출: `https://maayanlab.cloud/Enrichr/`
- Gene sets: `GO_Biological_Process_2023`, `KEGG_2021_Human` (탭 전환)
- Canvas API로 bubble chart 구현
- 버블: x=Combined Score, 크기=overlap gene count, 색상=FDR 강도
- 버블 클릭 시 gene list 팝업 표시
- 로딩 스피너 포함

### 코드 작성 규칙 (중요)

```
1. JS 코드는 Python f-string 안에 넣지 말 것
   → JS를 r"""...""" raw string으로 분리 후 {JS} 로 삽입
2. 데이터는 <script type="application/json" id="app-data"> 에 분리
   → JS에서 JSON.parse(document.getElementById('app-data').textContent) 로 읽기
3. Chart.js와 Plotly.js 동시 로드 금지
4. Canvas 크기는 ResizeObserver로 부모에 맞게 동적 조정
```

---

## [분석 흐름]

```
1. 데이터 로드 → 2. Volcano plot → 3. Heatmap → 4. GO enrichment
```

### Volcano plot 권장 기준

```python
# significant 기준
fc_thresh  = 1.0   # |log2FC| >=
pval_thresh = 0.01  # p_value <=

sig = de[(de['p_value'] <= pval_thresh) & (de['log2FC'].abs() >= fc_thresh)]
```

### GO enrichment 권장 코드

```python
import gseapy as gp

sig_genes = sig.index.tolist()

enr = gp.enrichr(
    gene_list=sig_genes,
    gene_sets=['GO_Biological_Process_2023', 'KEGG_2021_Human'],
    organism='human',
    cutoff=0.05
)

results = enr.results[enr.results['Adjusted P-value'] < 0.05]
results = results.sort_values('Adjusted P-value')
```
