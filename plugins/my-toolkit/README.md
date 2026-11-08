# my-toolkit

namyoungjun96이 직접 만든 Claude Code 스킬과 슬래시 커맨드를 모아 둔 플러그인입니다.
평소 개발 작업에서 반복하는 일을 스킬로 만들어 여기에 추가하고, git으로 변경 이력을 관리합니다.

## 포함된 것

- `skills/hello` - 설치 확인용 예시 스킬 ("my-toolkit 확인"이라고 요청하면 동작)
- `commands/hello.md` - 설치 확인용 예시 커맨드 `/my-toolkit:hello [이름]`

## 새 스킬 추가하기

`skills/<스킬이름>/SKILL.md` 파일을 만들고, frontmatter에 `name`과 `description`을 적습니다.
Claude는 `description`을 보고 언제 이 스킬을 쓸지 판단하므로 사용 시점을 구체적으로 적어 주세요.
