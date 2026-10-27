# v1/v2 도구 호출과 컨텍스트 전달

역할·모델·추론 강도·업무 범위·재위임 정책은 v1/v2에 공통이야. 실제 호출 도구가 제공하는 인자를 맞춰 사용해.

템플릿은 버전을 강제하거나 전환하지 않아. 지원이 확인되지 않은 버전 전환·컨텍스트 제어·재위임 깊이 설정 키를 임의로 추가하지 마.

## Ultra의 실제 메인 추론 강도

- Codex는 선택된 `Ultra`를 모델의 `multi_agent_reasoning_effort`로 변환해. 유효한 모델 지정값이 없으면 지원되는 Max, 그 외 지원 목록의 마지막 non-Ultra 값, Medium 순으로 선택해. [정규화 구현](https://github.com/openai/codex/blob/6905ecede7816e91373e9eabda3f6b0368fc747b/codex-rs/protocol/src/openai_models/reasoning_effort.rs#L8-L35).
- 이 정규화는 일반 추론 요청의 `reasoning.effort`를 구성하는 경로에도 사용돼. 서브에이전트 기본값만 설명하는 필드로 해석하지 않아. [요청 구성 코드](https://github.com/openai/codex/blob/6905ecede7816e91373e9eabda3f6b0368fc747b/codex-rs/core/src/client.rs#L957-L976).
- 확인한 `0.164.0-alpha.11.1` 태그와 설치본의 기본 카탈로그에서는 `gpt-6-astra`, `gpt-6.1-sol` 모두 해당 값이 `xhigh`야. [Astra](https://github.com/openai/codex/blob/6905ecede7816e91373e9eabda3f6b0368fc747b/codex-rs/models-manager/models.json#L22), [Sol](https://github.com/openai/codex/blob/6905ecede7816e91373e9eabda3f6b0368fc747b/codex-rs/models-manager/models.json#L195).
- 실제 적용값은 해당 세션이 사용하는 모델 메타데이터와 엔진 버전에 따라 확인해. 모든 Codex 버전에서 메인이 high로 고정된다고 가정하지 마. 이 템플릿은 메인 모델·모드를 덮어쓰지 않고 역할별 서브 설정을 지정해.

## 컨텍스트 선택

| 의도 | v2 도구의 `fork_turns` | v1 도구의 `fork_context` |
|---|---|---|
| 새 서브에 필요한 정보만 전달 | `fork_turns="none"` | `fork_context=false` |
| 새 서브에 전체 부모 대화 이력 전달 | `fork_turns="all"` | `fork_context=true` |

- 해당 인자와 값을 실제 스키마가 지원할 때만 사용해. 기본은 선별 전달이야.
- 목표·필수 맥락·확정된 결정·고정 제약·근거 위치·담당 파일·완료 기준·재위임 허용 여부와 슬롯 수를 자체적으로 충분하게 전달해.
- 전체 이력 자체가 필요하거나 사용자가 요청하면 전체 전달을 명시하고 이유를 짧게 남겨.
- 두 인자를 함께 보내거나 한 인터페이스의 인자를 다른 도구에 보내지 마. 둘 다 없으면 문서화된 다른 전달 방식을 확인하고, 방법이 없으면 제한을 보고해.
- 전체 이력 상속과 모델/강도 지정의 충돌은 해당 도구에서 실제 확인된 경우에만 적용해. 충돌할 때 필요한 근거 원문을 선별 전달하고 선택한 모델을 몰래 바꾸지 마.
- none/false는 새 서브 생성 시 부모 이력 상속 선택이야. 기존 서브에 후속 메시지를 보내면 그 서브의 이력은 유지돼.

## 역할·모델 선택

1. 명명 역할 인자가 있고 실제 노출된 정의가 현재 파일과 일치하면 그 역할을 사용해.
2. 명명 역할이 없거나 오래된 정의라면 해당 TOML을 읽고 지원되는 명시 모델·강도 인자와 역할 지침을 전달해.
3. 필요한 인자를 지원하지 않으면 제한을 보고해. 프롬프트에 모델명만 썼다고 실제 모델이 바뀐 것으로 간주하지 마.
4. 역할 파일에 명시한 모델·강도와 실제 선택을 구분해 확인해. 역할 파일을 적용하는 경로에서는 파일의 모델·강도가 우선하므로 다른 강도로 실행됐다고 임의 주장하지 마.
5. 계정·클라이언트가 지원하지 않는 모델·강도를 다른 값으로 자동 대체하지 마.

## 새 프로젝트에서 확인할 사항

- 대상 프로젝트의 기존 설정·신뢰 상태·계정별 모델 지원을 확인해.
- 문법 검사·실제 설정 로드·현재 도구 정의·실제 작업 결과를 구분해 기록해. 일반 스레드 API나 프로토콜 생성물의 버전 폴더명으로 생성 도구의 인자를 추정하지 마.
- 첫 필요한 실제 위임에서 역할과 호출 인자가 의도대로 적용되는지 확인해. 의미 없는 검증용 에이전트나 모델 요청을 만들지 않아.
- Terra를 비롯한 역할을 특정 호출 버전 전용으로 분리하지 않아. 각 역할의 공통 지침을 유지하고 도구 호출부만 맞춰.

## 공식 근거

- [Custom agents와 역할 파일 우선순위](https://learn.chatgpt.com/docs/agent-configuration/subagents#custom-agents)
- [프로젝트 설정 로드와 신뢰](https://learn.chatgpt.com/docs/config-file/config-basic)
- [지원 모델 안내](https://learn.chatgpt.com/docs/models)
