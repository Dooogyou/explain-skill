# explain-skill — explain v1.1 공개 검토용 초안

> 이 스킬은 Andrej Karpathy의 포스트에서 아이디어를 얻었습니다. Andrej Karpathy나 ASD와는 관련이 없으며, 그들이 승인하거나 보증한 것이 아닙니다. 공식 ASD-STE100 규격 준수를 주장하지 않습니다.

쉬운 글을 기본으로 하고, 비교에는 표, 구조·흐름에는 작은 정적 도식을 쓰도록 안내하는 Agent Skill이다. 짧은 답보다 원문의 수치·조건·예외·불확실성을 보존하는 설명을 목표로 한다.

**2026-10-14 기준 설치 대상은 v1.1이다.** v1.1은 개선 후보 c02를 승격한 버전이다. 원본 v1과 후보 c01은 Claude에서 “~일 때만” 같은 조건을 “~이면 반드시”로 바꾸거나 “열람만 가능” 같은 제한을 “알 수 없음”으로 약하게 바꿔 T02·T03 의미 보존 시험에 실패했다. c02는 미리 정한 반복 시험 기준을 넘었다. 핵심 문제 25회 중 22회 통과했고, 처음 보는 두 문제에서는 10회 중 9회 통과했다(같은 문제에서 v1과 스킬 미적용은 0/6). **그래도 실패가 남아 있고 표본이 작으므로 검증 완료판이나 안정성이 입증된 버전으로 표시하지 않는다.**

## 포함된 파일

