# 프로젝트 구조 설명

**주의**: 이 문서는 저장소의 폴더 구조와 주요 파일들의 역할을 간략히 요약한 것입니다. 모든 파일을 일일이 설명하기에는 양이 방대하므로, 핵심적인 파일과 디렉터리를 중심으로 정리했습니다.

## 최하단 파일들
- `assets/` : 확장 프로그램에서 사용하는 아이콘과 GIF 이미지 등 정적 리소스가 위치합니다.
- `assets/icons/icon.png` 등 : VS Code 마켓플레이스에 표시될 확장 아이콘 파일입니다.
- `standalone/runtime-files/vscode/` : standalone 모드에서 VS Code API를 에뮬레이션하기 위한 스텁/구현 스크립트들이 있습니다.

## src 하위 구조
`src/` 폴더는 확장 프로그램의 대부분의 로직을 포함합니다.

- `src/extension.ts` : VS Code 확장의 진입점으로, 확장 활성화 시 실행되는 코드가 정의되어 있습니다.
- `src/api/` : 다양한 LLM API 제공업체와 통신하는 핸들러를 모아둔 곳입니다.
    - `providers/` : OpenAI, Anthropic 등 각 서비스별 구현체 파일들이 있습니다.
    - `transform/` : API 응답 스트림을 공통 포맷으로 변환하기 위한 유틸리티 코드가 위치합니다.
- `src/core/` : Cline의 핵심 동작을 담당합니다.
    - `controller/` : gRPC 서비스 핸들러 및 작업 흐름 관리 로직이 포함됩니다.
    - `context/`, `task/` 등 : 작업 상태 관리와 맥락 구성을 위한 모듈입니다.
- `src/integrations/` : 편집기, 터미널, Git, 브라우저 등 외부 도구와의 연동을 담당합니다.
    - `editor/DiffViewProvider.ts` : 파일 생성/수정 시 diff 뷰를 보여주는 기능을 구현합니다.
    - `terminal/` : VS Code 터미널에서 명령을 실행하고 출력을 수집하는 로직이 있습니다.
- `src/services/` : 계정 관리, 브라우저 세션, 로깅 등 독립적인 서비스 객체들이 모여 있습니다.
- `src/shared/` : 확장과 웹뷰 간에 공유되는 타입 정의가 위치합니다.

## webview-ui
웹뷰(React 기반)의 소스가 들어 있습니다. `App.tsx`를 포함해 각종 컴포넌트와 context가 존재하며, 확장과 메시지를 주고받아 UI를 표시합니다.

## proto
gRPC 통신에 사용되는 protobuf 정의와 빌드 스크립트가 위치합니다. `build-proto.js`를 실행해 타입스크립트 코드와 descriptor set을 생성합니다.

## docs
사용자 문서와 가이드가 있는 폴더입니다. Getting Started, 각종 기능 설명서, 프롬프트 팁 등 마크다운 파일로 제공됩니다.

## evals
`cline-eval` CLI 도구와 관련된 코드가 들어 있습니다. 벤치마크 실행, 리포트 생성 등의 명령을 제공합니다.

## 기타 루트 파일
- `package.json` : VS Code 확장 메타데이터와 의존성, 명령/키바인딩 등록 등이 정의되어 있습니다.
- `README.md` : 프로젝트 소개와 사용 방법, 기여 방법 등이 기술되어 있습니다.
- `esbuild.js` : 확장 빌드 설정을 위한 스크립트입니다.

이상으로 저장소의 주요 디렉터리와 파일들의 역할을 간략히 정리했습니다. 자세한 구현 내용은 각 파일의 코드를 참고하세요.
