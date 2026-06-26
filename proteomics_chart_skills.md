# 단백체 분석 시각화 스킬 모음
> **claude.ai Projects → 파일**에 이 파일을 업로드하세요.  
> 아래 스킬 중 요청에 맞는 것을 자동으로 참조합니다.

---

## 스킬 1 · Volcano Plot (정적 PNG)

**트리거**: "volcano plot", "화산 그림", "차등 발현 시각화"

### 사전 확인
- DE 결과 데이터프레임 변수명 / log2FC 컬럼 / FDR 컬럼 / gene 이름 컬럼
- 강조 단백질 목록 (없으면 상위 10개 자동)

### 색상 분류
- FDR < 0.05 AND log2FC > 1 → 빨강 `#E74C3C` (Up)
- FDR < 0.05 AND log2FC < -1 → 파랑 `#3498DB` (Down)
- 나머지 → 회색 `#95A5A6` (ns)

### 고정 스타일
- 기준선: 수평 점선(FDR=0.05), 수직 점선(±1)
- 단백질 이름: adjustText로 겹침 방지, 상위 10개
- 그림 크기 (10, 8), Arial, 300 DPI, `volcano_plot.png`
- 범례: "Up (N개)", "Down (N개)", "ns (N개)"
- 한글 폰트: NanumGothic 설정 포함

---

## 스킬 2 · 인터랙티브 Volcano + Heatmap + GO (HTML)

**트리거**: "인터랙티브 volcano", "volcano html", "heatmap GO 연동"

### 핵심 원칙 (반드시 준수)
- 라이브러리: **Chart.js 4.4.3** + hammerjs + chartjs-plugin-zoom (CDN)
- **Plotly.js 사용 금지**
- JS 코드: Python f-string 안에 직접 쓰지 말 것 → `r"""..."""` raw string으로 분리 후 삽입
- 데이터: `<script type="application/json" id="app-data">` 에 넣고 `JSON.parse()`로 읽기

### 슬라이드 레이아웃
- 크기: 1280×720px 고정, `transform:scale()` 반응형
- 슬라이드 내부 배경: 흰색 / 외부: `#0f172a`
- 헤더: `background:#1e3a5f`, 악센트 바 `#4fd1c5`
- 컨트롤 패널: `background:#f9f8f5`

### Volcano 뷰 필수 기능
- 슬라이더 2개: log₂FC (0.5–3.0), p-value (-log10 1.0–4.0)
- 실시간 기준선(afterDraw 플러그인), 커스텀 툴팁 div
- 버튼: PNG 저장(`devicePixelRatio:3`), 줌 초기화, ns 토글
- 버튼: **📊 Heatmap**, **🔬 GO 분석** (뷰 전환)
- 색상: Increase `#E74C3C`, Decrease `#3498DB`

### Heatmap 뷰 (Canvas API — Plotly 금지)
- significant 단백질 log2FC 기준 정렬
- Z-score 정규화, 파랑(`#2166AC`) → 흰색 → 빨강(`#B2182B`)
- 샘플 그룹 컬러바, log2FC 강도 바, Z-score 범례

### GO 분석 뷰
- Enrichr API: `https://maayanlab.cloud/Enrichr/`
- Gene sets: `GO_Biological_Process_2023`, `KEGG_2021_Human` (탭 전환)
- Canvas bubble chart: x=Combined Score, 크기=overlap count, 색=FDR
- 버블 클릭 → gene list 팝업, 로딩 스피너

### 자주 발생하는 오류
| 오류 | 원인 | 해결 |
|------|------|------|
| 화면이 비어 있음 | JS `{ }` 이스케이프 누락 | JS는 `r"""..."""` 사용 |
| Plotly+Chart 충돌 | 동시 로드 | Chart.js만 사용 |
| Canvas 높이 0 | ResizeObserver 미적용 | `p.clientHeight \|\| 540` 폴백 |
| GO 결과 없음 | significant 단백질 < 3개 | threshold 완화 |

---

## 스킬 3 · GO Enrichment Bubble Chart

**트리거**: "GO 분석", "GO bubble", "enrichment 시각화", "경로 분석 그려줘"

### 사전 확인
- GO enrichment 결과 데이터프레임 / term 이름 컬럼 / p-value 컬럼 / gene count 컬럼
- gene ratio 컬럼 (없으면 `intersection_size / term_size`로 계산)
- 표시할 term 수 (기본 20개)

### 버블 속성
- x축: Gene Ratio / y축: GO term 이름 (p-value 낮은 것이 위)
- 버블 크기: gene count × 30
- 버블 색상: -log10(p_value), `Reds` colormap
- 버블 테두리: 회색 0.3px, alpha=0.85

### 고정 스타일
- colorbar: `-log10(p-value)`, 버블 크기 범례 3개
- 배경 흰색, Arial 10pt, x축 격자 점선
- 그림 크기 (11, 8), 300 DPI, `go_bubble.png`
- 한글 폰트: NanumGothic

