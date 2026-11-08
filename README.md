# claude-plugins

namyoungjun96의 개인 [Claude Code](https://claude.com/claude-code) 플러그인 마켓플레이스입니다.
직접 만든 스킬과 커맨드를 git으로 버전 관리하고, 어느 PC에서든 한 줄로 설치할 수 있습니다.

## 설치

```
/plugin marketplace add namyoungjun96/claude-plugins
/plugin install my-toolkit@namyoungjun-plugins
```

로컬에서 개발할 때는 GitHub 대신 로컬 폴더를 연결합니다.

```
/plugin marketplace add C:\portfolio\claude-plugins
```

## 구성

| 플러그인 | 종류 | 설명 |
|---|---|---|
| `my-toolkit` | 직접 제작 | 개인 스킬과 커맨드 (`plugins/my-toolkit`) |
| `superpowers` | 외부 링크 | [obra/superpowers](https://github.com/obra/superpowers). 파일을 복사하지 않고 원본 저장소를 가리킴 |

> 외부 링크 플러그인은 "내가 쓰는 플러그인 목록"을 관리하려고 넣은 것입니다. 이미 원래 마켓플레이스에서
> 설치했다면 중복이므로 여기서 다시 설치하지 않아도 됩니다.

## 구조

```
.claude-plugin/marketplace.json        마켓플레이스 정의 (플러그인 목록)
plugins/my-toolkit/
├── .claude-plugin/plugin.json         플러그인 메타데이터
├── skills/<이름>/SKILL.md             스킬: Claude가 상황에 맞으면 자동으로 사용
└── commands/<이름>.md                 슬래시 커맨드: /my-toolkit:<이름>
```

## 작업 흐름

1. `plugins/my-toolkit/skills/<새-스킬>/SKILL.md` 추가 또는 수정
2. `plugins/my-toolkit/.claude-plugin/plugin.json`의 `version` 올리기 (예: 0.1.0 → 0.1.1)
   - 설치된 플러그인은 버전별로 캐시되므로, 버전이 그대로면 수정 내용이 반영되지 않을 수 있음
   - 버전은 `plugin.json` 한 곳에서만 관리 (`marketplace.json`에는 적지 않음)
3. `claude plugin validate .` 로 검증
4. 커밋 후 push
5. 반영
   - 자동 업데이트 ON: Claude Code 재시작만 하면 됨 (`/plugin` → Marketplaces → namyoungjun-plugins → Enable auto-update)
   - 자동 업데이트 OFF: `/plugin marketplace update namyoungjun-plugins` 실행

## 라이선스

MIT (외부 링크 플러그인은 각 원본 저장소의 라이선스를 따릅니다)
