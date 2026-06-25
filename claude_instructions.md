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
