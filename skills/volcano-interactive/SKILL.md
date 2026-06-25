---
name: volcano-interactive
description: >
  DE 결과 + quant 데이터로 Volcano Plot과 Heatmap을 하나의 HTML에 통합 생성합니다.
  슬라이더로 threshold를 실시간 조정하고, "📊 Heatmap 보기" 버튼으로 현재
  significant 단백질의 heatmap을 즉시 전환해 볼 수 있습니다.
  "인터랙티브 volcano", "volcano heatmap 연동", "significant 단백질 heatmap" 요청 시 실행.
---

# 인터랙티브 Volcano + Heatmap 통합 HTML 생성

## 필요 파일

| 파일 | 내용 | 필수 컬럼 |
|------|------|-----------|
| DE 결과 | parquet 또는 CSV | `gene`, `log2FC_컬럼`, `p_value_컬럼`, `fdr_컬럼` |
| quant 데이터 | 샘플별 abundance | index=샘플명, columns=gene symbol |

quant 데이터 index 형태 확인 필수:
```python
quant.index.tolist()  # ['ctrl_1','ctrl_2','ctrl_3','ffa_1','ffa_2','ffa_3'] 등
```

---

## Step 1. 데이터 준비 및 JSON 변환

```python
import pandas as pd, numpy as np, json

de    = pd.read_parquet('DE결과.parquet')
quant = pd.read_parquet('quant데이터.parquet')

# ── Volcano 데이터 ──────────────────────────────────
de['neglog10p'] = -np.log10(de['p_value_컬럼'].clip(lower=1e-10))

ALL = [
    {
        "x":    round(float(r['log2FC_컬럼']), 4),
        "y":    round(float(r['neglog10p']), 4),
        "gene": str(r['gene_컬럼']),
        "p":    float(r['p_value_컬럼']),
        "fdr":  round(float(r['fdr_컬럼']), 4) if pd.notna(r.get('fdr_컬럼')) else None,
        "ps":   f"{r['p_value_컬럼']:.2e}"
    }
    for _, r in de.iterrows()
]

# ── Quant 데이터 (p < 0.1 후보만 포함해 용량 절약) ──
# p < 0.1 기준: 슬라이더로 조정 가능한 threshold 범위를 넉넉히 커버
cands = set(de[de['p_value_컬럼'] < 0.1]['gene_컬럼'].values)
sample_order = ['ctrl_1','ctrl_2','ctrl_3','ffa_1','ffa_2','ffa_3']  # 실제 index에 맞게 수정
QUANT = {
    g: quant.loc[sample_order, g].clip(-5, 5).round(3).tolist()
    for g in cands if g in quant.columns
}

# ── 통합 JSON ──
SAMPLES_DISPLAY = ['ctrl_1','ctrl_2','ctrl_3','FFA_1','FFA_2','FFA_3']  # 표시용 레이블
DATA = {"all": ALL, "quant": QUANT, "samples": SAMPLES_DISPLAY}
DATA_JS = json.dumps(DATA, ensure_ascii=False)
print(f"DATA: {len(DATA_JS)//1024}KB | volcano: {len(ALL)}개 | quant 후보: {len(QUANT)}개")
```

---

## Step 2. HTML 생성 — 핵심 원칙

### ⚠️ 반드시 지켜야 할 코드 작성 원칙

**원칙 1: JS 코드는 Python f-string 밖에 raw string으로 작성**

```python
# ✅ 올바른 방법 — JS를 r"""...""" 로 분리
JS = r"""
const DATA = JSON.parse(document.getElementById('app-data').textContent);
// { }를 자유롭게 사용 가능, f-string 이스케이프 불필요
function applyThresh() {
    const sig = ALL.filter(d => Math.abs(d.x) >= curLfc && d.p <= curPval);
}
"""

# ❌ 잘못된 방법 — JS를 f-string 안에 넣으면 { } 모두 {{ }} 로 이스케이프해야 함
# → 한 곳이라도 빠지면 JS 구문 오류 → 화면에 아무것도 안 보임
HTML = f"""
<script>
function applyThresh() {{  // {{ }} 가 너무 많아 오류 발생 위험
    const sig = ALL.filter(d => Math.abs(d.x) >= curLfc && d.p <= curPval);
}}
</script>
"""
```

