# CPTAC HCC 강의용 데이터셋

CPTAC (Clinical Proteomic Tumor Analysis Consortium) HCC (간세포암) 코호트에서
분자적 서브타입 C1 vs C2 비교 분석 결과입니다.

## 파일 설명

### `cptac_hcc_DE.csv`
Differential Expression 분석 결과 (limma 기반)

| 컬럼 | 설명 |
|------|------|
| gene | Gene symbol (HGNC) |
| log2FC | log₂ Fold Change (C1 / C2) |
| p_value | Raw p-value |
| FDR | Benjamini-Hochberg adjusted p-value |
| C1_mean | Cluster 1 평균 발현량 (log₂ IRS-normalized) |
| C2_mean | Cluster 2 평균 발현량 |
| C1_n | Cluster 1 샘플 수 |
| C2_n | Cluster 2 샘플 수 |

### `cptac_hcc_quant.csv`
샘플별 단백질 발현량 (강의용, 상위 significant 단백질 포함)

| 컬럼 | 설명 |
|------|------|
| gene | Gene symbol |
| C1_s1 ~ C1_s5 | Cluster 1 샘플 5개 |
| C2_s1 ~ C2_s5 | Cluster 2 샘플 5개 |

## 사용 기준 (권장)

- Volcano plot 기본 threshold: |log2FC| ≥ 1.0, p-value ≤ 0.01
- GO enrichment: FDR < 0.05 → 278개 GO term

## 출처

- 원본: CPTAC HCC proteomics cohort (PDC000197)
- 단백질 식별: UniProt Human proteome
- 발현 정규화: IRS (Internal Reference Scaling)
- 분자 서브타입: k-means clustering (C1/C2/C3)