---

## 스킬 4 · 발현 Heatmap

**트리거**: "heatmap", "발현 히트맵", "클러스터링 그려줘", "발현 패턴 시각화"

### 사전 확인
- 발현 행렬 (행: 단백질, 열: 샘플) / 표시할 단백질 수 (기본 50개)
- 샘플 그룹 정보 / 정규화 방법 (z-score 기본)

### 코드 작성 규칙
- `seaborn.clustermap`, z_score=0 (단백질별 행 방향)
- 색상: `RdBu_r`, vmin=-2, vmax=2
- 클러스터링: 행/열 모두 ward + euclidean
- 단백질 이름: 50개 이하 표시(fontsize=8), 초과 시 숨김
- 그림 크기 (14, 10), 300 DPI, `heatmap.png`
- 한글 폰트: NanumGothic

---

## 스킬 5 · Kaplan-Meier 생존 곡선

**트리거**: "KM curve", "생존 분석", "Kaplan-Meier", "생존 곡선 그려줘"

### 사전 확인
- 메타데이터 데이터프레임 / 생존 기간 컬럼 / 이벤트 컬럼 (1=사망, 0=censored)
- 그룹 컬럼 / 분석 종류 (OS / DFS)

### 코드 작성 규칙
- `lifelines.KaplanMeierFitter`, log-rank: `multivariate_logrank_test`
- 그룹 색상: 2그룹(파랑/주황), 3그룹(파랑/주황/초록), 4개↑ tab10
- 95% 신뢰구간 alpha=0.15
- 그림 안 log-rank p-value 표시
- 하단 at-risk table
- 범례: "그룹명 (n=N, events=E)"
- 그림 크기 (10, 7), Arial, 300 DPI, `km_curve.png`
- 한글 폰트: NanumGothic

---

## 스킬 6 · STRING 단백질 네트워크

**트리거**: "STRING 네트워크", "단백질 네트워크", "PPI 그려줘", "상호작용 시각화"

### 사전 확인
- 단백질/유전자 목록
- STRING edge 데이터: 로컬 `string_edges.parquet` 또는 STRING API
- combined_score 필터 기준 (기본 700)
- 노드 색상(log2FC 등) / 노드 크기(-log10(FDR) 등)

### 코드 작성 규칙
- `networkx`, 입력 단백질 간 서브네트워크만 추출
- spring layout, `seed=42` 고정
- 노드 크기: -log10(FDR) × 200 (기본 500)
- 노드 색상: log2FC → `RdBu_r` (없으면 `#4A90D9`)
- edge 굵기: `combined_score / 1000 × 3`
- colorbar 포함 (FC 색상인 경우)
- 그림 크기 (12, 10), 300 DPI, `string_network.png`
- 한글 폰트: NanumGothic

---

## 스킬 7 · 인터랙티브 STRING PPI (Cytoscape.js)

**트리거**: "인터랙티브 PPI", "cytoscape PPI", "STRING 인터랙티브 네트워크", "그룹별 PPI 슬라이드"

### 사전 확인
- STRING edge 파일: `string_edges.parquet` (컬럼: protein1, protein2, combined_score, organism_code)
- 실제 정량값: `tumor_quant.parquet` (샘플 × 단백질, UniProt accession)
- 샘플 메타: cluster 컬럼 포함 (cluster_km_k3 등)
- DE 결과: UniProt accession index, log2FC, FDR 컬럼
- 클러스터 수 및 명칭 (예: C1=대사형, C2=면역형)

### 출력 구조
- HTML 파일 1개 (슬라이드 N장, 클러스터별 각 1장)
- 1280×720px 슬라이드 형식
- 각 슬라이드: 왼쪽 cy.js 네트워크 + 오른쪽 범례/정보 패널

### 노드 스타일 규칙
- **색상**: 클러스터별 실제 정량값 Z-score → RdBu_r 색상 (파랑=낮음, 빨강=높음)
  ```python
  cluster_mean = quant[cluster_samples].mean()
  z = (cluster_mean - quant.mean()) / quant.std()
  # z → hex 색상 변환 (vmin=-2, vmax=2)
  ```
- **크기**: -log10(FDR) 비례, seed 단백질은 더 크게 + 테두리 강조
- **레이블**: gene symbol, seed 단백질은 ★ 표시

### Edge 스타일 규칙 (STRING confidence 구간별)
| score 구간 | 색상 | 굵기 | 의미 |
|-----------|------|------|------|
| ≥ 900 | `#0e59a2` (진파랑) | 두꺼움 | Very high |
| 700–900 | `#22863a` (초록) | 중간 | High |
| 500–700 | `#e36209` (주황) | 얇음 | Medium |

### 레이아웃 규칙 — Python preset (★ 확정 방식)

**cytoscape.js 내장 layout 사용 금지** (cose/fcose 모두 겹침 문제 발생)
→ **Python networkx에서 좌표 계산 → cytoscape preset layout**으로 전달

