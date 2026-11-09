# AI Creative Agent 연동 가이드

## 1. Copilot Studio 이벤트 규격
Copilot Studio 토픽에서 **이벤트 활동 보내기(Send event)** 노드로 전송합니다. `step`은 **0부터 시작**(0=요청 접수 … 4=최종 전달)합니다.

| name | value |
|---|---|
| `reportStarted` | 없음 |
| `reportProgress` | `{ "progress": 60, "step": 2, "message": "보고서 내용을 작성하고 있습니다.", "state": "running" }` |
| `reportCompleted` | 없음 |
| `reportFailed` | `{ "step": 2, "message": "실패 사유" }` |

- `progress`를 생략하면 퍼센트 대신 "작업 진행 중"으로 표시됩니다(가짜 진행률 없음).
- 이벤트가 전혀 오지 않으면 작업 현황은 "대기 중"으로 유지됩니다.

## 2. Azure Function 수정 (필수)
브라우저는 더 이상 `SEALED`를 보내지 않고 `POST {}`만 보냅니다.
1. `SEALED` 값을 Function App 환경 변수(`SEALED`)로 이동하고 서버에서 사용합니다.
2. 응답은 `{ "token": "<Direct Line token>" }` 형식이어야 합니다.
3. CORS 허용 출처에 `https://kimanastasiya.github.io`를 추가합니다(로컬 테스트 시 `http://127.0.0.1:8080`도).
4. 이전 커밋에 노출된 SEALED 값은 **재발급(폐기)** 하세요.

위 수정 전에는 토큰 발급이 실패하여 "연결 오류"가 표시됩니다.

## 3. 진행 상황 테스트
채팅이 연결된 상태에서 브라우저 콘솔에서 이벤트를 모의 수신할 수 없으므로, Copilot Studio 테스트 토픽에서 위 이벤트를 순서대로 보내 확인합니다: Started → Progress(step 1, 30) → Progress(step 2, 60) → Completed. 실패 확인은 `reportFailed`를 보냅니다.

## 4. 미니게임 테스트
오른쪽 패널에서 보기 선택 → 정답/오답과 해설 표시 → 재클릭 불가 → "다음 문제" → 5문제 후 처음으로 순환. 채팅과 독립적으로 동작합니다.

## 5. 배포·실행
정적 파일이므로 `main`에 푸시하면 GitHub Pages(`https://kimanastasiya.github.io/newAIDE/`)에 반영됩니다. 로컬은 `python -m http.server 8080` 후 `http://127.0.0.1:8080/`.
