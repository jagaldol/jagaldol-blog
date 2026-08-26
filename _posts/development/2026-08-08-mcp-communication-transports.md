---
layout: single
title: "MCP 통신 구조 이해하기 - host, client, server와 두 가지 transport"
date: 2026-08-08 20:00:00 +0900
categories: development
header:
  teaser: /assets/images/2026/08/08/mcp-transports-hero.png
---

MCP(Model Context Protocol)는 AI 애플리케이션과 외부 기능이 도구·리소스·프롬프트를 주고받는 규약이다. 메시지는 JSON-RPC 2.0으로 표현하고, 로컬에서는 `stdio`, 원격에서는 Streamable HTTP로 옮긴다.

![MCP host와 server를 로컬 파이프와 원격 스트림으로 연결한 두 전송 방식](/assets/images/2026/08/08/mcp-transports-hero.png){: .align-center }

이 글은 `2026-07-28` 규격을 기준으로 한다. 현행 MCP는 요청마다 버전과 capability를 넣는 stateless protocol이다. `initialize`와 `Mcp-Session-Id`를 쓰는 설명은 2025 계열 규격에 해당한다.

## 모델이 MCP 메시지를 직접 보내지는 않는다

MCP를 구성하는 네 역할부터 구분해야 한다.

| 층 | 하는 일 |
| --- | --- |
| 모델 | 도구 사용 여부와 인자를 판단 |
| Host | UI, 모델, 권한, context를 조정 |
| MCP client | Host 안에서 서버와 protocol 통신 |
| MCP server | Tools, Resources, Prompts 제공 |

예를 들어 사용자가 날씨를 물었을 때 모델이 `tools/call` JSON을 직접 만드는 것은 아니다.

```text
사용자 요청
→ Host가 모델에 도구 정의 제공
→ 모델이 도구 이름과 인자 선택
→ MCP client가 tools/call 요청 생성
→ MCP server가 실행 결과 반환
→ Host가 결과를 모델 context에 추가
→ 모델이 최종 답변 생성
```

사용자 메시지와 최종 자연어 답변의 형식은 모델 provider와 host의 계약이다. MCP가 정하는 부분은 client와 server 사이의 메시지다.

## 메시지 층과 전송 층

MCP 통신은 두 층으로 나눠 읽으면 덜 헷갈린다.

- 메시지 층: `tools/list`, `tools/call` 같은 JSON-RPC method와 결과의 의미
- 전송 층: JSON-RPC 객체를 표준입출력이나 HTTP에 싣는 방법

Transport가 바뀌어도 `tools/call`의 의미는 같다. 반대로 JSON-RPC 2.0이 허용하는 모든 기능을 MCP가 지원하는 것은 아니다.

| 항목 | JSON-RPC 2.0 | MCP 2026-07-28 |
| --- | --- | --- |
| `params` | 배열 또는 객체 | 객체만 |
| Request `id` | 문자열, 숫자, `null` | 문자열 또는 숫자 |
| Batch | 허용 | 지원하지 않음 |
| 메타데이터 | 별도 규칙 없음 | `params._meta` |

JSON-RPC의 기본 메시지 구조가 궁금하다면 [JSON-RPC 2.0 글](/development/json-rpc-2-0/)에서 먼저 볼 수 있다.

## 현행 규격은 요청 하나가 완결돼 있다

2026 규격에서는 각 요청의 `params._meta`에 protocol version과 client capability를 넣는다.

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/list",
  "params": {
    "_meta": {
      "io.modelcontextprotocol/protocolVersion": "2026-07-28",
      "io.modelcontextprotocol/clientInfo": {
        "name": "ExampleClient",
        "version": "1.0.0"
      },
      "io.modelcontextprotocol/clientCapabilities": {}
    }
  }
}
```

서버는 요청 하나만 보고 적용할 version과 사용할 수 있는 client feature를 판단한다. 기능을 먼저 확인하고 싶다면 선택적인 `server/discover`를 쓸 수 있지만 필수 handshake는 아니다.

Stateless는 서버 애플리케이션이 상태를 저장할 수 없다는 뜻이 아니다. Protocol이 숨은 session에 의존하지 않는다는 뜻이다. 지속 상태가 필요하면 server가 handle을 발급하고 이후 tool argument로 되돌려 받는 식으로 명시한다.

## 도구 발견과 호출

Host는 필요한 시점에 `tools/list`로 도구 정의를 가져온다.

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/list",
  "params": {
    "_meta": {
      "io.modelcontextprotocol/protocolVersion": "2026-07-28",
      "io.modelcontextprotocol/clientCapabilities": {}
    }
  }
}
```

서버 응답에는 이름, 설명, input schema가 포함된다. Host는 이를 모델 API의 tool definition 형식으로 바꿔 전달한다. 목록은 매 사용자 메시지마다 다시 받을 필요가 없다. `ttlMs`가 남아 있고 변경 알림이 없다면 cache를 재사용할 수 있다.