```python
# 1) 컴포넌트 분리 (연결 vs 단독)
comps = sorted(nx.connected_components(G), key=len, reverse=True)
connected = [c for c in comps if len(c) > 1]
isolated  = [list(c)[0] for c in comps if len(c) == 1]

# 2) 연결 컴포넌트: 왼쪽 영역에 spring_layout
CW, CH = 1020, 590        # cy-wrap 실제 크기 (px)
main_w = CW * 0.68        # 왼쪽 68% 할당
scale  = min(main_w, CH) / 2.5   # ★ 화면 기반 scale
sub_pos = nx.spring_layout(sub, k=None, seed=42,
                           iterations=600, weight='weight', scale=scale)
# 중심을 왼쪽 영역 중앙으로 오프셋
for nd, (x, y) in sub_pos.items():
    pos[nd] = (main_w/2 + x, CH/2 - y)

# 3) 단독 노드: 오른쪽 영역에 격자 배치
solo_x_start = main_w + 30
NODE_STEP    = min(80, (CW - solo_x_start - 20) / max(1, ceil(len(isolated)/6)))
cols = max(1, int((CW - solo_x_start - 20) // NODE_STEP))
for i, nd in enumerate(isolated):
    pos[nd] = (solo_x_start + (i%cols)*NODE_STEP + NODE_STEP/2,
               60 + (i//cols)*NODE_STEP + NODE_STEP/2)
```

```javascript
// cytoscape: preset layout (Python 계산 좌표 사용)
layout: { name: 'preset', fit: true, padding: 28 }
```

### 노드 크기 규칙 (★ 확정값)
```javascript
// seed 단백질: 30 + min(neglogfdr*3, 14) → 30~44px
// neighbor:    18 + min(neglogfdr*3, 14) → 18~32px
'width':  e => (e.data('is_seed') ? 30 : 18) + Math.min(e.data('neglogfdr')||1 * 3, 14),
'height': e => (e.data('is_seed') ? 30 : 18) + Math.min(e.data('neglogfdr')||1 * 3, 14),
'font-size': e => e.data('is_seed') ? 11 : 9,
```

### 슬라이드 구조 규칙
- `display:none` **금지** → `visibility:hidden` 사용 (cytoscape 크기 인식 필요)
- `minZoom:0.3, maxZoom:4` 설정

### ★ 합본 통합 필수 패턴 (반드시 준수)

합본에서 PPI가 안 보이는 가장 흔한 원인: `get_slides()`가 `.slide` div만 추출하고 JS가 빠짐

**해결**: `window.initPPI()`로 export하여 합본 window.load에서 호출

```javascript
/* ── 1. 데이터 변수 (전역) ── */
var _PPI_C1 = /* C1 elements JSON */;
var _PPI_C2 = /* C2 elements JSON */;
var _cy1 = null, _cy2 = null;

/* ── 2. 헬퍼 함수 ── */
function _makeCy(cid, elements, seedBorder) { ... }
function _bindInfo(cy, pfx) { ... }

/* ── 3. ★ 핵심: window.initPPI()로 export ── */
window.initPPI = function() {
  if (_cy1 || _cy2) return;  /* 중복 초기화 방지 */
  _cy1 = _makeCy('cy1', _PPI_C1, '#7f1d1d');
  _cy2 = _makeCy('cy2', _PPI_C2, '#1e3a8a');
  /* 노드 수, 버튼 onclick 등 설정 */
};

/* ── 4. standalone 전용 네비게이션 (합본에서는 실행 안 됨) ── */
(function() {
  var allSlides = document.querySelectorAll('#wrapper > .slide');
  if (allSlides.length > 5) return;  /* ★ 합본이면 건너뜀 */
  /* 네비게이션 코드 + window.load에서 initPPI() + go(0) */
})();
```

**합본 생성 시**:
```python
# 1. PPI script를 id로 추출
def get_ppi_script(path):
    with open(path) as f: html = f.read()
    m = re.search(r'<script id="ppi-init-script">(.*?)</script>',
                  html, re.DOTALL)
    return m.group(1) if m else ''

# 2. 합본 <script> 태그에 PPI script 포함
HTML = f"""
<script>
{ppi_script}           /* ← PPI 초기화 코드 */
/* 합본 네비게이션 */
window.addEventListener('load', () => {{
  scale();
  if (typeof initPPI === 'function') initPPI();  /* ← 합본에서 PPI 초기화 */
  go(0);
}});
</script>
"""
```

### 코드 작성 금지 사항
- fcose, cola 등 외부 플러그인 CDN 사용 금지 (로드 실패 위험)
- cytoscape 내장 cose/random layout 사용 금지 (겹침 발생)
- cytoscape 컨테이너에 `display:none` 사용 금지
- `window.addEventListener('load', initCy)` 단독 사용 금지 → 반드시 `window.initPPI()`로 export
- script id 없이 작성 금지 → `<script id="ppi-init-script">` 로 합본에서 추출 가능하게