| 파일 | 용도 |
| --- | --- |
| [skills/explain/SKILL.md](skills/explain/SKILL.md) | v1.1 스킬(후보 c02 승격). 설치 대상 |
| [validation/v1-to-v1.1.diff](validation/v1-to-v1.1.diff) | 원본 v1에서 v1.1로 바뀐 부분 |
| [examples/before-after.md](examples/before-after.md) | 작성자가 만든 설명 예시 3개. 실제 실행·성능 측정 결과가 아님 |
| [validation/README.md](validation/README.md) | 검증 결과·실패 이력·미검증 범위 |
| [validation/cases.md](validation/cases.md) | 기본 질문 목록과 실제 추가 시험의 구분 |
| [validation/evidence/](validation/README.md#공개한-입력응답-근거) | T02·T03의 원본 v1 및 c01 입력·응답 원문, 원본 v1 추가 검증의 실패 사례 입력·응답 원문 |
| [validation/evidence/c02-stage1/](validation/README.md#후보-c02--v11로-승격-2026-10-14) | c02 반복 시험 1단계의 입력, 37개 응답 원문 전체, 블라인드 판정 |
| [validation/candidate-c01.diff](validation/candidate-c01.diff) | 실패한 후보 c01의 변경 두 항목. 검토·재현용이며 설치 파일에 적용되지 않음 |
| [manifest.json](manifest.json) | 이 묶음의 파일별 SHA-256·크기와 버전 구분 |
| [LICENSE](LICENSE) | MIT 라이선스 |

v1.1의 SHA-256은 `2E9666668BD1264EDFF9AC85C255B4C83FFB99718DF6A405AA8BED5D53C8C602`, 이전 원본 v1은 `E0D257FA8370B85029903A614B9D2ACD87A2480F4F65CB99F3B85247177666C5`다. 스킬과 예시, 공개한 입력·응답은 로컬 보존본의 바이트를 그대로 복사했다. 안내와 검증 요약은 공개용으로 새로 작성했으며 개인 경로·계정·세션 정보를 포함하지 않는다.

## 지원 범위

글·Markdown 표·텍스트 도식이 기본이며, 지원 환경에서는 Mermaid 같은 작은 정적 도식을 고려한다. 인터랙티브 HTML과 영상 제작은 이 스킬의 범위 밖이다. 사용자가 해당 매체를 요청하면 환경의 일반 기능으로 이어갈 수 있도록 한다.

스킬 파일 자체에는 외부 API·API 키·추가 패키지 의존성이 없다. 실행 제품의 설치·인증·이용 조건은 별도다. STE의 명료성 원칙을 참고했지만 공식 ASD-STE100 준수나 준수율을 주장하지 않는다.

아래 설치·호출 안내는 프로젝트의 기존 확인 기록을 옮긴 것이다. 이번 파일 분리 작업에서 다른 OS나 제품 UI를 새로 검증하지 않았다.

## 설치

Codex 또는 Claude Code가 설치된 환경에서 `skills/explain` 폴더를 아래 위치 중 하나로 복사한다. 이 프로젝트의 `skills/`는 배포 원본 경로이므로, 여기에 파일을 두는 것만으로 두 제품에서 자동 발견된다고 가정하지 않는다. 제품 설정을 바꾸거나 전역 설치하는 작업은 이번 초안 제작에 포함하지 않는다.

| 제품 | 현재 프로젝트에서 사용 | 모든 로컬 프로젝트에서 사용 |
| --- | --- | --- |
| Codex | `<프로젝트>/.agents/skills/explain/SKILL.md` | `~/.agents/skills/explain/SKILL.md` |
| Claude Code | `<프로젝트>/.claude/skills/explain/SKILL.md` | `~/.claude/skills/explain/SKILL.md` |

`~`는 실행 환경의 사용자 홈이다. Windows 기본 위치는 `%USERPROFILE%` 아래이며, WSL에서는 WSL의 홈과 프로젝트 경로를 사용한다. Codex의 경로·탐색은 [공식 스킬 문서](https://learn.chatgpt.com/docs/build-skills), Claude Code의 경로는 [공식 스킬 문서](https://code.claude.com/docs/en/skills)를 기준으로 했다(2026-10-09 확인).

이 프로젝트 루트에서 PowerShell로 프로젝트용 설치를 하려면 다음을 실행한다. Claude Code용은 첫 줄의 `.agents`를 `.claude`로 바꾼다.

```powershell
$skillInstallRoot = '.agents/skills'
$skillInstallTarget = Join-Path $skillInstallRoot 'explain'
if (Test-Path -LiteralPath $skillInstallTarget) {
    throw '이미 explain 폴더가 있습니다. 기존 내용을 확인한 뒤 설치하세요.'
}
New-Item -ItemType Directory -Path $skillInstallRoot -Force | Out-Null
Copy-Item -LiteralPath 'skills/explain' -Destination $skillInstallTarget -Recurse
```

macOS·Linux의 셸에서는 아래처럼 복사할 수 있다. Claude Code용은 첫 줄을 `skill_install_root=.claude/skills`로 바꾼다. 이미 있는 폴더는 덮어쓰지 않는다.

```sh
skill_install_root=.agents/skills
if [ -e "$skill_install_root/explain" ]; then
  echo 'explain already exists; inspect it before installing.' >&2
else
  mkdir -p "$skill_install_root" && cp -R skills/explain "$skill_install_root/explain"
fi
```

다른 프로젝트에 설치하려면 목적지 루트를 해당 프로젝트의 경로로 바꾸고, `skills/explain`은 이 프로젝트의 원본 폴더를 가리키게 한다. 두 제품에서 쓸 때는 각 위치에 복사하고, 수정 후에는 두 사본도 갱신한다.

## 호출

설치한 프로젝트를 열어 다음 내용을 **제품의 대화 입력창**에 입력한다. 셸에 입력하는 명령이 아니다.

Codex:

```text
$explain 캐시가 무엇인지 비전공자에게 설명해줘.
```

Codex CLI·IDE에서는 `/skills` 또는 `$`로 스킬을 선택할 수도 있다. 변경이 보이지 않으면 새 세션을 시작하거나 Codex를 재시작한다. [공식 호출 안내](https://learn.chatgpt.com/docs/build-skills)

Claude Code:

```text
/explain 캐시가 무엇인지 비전공자에게 설명해줘.
```

두 제품 모두 설명 요청과 `description`이 맞으면 자동 선택할 수 있다. 자동 선택의 결과는 모델과 세션에 따라 달라지므로, 확인할 때는 명시 호출과 자동 호출을 나누어 시험한다. Claude Code의 슬래시 호출은 [공식 안내](https://code.claude.com/docs/en/skills)를 따른다.

## 검증 결과를 읽는 방법

정상 실행과 스킬 본문 로딩 성공은 응답의 의미 보존 통과와 별개다. T02에서는 불명확한 용량 기준이나 명시된 기능 제한이, T03에서는 필요조건과 충분조건의 구분이 문제였다. 자세한 판정과 전체 응답은 [검증 요약](validation/README.md)에 있다.

원본 v1·후보 c01·c02와 서로 다른 제품의 결과를 섞어 하나의 통과율로 합치지 않는다. 사례마다 3~5회 반복했지만 표본이 작아 반복 안정성을 비율로 일반화하거나 통계적 유의성을 주장하지 않는다. 다른 입력으로의 일반화·사용자 이해도 향상은 확인되지 않았다.

원본 v1의 추가 검증에서는 T04 도식·T05 그림 없는 설명·T07 상관과 인과·T09-A HTML이 통과했다. T03의 필요조건 → 충분조건 실패는 스킬을 쓰지 않은 답변에서도 나타났다. 따라서 이 실패는 스킬이 만든 것이 아니라 **스킬이 막지 못한 것**으로 봤다.

v1.1(c02)은 이 부분을 직접 다룬다. 바뀐 내용은 다음과 같다.

- “~일 때만”은 그 조건이 아니면 일어나지 않는다는 뜻이지, 조건을 만족하면 반드시 일어난다는 뜻이 아니라는 규칙을 일반 예시와 함께 넣었다.
- “A만 된다/막힌다”는 그 밖의 것에 대한 정보도 담는다는 규칙을 넣었다.
- 조건을 표로 옮길 때 칸 값이 원문보다 강해지지 않게 하는 규칙을 넣었다.

c02 반복 시험에서 필요조건 → 충분조건 실패는 한 번도 나오지 않았다. 다만 제한 약화 2회, 기간 이후 허용 단정 1회, T01 표 추가 1회가 남았다.

## 공개 상태와 라이선스

이 묶음은 [MIT 라이선스](LICENSE)로 배포한다. 라이선스는 이 묶음에서 작성한 스킬·문서·예시와 공개한 검증 기록에만 적용된다. Andrej Karpathy의 원문 포스트나 ASD-STE100 규격 문서처럼 다른 사람의 저작물은 포함하지 않았고, 이 라이선스로 다시 허락하지 않는다. 아이디어 출처의 원문 대화 기록은 공개 묶음에 넣지 않았다.

전체 실행 로그·환경 수집·로컬 transcript·개인 경로가 들어간 내부 보고서와 작업 폴더는 이 묶음에 포함하지 않았다. 공개 입력·응답만으로 로딩·환경·기능 검사 이력 전체를 독립 검증할 수는 없으며, 그 한계도 검증 요약에 적었다.
