# 단백체 분석 Claude Skills

단백체 분석에서 자주 만드는 그림을 **일관된 스타일**로 빠르게 생성하기 위한 Claude Skills 모음입니다.

## 포함된 스킬

| 스킬 | 명령어 | 용도 |
|------|--------|------|
| volcano-plot | `/volcano-plot` | Volcano plot (차등 발현 시각화) |
| go-bubble | `/go-bubble` | GO enrichment bubble chart |
| km-curve | `/km-curve` | Kaplan-Meier 생존 곡선 |
| string-network | `/string-network` | STRING 단백질 네트워크 |
| heatmap | `/heatmap` | 발현 heatmap |

## 설치 방법 (Claude Code)

```bash
# 스킬 폴더 클론
git clone https://github.com/heeyounh/proteomics-lecture.git
cd proteomics-lecture/skills

# 개인 스킬 폴더로 복사
cp -r volcano-plot go-bubble km-curve string-network heatmap ~/.claude/skills/
```

## 사용 방법

Claude Code 세션에서:

```
/volcano-plot
```

입력하면 Claude가 사용할 데이터 구조를 확인 후 바로 코드를 생성합니다.

## 공통 스타일 규칙

모든 스킬은 아래 스타일을 기본으로 적용합니다:

- 배경: 흰색
- 폰트: Arial
- 해상도: 300 DPI
- 저장 형식: PNG (`bbox_inches='tight'`)
- 한글 폰트: NanumGothic 자동 포함

## 참고

- 강의 자료: `https://github.com/heeyounh/proteomics-lecture`
- Claude Skills 공식 문서: `https://code.claude.com/docs/en/skills`
