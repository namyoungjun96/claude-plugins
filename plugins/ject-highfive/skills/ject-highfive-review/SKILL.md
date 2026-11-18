---
name: ject-highfive-review
description: highfive 백엔드 레포(com.ject.highfive)에서 사용자가 PR을 올리기 전에 자기 변경 사항을 리뷰해 달라고 하거나, 팀원 PR을 리뷰해 달라고 할 때 사용. "리뷰해줘", "PR 전에 봐줘", "3번 PR 리뷰해줘", "/ject-highfive:ject-highfive-review" 등.
---

# highfive 개인 리뷰

`/code-review`가 버그를 찾고, 이 스킬은 그 위에 기획서 대조와 팀 규칙 확인을 더해 하나의 목록으로 정리한다.

리뷰만 한다. 코드는 사용자가 고치라고 할 때까지 수정하지 않는다.

## 대상

- 인자가 없으면: 현재 브랜치와 `develop`의 merge-base부터 지금까지 (커밋 안 된 변경 포함)
- 인자로 PR 번호나 브랜치가 오면 그것

PR 번호나 다른 브랜치가 대상이면 기계 검사가 그 코드에서 돌도록 먼저 체크아웃한다.

1. `git status`로 작업 트리가 깨끗한지 확인한다. 커밋 안 된 변경이 있으면 멈추고 사용자에게 묻는다.
2. 지금 브랜치를 기억해 두고 `gh pr checkout {번호}` (브랜치면 `git checkout {브랜치}`)
3. 리뷰가 끝나면 기억해 둔 브랜치로 돌아간다

PR 작성자(`gh pr view {번호} --json author`)가 사용자(`git config user.name`)가 아니면 **팀원 PR**이다.
팀원 PR은 출력 마지막의 질문만 다르다.

## 절차

1. **기계 검사.** `./gradlew spotlessCheck checkstyleMain checkstyleTest test` (Windows PowerShell은
   `.\gradlew.bat`). 실패한 항목은 `[필수]`로 올린다. 이 명령이 잡는 포맷·네이밍·import 순서는 5단계에서 다시 보지 않는다.
2. **버그 리뷰.** Skill 도구로 `code-review`를 같은 대상으로 호출한다. 사용자가 수준(low~xhigh)을 말했으면
   그대로, 아니면 `medium`. 결과는 따로 보고하지 않고 마지막 목록에 합친다. `code-review`가 주는 줄 번호는
   diff 출력 기준일 수 있으므로 목록에 넣기 전에 `grep -n` 등으로 소스 파일의 줄 번호를 확인해 맞춘다.
3. **기능 특정.** 브랜치 이름(`feature/JECT-번호-기능`)·커밋 메시지·바뀐 코드로 `docs/기능명세서-v4.md`의
   `### #번호` 섹션을 찾는다. 특정할 수 없으면 사용자에게 묻는다.
4. **기획서 대조.** 아래 문서를 필요한 만큼 읽고 확인한다.
   - 명세에 있는데 구현되지 않은 것 / 명세와 다르게 구현된 것 / 명세에 없는 동작이 들어간 것

   | 문서 | 언제 |
   |---|---|
   | `docs/기능명세서-v4.md` 해당 섹션 | 항상 |
   | `docs/api/API명세서-v0.1.md` | 컨트롤러나 Request/Response DTO가 바뀌었을 때 |
   | `docs/fisrt_highfive_ERD.sql` (최신본), `docs/ERD/erd-review_v4.md` 5장 | 엔티티가 바뀌었을 때. 컬럼·테이블 이름과 5장의 UNIQUE/FK/AUTO_INCREMENT 제약 반영 여부. `docs/ERD/*_v{번호}.sql`은 지난 버전 기록이라 기준으로 쓰지 않는다 |
   | `docs/의사결정.md` | 명세와 코드가 다를 때. 결정(`D번호`)으로 바뀐 것이면 지적하지 않는다 |

   `docs/협의할-점.md`와 컨벤션 리뷰 문서의 "미정" 항목은 기준으로 쓰지 않는다. 미정인 것을 코드가 임의로
   정했다면 `[참고]`로 알린다.
5. **팀 규칙.** 아래 규칙을 바뀐 코드에만 대조한다. 원본은 `docs/develop_convention.md`이고, 둘이 다르면 원본을 따른다.

## 규칙

**JPA 엔티티**
- `@AllArgsConstructor`, `@Setter`, `@Data` 없음. `@NoArgsConstructor(access = AccessLevel.PROTECTED)`
- `@Builder`는 클래스가 아니라 생성자에
- 상태 변경은 setter 대신 의미 있는 메서드 (`recruitPost.close()`)
- `@ManyToOne`, `@OneToOne`에 `fetch = FetchType.LAZY` 명시. 테이블·컬럼 snake_case

**DTO**
- `record`, 이름 `XxxRequest` / `XxxResponse`
- 엔티티 → DTO는 DTO의 정적 팩토리 `from(entity)`. 엔티티를 응답으로 직접 내보내지 않는다
- 검증은 record 컴포넌트에 Bean Validation, 컨트롤러에서 `@Valid`

