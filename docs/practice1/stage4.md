# Stage 4. Skill 맛보기

<div class="stage-nav" markdown>
**← 이전** [Stage 3. 사람 확인 및 정답 비교](stage3.md) · **다음 →** [부록. 에러 대처와 프롬프트 팁](appendix.md)
</div>

> 지금까지의 대화와 절차를 하나의 **재사용 가능한 작업 지침서**로 묶는 단계입니다.

!!! info ""
    **실습 2에서 본격적으로 Skill을 만들고 테스트할 예정**이므로, 여기서는 "아 이런 식이구나"를 느끼면 충분하다.

| :material-timer-outline: 예상 소요 | :material-package-variant: 핵심 산출물 | :material-star-outline: 난이도 |
|:--:|:--:|:--:|
| 15분 | Skill 초안 + 타 대학 테스트 결과 | 보통 |

### 스킬(Skill)이란?

AI에게 특정 태스크를 수행하는 방법을 가르치는 **구조화된 지식 문서**입니다. 사람으로 치면 "이 일은 이런 순서로 하고, 이런 점을 조심하고, 이런 도구를 써라"라고 적어둔 매뉴얼과 같습니다.

- 기본적으로 `.md` (마크다운) 문서 형태로 작성되며, 설명과 사용 조건, 작업 절차가 담겨 있습니다
- `name`, `description`을 바탕으로 AI가 어떤 스킬을 언제 사용할지 결정합니다
- **Skill 없이**: 매번 "PDF 열어줘, 표 찾아줘, 추출해줘, 검증해줘"를 처음부터 다시 설명
- **Skill 있으면**: "이 PDF를 모집요강 파싱 Skill로 처리해줘" 한 마디로 시작 가능

### Skill 저장 위치

| 에이전트 | 이 프로젝트에서만 | 전체 프로젝트에서 |
|---------|:-:|:-:|
| 클로드 코드 | `{폴더명}/.claude/skills/` | `~/.claude/skills/` |
| 코덱스 | `{폴더명}/.agents/skills/` | `~/.codex/` |
| 안티그래비티 | `{폴더명}/.agent/skills/` | `~/.gemini/antigravity/skills/` |

!!! warning ""
    "프로젝트"는 결국 작업 중인 폴더를 의미합니다. 다른 폴더에서도 동일한 스킬을 사용하고 싶다면 전체 프로젝트 위치에 넣어야 합니다.

---

## 4-1. Skill 만들기

!!! example "Skill 생성 프롬프트"

    === "Claude Code"
        ```text
        /skill-creator 지금까지 우리가 한 작업을 재사용 가능한 Skill로 만들어줘. 
        이름은 recruit-parser로 해줘.
        
        이 Skill의 목적:
        - 대학 모집요강 PDF를 넣으면 정해진 형식의 Excel이 나오는 것
        - 어떤 대학이든 동작해야 함
        
        Skill에 포함해야 할 것:
        1. PDF 전체를 훑어서 관련 페이지를 찾는 과정
        2. 표를 추출하고 셀 병합을 처리하는 과정
        3. 여러 페이지에 걸친 표를 합치는 과정
        4. Excel로 변환하는 과정
        5. 자동 검증 과정
        6. 텍스트 추출 실패 시 OCR, Vision API 같은 대체 경로로 분기하는 판단 기준
        ```

    === "Codex"
        ```text
        @Skill Creator 지금까지 우리가 한 작업을 재사용 가능한 Skill로 만들어줘. 
        이름은 recruit-parser로 해줘.
        (이하 동일)
        ```

    === "Antigravity"
        ```text
        @skill-creator 지금까지 우리가 한 작업을 재사용 가능한 Skill로 만들어줘. 
        이름은 recruit-parser로 해줘. 저장 경로는 루트 디렉토리의 .agent/skills
        (이하 동일)
        ```

---

## 4-2. 다시 써볼 수 있어야 진짜 완성

!!! tip "시간이 남으면"
    다른 대학 PDF에 Skill을 적용해본다. 시간이 없다면 실습 2에서 본격적으로 할 예정이니 넘어가도 된다.

!!! example "다른 PDF 테스트 프롬프트"
    ```text
    /recruit-parser 이 스킬 이용해서 연세대 모집요강도 처리해줘
    ```

??? note "스킬이 안떠요.."
    간혹 방금 만든 스킬이 인식이 안되는 경우가 있는데 IDE를 새로고침 해주세요. `Ctrl + Shift + P`를 누른 뒤 **Developer: Reload Window** 를 선택하면 됩니다.

!!! tip "판단 팁"
    - 지금 PDF 하나에서만 잘 되면 아직 Skill이 아닙니다
    - 다른 대학 PDF에서 어디가 깨지는지 봐야 진짜 범용성이 드러납니다
    - 실제 업무에서는 안 되는 부분이 나오면 Skill 수정 → 다시 테스트를 반복하세요

??? question "왜 Skill화가 중요한가요?"
    자동화는 한 번 잘 되는 것보다, 다음에도 더 빨리 다시 할 수 있게 만드는 것이 중요합니다. Skill은 사람의 경험과 AI 협업 방식을 함께 저장하는 가장 실용적인 포장 방식입니다.

---

## 체크포인트

- [ ] Skill을 생성했다
- [ ] 다른 대학 PDF에 적용해봤다
- [ ] 안 되는 부분을 수정해서 Skill을 개선했다

<div class="stage-nav" markdown>
**← 이전** [Stage 3. 사람 확인 및 정답 비교](stage3.md) · **다음 →** [부록. 에러 대처와 프롬프트 팁](appendix.md)
</div>
