---
description: 주제를 받아 press-release-writer → press-release-reviewer 루프를 돌려 가이드라인을 통과한 보도자료를 완성한다.
---

사용자가 `/press-release <주제 및 정보>` 형태로 이 명령을 실행했습니다. 다음 순서로 진행하세요.

1. **가이드라인 최신 상태 확인**: `guidelines/press-release-guidelines.md`가 존재하는지 확인한다. 없거나 사용자가 최신화를 요청했다면 먼저 `guideline-collector` 서브에이전트를 호출해 가이드라인을 준비/갱신한다.
2. **초안 작성**: `press-release-writer` 서브에이전트를 호출해, 사용자가 준 주제와 정보로 보도자료 초안을 작성하게 한다 (신규 작성 모드).
3. **검수**: 방금 나온 초안을 `press-release-reviewer` 서브에이전트에게 전달해 검수를 요청한다.
4. **반복 (최대 4회)**:
   - 검수 결과가 `PASS`이면 5번으로 이동.
   - `REVISE`이면, reviewer가 낸 "발견된 문제" 목록 전체를 `press-release-writer`에게 그대로 전달하며 수정 모드로 재작성을 요청한다. 재작성된 초안을 다시 `press-release-reviewer`에게 검수 요청한다.
   - 4회 반복 후에도 REVISE가 나오면 루프를 멈추고, 반복적으로 지적된 핵심 쟁점을 요약해 사용자에게 보고하고 다음 지시를 기다린다.
5. **완료 보고**: PASS된 최종본의 파일 경로를 사용자에게 알리고, 몇 차례 수정을 거쳤는지, 어떤 지점이 고쳐졌는지 간단히 요약한다.

각 서브에이전트는 `Agent` 도구로 `subagent_type`에 해당 이름(`press-release-writer`, `press-release-reviewer`, `guideline-collector`)을 지정해 호출한다. 서브에이전트를 호출할 때는 이전 단계의 산출물(초안 전문 또는 검수 결과 전문)을 프롬프트에 그대로 포함해 전달한다 — 서브에이전트는 이 대화의 이전 맥락을 보지 못한다.
