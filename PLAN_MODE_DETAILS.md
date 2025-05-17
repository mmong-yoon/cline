# Plan 모드 코드 구조 상세 설명

이 문서는 Cline 프로젝트에서 **Plan 모드**와 관련된 소스 코드의 흐름과 각 파일이 담당하는 역할을 정리한 것입니다. Plan 모드는 사용자의 요청에 대해 직접 작업을 수행하기 전에 정보를 수집하고 실행 계획을 세우기 위한 모드입니다. Act 모드와 상호 전환되며, 모드 전환 과정과 관련 상태가 다양한 파일에 걸쳐 구현되어 있습니다.

## 1. 핵심 개념 개요

`.clinerules/cline-overview.md`에서는 Plan/Act 모드 시스템을 다음과 같이 설명합니다.

- 모드 상태는 `chatSettings.mode` 값으로 관리되며, 컨트롤러가 이를 저장하고 갱신합니다.
- 모드 전환은 `togglePlanActModeWithChatSettings` 함수가 담당합니다.
- 필요에 따라 Plan 모드와 Act 모드에서 서로 다른 모델을 사용할 수 있습니다.
- 모드별 프롬프트가 구분되어 있어 계획 단계와 실행 단계가 명확히 나뉩니다.
- Plan 모드에서는 `plan_mode_respond` 도구를 사용해 사용자의 질문에 답하거나 계획을 제시합니다.【F:.clinerules/cline-overview.md†L507-L548】

또한 `docs/exploring-clines-tools/plan-and-act-modes-a-guide-to-effective-ai-development.mdx` 문서에서는 Plan 모드가 정보 수집과 전략 수립에, Act 모드가 실제 실행에 초점을 맞춘다고 설명합니다.【F:docs/exploring-clines-tools/plan-and-act-modes-a-guide-to-effective-ai-development.mdx†L18-L70】

## 2. 상태 관리와 설정

### 2.1 `ChatSettings`
`src/shared/ChatSettings.ts`에 정의된 `ChatSettings` 인터페이스는 `mode` 속성으로 `"plan" | "act"` 값을 갖습니다. 기본값은 Act 모드입니다.【F:src/shared/ChatSettings.ts†L3-L15】

### 2.2 전역 상태 로딩
`src/core/storage/state.ts`의 `getAllExtensionState` 함수에서는 Plan/Act 모드 관련 설정을 로딩합니다. 특히 `planActSeparateModelsSetting` 값을 처리해 새 사용자와 기존 사용자의 기본값을 다르게 설정합니다.【F:src/core/storage/state.ts†L270-L289】【F:src/core/storage/state.ts†L358-L384】

## 3. 모드 전환 로직

### 3.1 컨트롤러의 전환 함수
모드 전환은 `src/core/controller/index.ts`의 `togglePlanActModeWithChatSettings` 메서드에서 수행됩니다. 이 함수는 다음 작업을 처리합니다:

1. 텔레메트리로 모드 전환 이벤트를 기록합니다.
2. 기존 모드에서 사용 중이던 모델 정보를 저장하고, 새 모드로 전환할 때 이전 모드의 모델을 복원합니다(필요 시).
3. 전환 후 `chatSettings`를 갱신하고 웹뷰에 새로운 상태를 전파합니다.
4. Plan 모드에서 Act 모드로 전환될 때, Plan 모드 응답을 기다리던 상태라면 자동으로 메시지를 전송해 모드 변경 사실을 알립니다.【F:src/core/controller/index.ts†L669-L833】

설정 변경 시에는 `updateSettings` 처리 과정에서 `planActSeparateModelsSetting` 값 또한 저장됩니다.【F:src/core/controller/index.ts†L570-L599】

### 3.2 웹뷰 측 처리
`webview-ui/src/components/chat/ChatTextArea.tsx`에서는 토글 스위치를 통해 모드를 변경할 수 있습니다. 버튼 클릭 시 `togglePlanActMode` 메시지가 확장 프로그램으로 전송되며, 현재 입력 내용이 있을 경우 함께 전달됩니다. 또한 단축키 `Meta+Shift+A`로도 모드를 전환할 수 있습니다.【F:webview-ui/src/components/chat/ChatTextArea.tsx†L960-L989】【F:webview-ui/src/components/chat/ChatTextArea.tsx†L1677-L1693】