**원칙 2: 데이터는 `<script type="application/json">`에 분리**

```python
# ✅ 데이터와 코드를 분리
HTML = f"""
<script type="application/json" id="app-data">{DATA_JS}</script>
<script>{JS}</script>
"""
# JS에서 읽기: const DATA = JSON.parse(document.getElementById('app-data').textContent);
```

**원칙 3: Chart.js와 Plotly.js 동시 로드 금지**

Plotly.js(2.7MB)와 Chart.js를 함께 로드하면 충돌이 발생할 수 있음.
Heatmap은 **Canvas API로 직접 구현**하거나, Plotly만 사용할 경우 Chart.js를 제거.

```html
<!-- ✅ Chart.js만 사용 (heatmap은 Canvas로 직접) -->
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.3/dist/chart.umd.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/hammerjs@2.0.8/hammer.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/chartjs-plugin-zoom@2.0.1/dist/chartjs-plugin-zoom.min.js"></script>
```

---

## Step 3. HTML 구조 스펙

### 레이아웃 (1280×720px 슬라이드 형식)

```
┌─────────────────────────────────────────────────────────┐
│  [헤더] 제목                          8,167 proteins    │
│  [슬라이더] log2FC >= 1.00  p <= 1.0e-2  [버튼들] [📊]  │
├─────────────────────────────────────────────────────────┤
│  [Volcano 뷰 or Heatmap 뷰]                             │
│                                                         │
│  뷰 전환: 버튼 클릭 → display:none/block으로 교체       │
├─────────────────────────────────────────────────────────┤
│  Up: N  Down: N  ns: N                                  │
└─────────────────────────────────────────────────────────┘
```

### Volcano 뷰 기능

| 기능 | 구현 방법 |
|------|-----------|
| 점 색상 분류 | Increase=`#E74C3C`, Decrease=`#3498DB`, ns=`rgba(148,163,184,0.3)` |
| 기준선 | afterDraw 플러그인으로 수평/수직 점선 |
| 줌/패닝 | chartjs-plugin-zoom (wheel+drag) |
| 커스텀 툴팁 | `<div class="vc-tip">` + onHover 콜백 |
| PNG 저장 | `devicePixelRatio:3` + canvas.toDataURL() + 흰 배경 추가 |
| ns 토글 | `getDatasetMeta(0).hidden` 전환 |

### Heatmap 뷰 기능 (Canvas API)

```javascript
// Z-score 정규화
function zscore(arr) {
    const mean = arr.reduce((a,b) => a+b, 0) / arr.length;
    const std  = Math.sqrt(arr.reduce((a,b) => a+(b-mean)**2, 0) / arr.length) || 1;
    return arr.map(v => Math.max(-3, Math.min(3, (v-mean)/std)));
}

// 색상 매핑 (파랑→흰→빨강, RdBu)
function zColor(z) {
    const t = (z + 3) / 6;  // 0~1로 정규화
    if (t <= 0.5) {
        const u = t * 2;
        return `rgb(${Math.round(33+u*(247-33))},${Math.round(102+u*(247-102))},${Math.round(172+u*(247-172))})`;
    } else {
        const u = (t - 0.5) * 2;
        return `rgb(${Math.round(247+u*(178-247))},${Math.round(247+u*(24-247))},${Math.round(247+u*(43-247))})`;
    }
}

// 단백질 정렬: log2FC 기준 (Decrease → Increase 순서로 직관적)
const sorted = sig.sort((a, b) => a.x - b.x);
```

**Canvas heatmap 요소:**
- 상단: 샘플 그룹 컬러바 (ctrl=파랑, FFA=빨강)
- 왼쪽: 유전자 레이블 (단백질 수에 따라 font-size 자동 조정)
- 오른쪽: log2FC 강도 바 + Z-score 범례
- hover 정보: 별도 구현 안 함 (Canvas tooltip은 복잡), PNG 저장으로 대체

