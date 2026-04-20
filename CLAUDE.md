# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

비개발자 대상 AI 코딩 에이전트(Claude Code 등) 실습 교육용 MkDocs Material 위키. 콘텐츠는 한국어 경어체로 작성되며, 코드는 거의 없고 **마크다운 문서가 사실상의 산출물**이다.

## Build & Local Preview

```bash
pip install mkdocs-material        # 의존성 설치 (mkdocs-material 한 패키지가 mkdocs까지 끌어옴)
mkdocs serve                       # http://127.0.0.1:8000 로컬 미리보기 (저장 시 핫리로드)
mkdocs build                       # ./site 로 정적 빌드 (gitignored)
mkdocs gh-deploy --force           # 수동 배포 (보통은 GitHub Actions가 자동 처리)
```

CI는 [.github/workflows/deploy.yml](.github/workflows/deploy.yml). **`main` 브랜치에 push되고 `docs/**` 또는 `mkdocs.yml`이 변경된 경우에만** 트리거되어 `mkdocs gh-deploy --force`로 GitHub Pages에 배포한다. 다른 파일(README, CLAUDE.md, .github 자체)을 고쳐도 배포가 일어나지 않으므로, 콘텐츠 변경 없이 배포를 강제하고 싶다면 `mkdocs.yml`을 함께 건드리거나 워크플로를 수동 실행해야 한다.

## Content Architecture

전체 사이트는 **2개 실습 트랙 × Stage 0~4 프레임워크**로 구성된다. 두 트랙이 의도적으로 같은 골격을 공유하므로, 한쪽을 수정하면 다른 쪽의 대응 페이지도 일치 여부를 확인해야 한다(예: `practice1/autoapprove.md`와 `practice2/autoapprove.md`는 거울 구조).

- **practice1** — 모집요강 PDF에서 Excel로 텍스트 추출 (텍스트 자동화)
- **practice2** — 수학 교재 PDF 디지털화 (이미지 자동화, Stage 2b·3 추가)

공통 Stage 의미:

| Stage | 핵심 질문 |
|------|---------|
| 0 | 일단 시켜본다 (어디서 헷갈리는지 관찰) |
| 1 | 디테일/도메인 지식을 채워준다 |
| 2 | 자동화 설계 + 검증 |
| 3 | 사람 확인 (실습 1은 Stage 2에 흡수됨) |
| 4 | Skill로 포장해 재사용 |

위키의 핵심 메시지는 **"관찰담"**(비개발자가 자료에서 발견한 패턴을 자연어로 AI에 전달) 개념이다. 새 문서를 쓸 때 이 톤을 유지해야 한다.

## Navigation Is Manual

`mkdocs.yml`의 `nav:` 섹션이 사이트 좌측 사이드바와 페이지 순서를 **완전히 수동으로** 통제한다. 새 `.md`를 만들면 자동으로 노출되지 않는다 — 반드시 `nav:`에 한국어 라벨과 함께 등록해야 한다. 라벨에 콜론(`:`)이 있으면 따옴표로 감싸는 기존 패턴을 따른다.

## Authoring Conventions

- **언어**: 한국어 경어체 (반말 → 경어 통일이 이미 진행됨, 커밋 `64a786a` 참고). 새 글도 경어체로 작성.
- **Admonition 사용**: `!!! info`, `!!! tip`, `!!! warning`, `!!! abstract`, `!!! success`, `!!! note` 적극 활용. 빈 제목은 `!!! tip ""` 형태.
- **Stage 간 이동**: 페이지 상단/하단에 `<div class="stage-nav" markdown>` 블록으로 다음/이전 링크 명시.
- **Mermaid 다이어그램**: ` ```mermaid ` 코드 블록 사용 (mkdocs.yml의 `superfences custom_fences`로 등록됨).
- **이미지 경로**: `../images/practice1/...` 또는 `../images/practice2/...` 상대 경로. 트랙별로 폴더가 분리되어 있다.
- **프롬프트 수 제한**: Stage 페이지당 프롬프트는 **최대 2개**로 압축한다는 편집 방침이 있다(커밋 `f244415`). 새 Stage 페이지를 추가하거나 늘릴 때 이 원칙을 따른다.
- **표/admonition 폰트는 본문보다 작게** 잡혀 있다(`docs/stylesheets/extra.css`) — 표가 너무 길어지면 가독성을 직접 확인할 것.

## Styling

`docs/stylesheets/extra.css`가 Material 기본 테마를 두 군데에서 의도적으로 깬다:

1. **페이지 폭 제한 해제**: `.md-grid`, `.md-content` 등의 `max-width`를 모두 `none`으로 — 화면 가득 사용 (커밋 `0aace49`). 새 컴포넌트가 폭에 의존하면 안 된다.
2. **사이드바 축소**: 본문에 더 많은 공간을 할애.

스타일 변경 시 두 트랙 모두에서 시각적으로 확인한다.

## Korean Filename / Label Caveats

`mkdocs.yml`에 한국어 라벨을 쓰지만 **파일명은 ASCII**(`stage0.md`, `autoapprove.md`)로 유지한다. URL 파편화·인코딩 이슈 방지 목적.

## What Lives Outside `docs/`

- `site/` — `mkdocs build` 산출물, gitignored
- `.omc/` — 로컬 OMC 상태 디렉터리, 콘텐츠와 무관
- `.agent/`, `.claude/` — 학습자가 다운받는 `practice_*.zip` 안에 들어있는 폴더 (위키 저장소엔 없음). 학습자 환경의 일부일 뿐, 이 저장소엔 절대 추가하지 말 것.