모델이 `get_weather`와 `{"location":"서울"}`을 선택하면 client가 MCP 요청으로 바꾼다.

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "tools/call",
  "params": {
    "name": "get_weather",
    "arguments": {"location": "서울"},
    "_meta": {
      "io.modelcontextprotocol/protocolVersion": "2026-07-28",
      "io.modelcontextprotocol/clientCapabilities": {}
    }
  }
}
```

정상 완료와 tool 실행 실패는 모두 `resultType: "complete"`다. 업무 실패는 `isError: true`로 표현하고, 존재하지 않는 method나 잘못된 request 구조 같은 protocol failure는 최상위 JSON-RPC `error`를 쓴다.

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "result": {
    "resultType": "complete",
    "content": [{"type": "text", "text": "현재 27°C, 맑음"}],
    "structuredContent": {"temperature": 27, "conditions": "맑음"},
    "isError": false
  }
}
```

Server result가 곧 사용자의 최종 답변은 아니다. Host가 이 결과를 model용 tool result로 변환해 다시 모델을 호출한다. 모델은 답변하거나, 인자를 고쳐 재시도하거나, 다른 tool을 선택할 수 있다.

## `stdio`: 같은 기기의 로컬 서버

`stdio`에서는 host가 server를 child process로 실행한다.

- stdin: Client가 request와 notification을 한 줄씩 전송
- stdout: Server가 response와 notification을 한 줄씩 전송
- stderr: 진단 log

각 JSON-RPC message는 newline으로 구분한다. stdout에 일반 log를 출력하면 protocol stream이 오염되므로 log는 stderr로 보내야 한다.

```text
Host process
  ├─ child stdin  → JSON-RPC request
  ├─ child stdout ← JSON-RPC response
  └─ child stderr ← diagnostic log
```

개인 기기 안의 filesystem, database, developer tool을 연결할 때 구조가 단순하다. 인증 경계는 별도의 HTTP token보다 process 권한과 host 설정에 가깝다.

## Streamable HTTP: 원격 MCP 서버

현행 Streamable HTTP는 JSON-RPC request 하나마다 MCP endpoint에 새 POST를 보낸다.

```http
POST /mcp HTTP/1.1
Content-Type: application/json
Accept: application/json, text/event-stream
MCP-Protocol-Version: 2026-07-28
Mcp-Method: tools/call
Mcp-Name: get_weather
```

Server는 일반 JSON으로 한 번에 응답하거나, 같은 POST response를 SSE로 열어 progress notification 뒤에 최종 response를 보낼 수 있다.

```text
POST /mcp
  ← notifications/progress
  ← notifications/progress
  ← 최종 JSON-RPC response
  ← stream close
```

이 SSE는 기본적으로 request 하나에 붙은 response stream이다. 장기적인 목록 변경 알림은 별도의 `subscriptions/listen` 요청으로 구독한다.

2026 Streamable HTTP에는 2025 방식의 `GET /mcp`, `Mcp-Session-Id`, `DELETE /mcp`, `Last-Event-ID` 재개가 없다. 서로 다른 spec 세대의 header와 lifecycle을 한 구현 안에서 섞으면 상호운용성이 깨진다.

## 추가 입력이 필요할 때

Server가 사용자 확인이나 정보 없이는 작업을 끝낼 수 없다면 JSON-RPC error 대신 `input_required` 결과를 돌려준다.

```text
tools/call id=7
← resultType: input_required
   inputRequests + requestState

Host가 사용자 입력 수집

tools/call id=8
→ inputResponses + 같은 requestState
← resultType: complete
```

두 `tools/call`은 독립된 request라 새 `id`를 사용한다. `requestState`는 server가 만든 opaque 값이며 client는 해석하거나 수정하지 않고 되돌려준다.

## 구현할 때 지킬 경계

- 로컬 전용 HTTP server는 `0.0.0.0` 대신 `127.0.0.1`에 bind한다.
- HTTP request의 `Origin`을 검증하고 인증·인가를 별도로 둔다.
- Tool description과 annotation은 신뢰되지 않은 입력으로 취급한다.
- JSON-RPC `error`와 tool result의 `isError`를 구분한다.
- 문서와 SDK가 어느 MCP spec version을 구현하는지 먼저 확인한다.
- Server가 conversation state를 알아야 한다면 암묵적 session 대신 argument나 handle로 전달한다.

MCP를 이해하는 가장 빠른 방법은 AI 모델의 tool use, JSON-RPC message, transport를 한 덩어리로 보지 않는 것이다. 모델은 도구를 선택하고, host가 이를 MCP request로 변환하며, transport는 그 request를 옮길 뿐이다.

## 참고 자료

- [MCP 2026-07-28 specification](https://modelcontextprotocol.io/specification/2026-07-28)
- [MCP architecture](https://modelcontextprotocol.io/specification/2026-07-28/architecture)
- [MCP tools](https://modelcontextprotocol.io/specification/2026-07-28/server/tools)
- [MCP transports](https://modelcontextprotocol.io/specification/2026-07-28/basic/transports)
- [Streamable HTTP](https://modelcontextprotocol.io/specification/2026-07-28/basic/transports/streamable-http)