**Canvas 크기 설정 (중요):**
```javascript
// 부모 div 크기에 맞게 canvas 동적 조정
(function resizeCanvas() {
    const c = document.getElementById('hm-canvas');
    const p = c.parentElement;
    function setSize() {
        c.width  = p.clientWidth  || 1100;
        c.height = p.clientHeight || 540;
    }
    setSize();
    window.addEventListener('resize', setSize);
})();
```

### 뷰 전환 JS 패턴

```javascript
function showHeatmap() {
    document.getElementById('volcano-view').classList.add('hidden');
    document.getElementById('heatmap-view').classList.remove('hidden');
    document.getElementById('volcano-btns').style.display = 'none';
    document.getElementById('heatmap-btns').style.display = 'flex';
    renderHeatmap();  // 현재 threshold로 즉시 그리기
}
function showVolcano() {
    document.getElementById('heatmap-view').classList.add('hidden');
    document.getElementById('volcano-view').classList.remove('hidden');
    document.getElementById('heatmap-btns').style.display = 'none';
    document.getElementById('volcano-btns').style.display = 'flex';
    setTimeout(() => vchart && vchart.resize(), 50);  // canvas 리사이즈 필요
}
```

### 슬라이더 연동 (핵심)

```javascript
function onSl() {
    curLfc  = parseFloat(document.getElementById('lfc-sl').value);
    const negP = parseFloat(document.getElementById('pval-sl').value);
    curPval = Math.pow(10, -negP);
    applyThresh();
    // heatmap 뷰가 열려있으면 실시간 업데이트
    if (!document.getElementById('heatmap-view').classList.contains('hidden'))
        renderHeatmap();
}
```

---

## Step 4. HTML 조립 패턴

```python
CSS = """..."""   # 일반 문자열 (f-string 아님)
JS  = r"""..."""  # raw string (f-string 아님) — { } 자유롭게 사용

HTML = f"""<!DOCTYPE html>
<html lang="ko">
<head>
...
<style>{CSS}</style>
</head>
<body>
...HTML 구조...
<script type="application/json" id="app-data">{DATA_JS}</script>
<script>
(function resizeCanvas() {{
  // ← HTML 조립 부분만 f-string이므로 {{ }} 사용
}})();
</script>
<script>{JS}</script>
</body>
</html>"""

with open('volcano_interactive.html', 'w', encoding='utf-8') as f:
    f.write(HTML)
print(f"저장 완료: {len(HTML)//1024}KB")
```

---

## 디자인 토큰

```
배경:       #ffffff / 외부 #0f172a
헤더:       #1e3a5f (네이비)
악센트:     #4fd1c5 (틸)
컨트롤 bg:  #f9f8f5
Increase:  #E74C3C
Decrease:  #3498DB
ns:        rgba(148,163,184,0.3)
ctrl 그룹: #3498DB
FFA 그룹:  #E74C3C
폰트:      Noto Sans KR, JetBrains Mono
슬라이드:  1280×720px, transform:scale() 반응형
```

---

## 자주 발생하는 오류와 해결법

| 증상 | 원인 | 해결 |
|------|------|------|
| 화면에 아무것도 안 보임 | JS f-string `{` `}` 이스케이프 누락 → SyntaxError | JS 코드를 `r"""..."""` raw string으로 분리 |
| 화면에 아무것도 안 보임 | Chart.js + Plotly.js 동시 로드 충돌 | Plotly 제거, Canvas API로 heatmap |
| `replace(/\.$/'')` 오류 | f-string 안 정규식 작은따옴표 충돌 | 정규식 부분만 f-string 밖으로 이동 |
| heatmap에 단백질 없음 | quant 후보가 p<0.1로 너무 좁음 | 후보 기준을 p<0.2로 완화 |
| canvas 높이 0 | 부모 div에 높이 미지정 | `c.height = p.clientHeight \|\| 540` |
| vchart.resize() 오류 | 뷰 전환 후 canvas 크기 변경 필요 | `setTimeout(() => vchart.resize(), 50)` |
