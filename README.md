# news_maker — 보도자료 3-에이전트 시스템

주제만 주면 보도자료를 작성하고, 스타일 가이드라인에 맞춰 검수·재작성을 반복해 완성도를 높이는 Claude Code 서브에이전트 3종 세트입니다.

## 구성

| 에이전트 | 파일 | 역할 |
|---|---|---|
| `press-release-writer` | `.claude/agents/press-release-writer.md` | 주제를 받아 보도자료 초안을 작성. reviewer의 수정 지시를 받아 재작성도 담당. |
| `guideline-collector` | `.claude/agents/guideline-collector.md` | 보도자료 작성 가이드라인을 수집해 `guidelines/press-release-guidelines.md`를 관리·최신화. |
| `press-release-reviewer` | `.claude/agents/press-release-reviewer.md` | writer의 결과물을 가이드라인 기준으로 검수하고 PASS/REVISE 및 구체적 수정 지시를 냄. |

## 동작 흐름

```
      주제
       │
       ▼
┌─────────────────────┐        가이드라인 참조
│ press-release-writer │◄───────────────────────────┐
└─────────┬────────────┘                             │
          │ 초안                                      │
          ▼                                          │
┌──────────────────────┐   REVISE + 수정 지시목록      │
│ press-release-reviewer├──────────────► writer가 재작성 │
└─────────┬────────────┘                             │
          │ PASS                              guideline-collector
          ▼                              (guidelines/press-release-guidelines.md 관리)
      최종 보도자료
```

1번(writer)이 작성한 결과물을, 2번(guideline-collector)이 관리하는 가이드라인 문서를 기준으로 3번(reviewer)이 검수합니다.
3번이 REVISE 판정과 함께 수정 지시를 내리면 1번이 그 지시를 반영해 다시 작성하고, 다시 3번이 검수하는 과정을 PASS가 나올 때까지 반복합니다.

## 사용 방법

```
/press-release 신제품 A 출시 소식. 회사명: OO테크, 출시일: 2026-09-15, 특징: ...
```

`.claude/commands/press-release.md`에 정의된 오케스트레이션이 writer → reviewer 루프(최대 4회)를 자동으로 돌립니다.
개별 에이전트만 호출하려면 `press-release-writer`, `press-release-reviewer`, `guideline-collector`를 Task/Agent 도구로 직접 지정해도 됩니다.

## 가이드라인 데이터 소스

`data/newswire_style_guide_links.csv`에 뉴스와이어 블로그(`blog.newswire.co.kr`)의 보도자료/기사 작성 관련 글 234편의 제목·링크·카테고리·처리 여부(`fetched`)가 정리되어 있습니다.

`guidelines/press-release-guidelines.md`는 이 목록을 기반으로 한 살아있는 문서입니다. 최초 버전(v0.1)은 이 저장소를 만든 세션의 네트워크 정책상 `blog.newswire.co.kr` 본문을 직접 열 수 없어 **제목만으로 정리한 초안**입니다. 이후 `guideline-collector` 에이전트를 (해당 도메인 접근이 가능한 환경에서) 실행하면 각 링크 본문을 열어 근거와 예시를 보강하고 `fetched` 컬럼을 갱신하면서 문서를 계속 발전시킵니다.

```
가이드라인 최신화해줘
```

라고 요청하면 `guideline-collector`가 아직 처리되지 않은(`fetched=FALSE`) 링크들을 배치로 열어 가이드라인 문서를 보강합니다.

## 디렉터리

```
.claude/agents/       서브에이전트 3종 정의
.claude/commands/      /press-release 오케스트레이션 커맨드
data/                  가이드라인 원본 링크 목록 (CSV)
guidelines/            살아있는 가이드라인 문서
output/                writer가 생성한 보도자료 초안/최종본
```