**패키지**
- 도메인형 `{domain}/controller`, `{domain}/service` …, 공통은 `global/{config, exception, security, common, util}`
- 새 `util` 클래스는 꼭 필요한지 확인

**API**
- URL `/api/v1/...`, 복수형, kebab-case. 로그인 없이 여는 API(관객 신청·조회, 공유 링크)는 `/api/v1/public/**`
- 성공 `{ status, code, message, data }`, 코드 `COMMON_000`, 빈 결과는 `data: {}` (목록이면 빈 배열)
- 실패 `{ status, code, message }`, 검증 실패는 `errors: [{ field, reason }]`에 필드별 오류 전부
- 에러 코드 `{DOMAIN}_{세 자리}`, 접두어 `COMMON, AUTH, USER, BAND, CONCERT, RECRUIT, CHAT, TICKET, BOOKING, ENTRY, NOTIFICATION`
- 예외는 `@RestControllerAdvice`에서 처리. 컨트롤러에서 try-catch로 응답을 만들지 않는다
- 페이징: offset, page는 1부터(0이 와도 1), size 기본 10·최대 100. `data`에 `content, page, size, totalElements, totalPages, hasNext`
- 날짜만 `LocalDate`, 일시 `LocalDateTime`, 따로 "미정"일 수 있는 공연 날짜·시간은 `LocalDate` + `LocalTime`. 시간대 `Asia/Seoul`
- 새 API에 springdoc 문서 어노테이션

**인증·권한**
- Security는 로그인 여부(401)만. 밴드·공연별 역할(대표자, 주최 대표자, 계좌 주인, 참여 팀 멤버)은 서비스에서 DB로 확인하고 403 (`AUTH_005`)
- 권한이 필요한 서비스 메서드에 역할 확인이 빠지지 않았는지 (남의 밴드·공연을 수정·삭제할 수 있는지)
- 컨트롤러는 `@AuthenticationPrincipal`(또는 `@LoginUser`)로 userId를 받는다. 요청 바디·파라미터의 userId를 믿지 않는다
- JWT 클레임은 `sub`, `type`만. 비밀 키는 환경변수, 레포에 커밋하지 않는다

**테스트**
- 새 서비스 로직, 특히 권한 확인과 상태 변경에 테스트가 있는지

## 지적 기준

목록에는 지금 코드에서 실제로 틀리거나 명세와 다른 것만 올린다. `code-review` 결과도 이 기준으로 거른다.

**지적하지 않는 것**
- 지금은 일어나지 않고, 다른 티켓이나 아직 없는 기능이 생겨야 문제가 되는 것
- 테스트 편의를 위한 개선, 취향 차이의 리팩터링, 추상화 제안 (helper·공통 클래스는 같은 코드가 세 번째 나올 때)
- 이미 TODO나 티켓으로 미룬 항목. 목록에서 빼고 맨 아래에 `미룬 항목: JECT-248, JECT-249`처럼 한 줄로만 적는다
- 가상의 시나리오(악의적 입력, 동시 요청 등)는 지금 데이터가 깨지거나 잘못된 응답이 나갈 때만 남긴다

**대신 지적하는 것**
- 지우거나 더 단순하게 만들 수 있는 코드: DB 제약·Bean Validation·전역 예외 처리기가 이미 막는 것을 다시 막는
  방어 코드, 쓰이지 않는 검증, 다음 티켓을 대비해 미리 만든 코드

## 출력

맨 위에 한 줄 결론: `병합 가능` / `수정 후 병합` / `다시 작업 필요`와 심각도별 건수.

그 아래 심각도 순으로, 한 항목에 한 줄:

```
[심각도] 파일:줄 - 문제 - 제안 (근거)
```

| 심각도 | 기준 |
|---|---|
| `[필수]` | 병합 전에 고쳐야 함. 버그, 명세 불일치, 권한 확인 누락, 기계 검사 실패, 규칙 위반 |
| `[권장]` | 고치는 게 낫지만 병합을 막지 않음 |
| `[참고]` | 판단은 사용자에게. 미정 항목을 코드가 임의로 정한 경우 등 |

근거는 `버그`, `기능명세서 #12`, `규칙: DTO`처럼 어디서 나온 항목인지 적는다. 같은 문제가 여러 단계에서
나오면 한 줄로 합치고 근거를 함께 적는다. 항목이 없으면 `없음`.

마지막 질문은 대상에 따라 다르다.

- 내 변경: "어느 항목을 고칠까요?"라고 묻고 멈춘다.
- 팀원 PR: 팀원 코드는 고치지 않는다. "PR 코멘트로 남길 항목이 있을까요?"라고 묻고 멈춘다. 코멘트는
  사용자의 GitHub 계정으로 게시되므로, 남기기로 하면 문구를 먼저 보여 주고 확인을 받은 뒤에 올린다.