사용자가 다른 모델을 Plan 모드와 Act 모드에 각각 지정하도록 허용하는 옵션은 설정 화면(`SettingsView.tsx`)에서 `Use different models for Plan and Act modes` 체크 박스로 제공됩니다.【F:webview-ui/src/components/settings/SettingsView.tsx†L160-L230】

컨텍스트 저장소(`ExtensionStateContext.tsx`)에서도 `planActSeparateModelsSetting` 값을 관리하고 변경 시 확장 쪽에 업데이트 메시지를 보냅니다.【F:webview-ui/src/context/ExtensionStateContext.tsx†L70-L109】【F:webview-ui/src/context/ExtensionStateContext.tsx†L288-L334】

## 4. 시스템 프롬프트와 Plan 모드 도구

`src/core/prompts/system.ts`에는 Plan 모드에서 사용할 수 있는 특별한 도구 `plan_mode_respond`가 정의되어 있습니다. 이 도구는 사용자의 질문에 대한 답변이나 계획 제시를 위해 사용되며, 반드시 `<plan_mode_respond><response>...</response></plan_mode_respond>` 형태로 응답해야 합니다.【F:src/core/prompts/system.ts†L251-L258】

또한 `src/core/prompts/responses.ts`의 `planModeInstructions` 함수는 Plan 모드일 때 사용자에게 집중해야 할 행동 지침을 제공합니다. 도구 사용 시점을 명확히 안내하며, Act 모드 전환을 직접 수행할 수 없음을 강조합니다.【F:src/core/prompts/responses.ts†L156-L161】

## 5. Task 클래스에서의 Plan 모드 처리

`src/core/task/index.ts`의 `Task` 클래스는 Plan 모드 응답을 다루는 로직을 포함합니다.

- 클래스 멤버 변수 `isAwaitingPlanResponse`와 `didRespondToPlanAskBySwitchingMode`는 Plan 모드 대화 흐름을 제어하기 위해 사용됩니다.【F:src/core/task/index.ts†L140-L153】
- `plan_mode_respond` 블록을 처리할 때, 응답 내용과 옵션을 파싱하여 텔레메트리 이벤트를 기록하고, 필요한 경우 Act 모드 전환 시 전달된 메시지를 사용자 피드백으로 변환합니다.【F:src/core/task/index.ts†L3355-L3447】

## 6. 텔레메트리 연동

모드 전환 및 Plan 모드 상호작용은 `src/services/posthog/telemetry/TelemetryService.ts`에서 추적됩니다. 대표적으로 다음과 같은 메서드가 제공됩니다.

- `captureModeSwitch` : Plan↔Act 전환 시 호출되어 전환된 모드 정보를 기록합니다.【F:src/services/posthog/telemetry/TelemetryService.ts†L307-L321】
- `captureOptionSelected` / `captureOptionsIgnored` : Plan 모드에서 제공된 선택지 사용 여부를 수집합니다.【F:src/services/posthog/telemetry/TelemetryService.ts†L696-L732】

## 7. 사용자 문서

`docs/exploring-clines-tools/plan-and-act-modes-a-guide-to-effective-ai-development.mdx` 문서에서는 Plan 모드와 Act 모드의 차이점, 전환 시나리오, 권장 워크플로 등을 그림과 함께 설명합니다. Plan 모드에서는 요구사항 파악과 계획 수립에 집중하고, 충분히 계획이 세워지면 Act 모드로 전환하여 실행하도록 안내합니다.【F:docs/exploring-clines-tools/plan-and-act-modes-a-guide-to-effective-ai-development.mdx†L18-L90】

## 8. 정리

Plan 모드는 Cline의 개발 흐름에서 중요한 부분을 차지합니다. 파일 읽기·질문 도구 등을 활용하여 충분한 정보를 수집하고, `plan_mode_respond` 도구를 통해 사용자와 상호 소통하며 실행 계획을 다듬습니다. 이러한 과정은 컨트롤러와 Task 클래스, 웹뷰 UI, 프롬프트 정의, 텔레메트리 서비스 등 여러 모듈이 협력하여 동작합니다. 위에 정리된 코드 위치를 참고하면 Plan 모드 기능의 전체 구조를 이해할 수 있습니다.

