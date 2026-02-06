# Superpowers Analysis

obra/Superpowers 리포지토리를 **openclaw 관점**에서 분석합니다.

## 목표
- Claude Code의 Superpowers 특성을 추상화
- openclaw 고유 특성(cron, sub-agent, peekaboo, Slack 토론)을 활용한 적용 계획 도출

## 분석 방법론
[openclaw_analysis](https://github.com/2ndlifeinc/openclaw_analysis) 접근법을 차용:
- 문서 우선 (Document-First)
- 핵심 패턴 추출
- 폴더별 분리된 분석 문서

## 구조
```
analysis/
├── 00-superpowers-analysis-plan.md  # 전체 계획
├── 01-*/                             # 주제별 분석
│   ├── 01-diagram.md
│   ├── 02-details.md
│   ├── 03-application-plan.md
│   └── 04-implementation-prompt.md
```

## 진행 상황
- [ ] 분석 계획 수립
- [ ] obra/Superpowers clone
- [ ] 핵심 패턴 추출
- [ ] openclaw 적용 계획

## 참고
- [obra/Superpowers](https://github.com/obra/Superpowers)
- [openclaw_analysis](https://github.com/2ndlifeinc/openclaw_analysis)
