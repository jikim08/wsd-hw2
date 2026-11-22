# 행사 신청 승인 서비스

학생의 행사 신청과 담당 운영자의 승인 업무를 관리하는 웹 서비스이다. 제공된 시작 코드를 기반으로 신청, 대기 취소, 직접 승인, 승인 취소와 대기자 자동 승급을 구현하였다.

- 학번 / 이름: 202219377 / 김지후
- 기술: Node.js 24.x, Express, SQLite, HTML/CSS/JavaScript

## 설치와 실행

Node.js 24.x와 npm이 필요하다. SQLite는 Node.js 내장 모듈을 사용하므로 별도 DB 서버는 필요하지 않다.

프로젝트 루트에서 다음 명령을 실행한다.

```bash
node --version
npm ci
npm start
```

- 접속 주소: http://127.0.0.1:3000
- 서버 종료: 실행 중인 터미널에서 `Ctrl+C`
- 첫 실행 시 DB와 JWT 서명 키가 자동 생성된다.
- 기본 실행에는 별도 환경변수 설정이 필요하지 않다.
- 브라우저 새로고침 후에는 다시 로그인해야 하며, DB 데이터는 유지된다.

## 테스트 계정

공통 비밀번호: `Classroom123!`

| 아이디 | 역할 |
|---|---|
| alice, bob, charlie | 학생 |
| manager1 | 행사 1 담당 운영자, 정원 2명 |
| manager2 | 행사 2 담당 운영자, 정원 1명 |

## DB 초기화

서버를 `Ctrl+C`로 종료한 뒤 다음 명령을 실행하고 `RESET`을 입력한다.

```bash
npm run db:reset
npm start
```

기존 신청과 변경 이력이 삭제되고 초기 계정과 행사가 생성된다.

## 변경 파일

| 파일 | 변경 내용 |
|---|---|
| `src/application-service.js` | `apply`, `cancel`, `approve`, `revoke`의 업무 규칙, 상태 변경, 이력 저장과 트랜잭션 구현 |
| `src/repository.js` | 같은 행사의 대기자를 신청 시각과 ID 오름차순으로 선택하는 `nextPending` 함수 추가 |

인증, API 입력 검사, DB 스키마, 기본 화면과 기존 저장 함수는 제공 코드를 활용하였다.

## 구현 완료 여부

| 기능 | 상태 |
|---|---|
| 학생 신청과 중복 신청 차단 | 완료 |
| 본인의 PENDING 신청 취소 | 완료 |
| 담당 운영자의 직접 승인과 정원 검사 | 완료 |
| 승인 취소와 대기자 최대 1명 자동 승급 | 완료 |
| 변경 이력 저장과 원인 이력 연결 | 완료 |
| 저장 실패 시 전체 롤백 | 완료 |
| 요청마다 현재 상태 재검사 | 완료 |

### 주요 업무 규칙

- 학생만 신청할 수 있으며, 본인의 PENDING 신청만 취소할 수 있다.
- 해당 행사 담당 운영자만 승인과 승인 취소를 수행할 수 있다.
- 동일 사용자와 행사별 신청은 한 건만 허용한다.
- CANCELLED와 REVOKED 상태에서도 재신청할 수 없다.
- APPROVED 상태의 신청만 정원에 포함한다.
- 직접 승인은 운영자가 선택하며, 정원이 찬 경우 거부한다.
- 승인 취소 시 같은 행사의 PENDING 신청을 `created_at ASC, id ASC`로 정렬하여 최대 한 명만 자동 승인한다.
- 자동 승급 이력의 `causeHistoryId`에 승인 취소 이력 ID를 저장한다.
- 상태 변경과 이력 저장은 하나의 트랜잭션으로 처리하며, 오류가 발생하면 전체를 롤백한다.
- 거부된 요청은 상태와 이력을 변경하지 않는다.

## 검증

```bash
# 제공 기반 기능 검증
npm test

# 필수 과제 테스트 T01~T32
npm run test:assignment
```

필수 테스트 T01~T32의 통과를 확인하였다. 테스트는 별도 임시 DB를 사용한다.

실패 주입 검사에서 출력되는 `TEST_INJECTED_FAILURE`는 롤백 검증을 위한 의도된 오류이다.

## 의존성과 제출 파일

`package.json`과 `package-lock.json`을 함께 포함하였다. `npm ci`로 잠금 파일에 명시된 의존성을 설치할 수 있다.

보고서 경로는 `docs/report.pdf`이다.

다음 파일은 제출에서 제외한다.

- `node_modules/`
- 생성된 DB 파일
- JWT 서명 키
- 실제 `.env` 파일과 JWT
- 개인정보

## AI 활용

ChatGPT를 활용하여 업무 규칙과 트랜잭션 처리 방법을 이해하고, 서비스 함수 구현 예시와 테스트 요청 코드를 참고하였다.

적용 위치는 `src/application-service.js`의 `apply`, `cancel`, `approve`, `revoke`와 `src/repository.js`의 `nextPending` 함수이다.

코드를 직접 작성하고 오류를 수정하였으며, 자동 테스트와 화면 검증으로 동작을 확인하였다.

## 참고 자료

- 제공된 과제 2 안내문과 시작 코드
- API 명세: `docs/API.md`
- Node.js SQLite: https://nodejs.org/docs/latest-v24.x/api/sqlite.html
- Express: https://expressjs.com/en/5x/api/