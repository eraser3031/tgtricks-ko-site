# 참고 자료와 용어

웹: /docs/reference/ · Markdown: /docs/reference.md

> 소스 코드 저장소와 커밋, 바이너리와 도구, 용어집, 연구 범위의 한계.

## 소스 코드 저장소 [#repos]

| 프로젝트 | 저장소 | 분석 커밋 | 설명 |
| --- | --- | --- | --- |
| iOS | [TelegramMessenger/Telegram-iOS](https://github.com/TelegramMessenger/Telegram-iOS/tree/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838) | `f1dd7a2d` (2026-10-07) | Bazel 빌드, UI는 AsyncDisplayKit 포크와 자체 개발한 `Display` 모듈 기반 |
| macOS | [overtake/TelegramSwift](https://github.com/overtake/TelegramSwift/tree/579cebbf0c01fd41b712eff3647fa7f69db9665d) | `579cebbf` (2025-07-29) | UI는 자체 개발한 TGUIKit 기반, `telegram-ios` 서브모듈의 공유 모듈을 정적 링크 |

사이트의 모든 소스 코드 링크는 위 표의 고정 커밋을 가리켜요. 그래서 저장소가 업데이트되어도 행 번호가 어긋나지 않아요.

## 바이너리와 도구 [#tools]

| 이름 | 용도 | 링크 |
| --- | --- | --- |
| Telegram for macOS 13.0 | 검증 대상 릴리스 빌드 바이너리(build 283305) | [telegram.org/dl/macos](https://telegram.org/dl/macos) |
| REA 6.7.0 | 바이너리 분석 오케스트레이션 도구. Mach-O, 서명, 로드 관계, Rizin 명령, LLDB 런타임 관측을 제공 | [morluto/rea](https://github.com/morluto/rea) |
| Rizin 0.9.1 | REA의 `execute-rizin-command` 백엔드. 문자열, 클래스 테이블, 디스어셈블리를 내보내요 | [rizinorg/rizin](https://github.com/rizinorg/rizin) |
| Fumadocs | 이 사이트가 사용하는 문서 프레임워크 | [fumadocs.dev](https://fumadocs.dev) |

## 용어집 [#glossary]

| 용어 | 정의 |
| --- | --- |
| AsyncDisplayKit(ASDK) | Facebook이 오픈소스로 공개했고 이후 Texture로 이름이 바뀐 비동기 UI 프레임워크예요. Telegram-iOS는 자체 포크를 유지하며 `ASDisplayNode`로 백그라운드 스레드에서 레이아웃과 그리기를 해요 |
| TGUIKit | TelegramSwift의 자체 UI 라이브러리예요. AppKit으로 UIKit의 뷰, 리스트, 내비게이션 관습을 재현해요 |
| 비공개 API | Apple SDK 헤더에 선언되어 있지 않은 클래스, 메서드, selector 또는 KVC 키 |
| KVC | Key-Value Coding이에요. 문자열 키로 객체 속성을 읽고 쓰며, 비공개 속성도 이렇게 접근할 수 있어요 |
| IMP | Objective-C 메서드의 함수 포인터예요. IMP를 얻으면 C 함수처럼 호출하거나 자체 구현으로 바꿔 끼울 수 있어요 |
| 메서드 스위즐링(swizzling) | 런타임에 특정 selector에 대응하는 IMP를 맞바꾸거나 다른 것으로 바꿔 끼우는 일 |
| ISA 교체 | 런타임에 객체의 클래스 포인터를 자동 생성한 서브클래스로 바꿔서 개별 인스턴스의 동작을 바꾸는 일 |
| Portal | 같은 레이어 내용을 다른 위치에 보여 주는 비공개 메커니즘이에요. 예: `_UIPortalView`, `CAPortalLayer` |
| `CABackdropLayer` | 자기 아래에 있는 화면 내용을 샘플링하는 비공개 레이어예요. 시스템 반투명 블러(frosted glass)의 하위 구현이에요 |
| `CAFilter` | Core Animation의 비공개 필터 클래스예요. 가우시안 블러, 컬러 매트릭스, 디스플레이스먼트 맵 같은 효과를 만들 수 있어요 |
| additive 애니메이션 | `isAdditive = true`인 애니메이션이에요. 값이 모델 값 위에 더해지므로 이전 위치에서 새 위치로 부드럽게 따라잡을 수 있어요 |
| presentation layer | 레이어가 지금 화면에 보이는 상태예요. 애니메이션을 중단할 때 여기서 시작점을 가져오면 튀는 현상을 피할 수 있어요 |
| 디스플레이 링크 | `CADisplayLink`(iOS) 또는 `CVDisplayLink`(macOS)예요. 화면 주사율에 맞춰 콜백해요 |
| `IOSurface` | 프로세스와 GPU 사이에서 공유할 수 있는 픽셀 버퍼 |
| Swift 작은 문자열 | 15바이트 이내의 Swift `String`은 별도 메모리 할당 없이 머신 워드 두 개에 바로 저장돼요. 컴파일 후에는 즉치값 형태로 나타나는 경우가 많아요 |
| `MOVZ/MOVK` | arm64에서 16비트 즉치값을 레지스터의 지정한 위치에 쓰는 명령어예요. 짧은 문자열은 흔히 이 명령어로 조립돼요 |
| FairPlay | App Store가 iOS 실행 파일에 적용하는 암호화예요. 그래서 iOS 릴리스 빌드는 바로 정적 분석을 할 수 없어요 |
| hardened runtime | macOS의 런타임 보호예요. 켜져 있으면 `get-task-allow`가 없는 프로세스에는 디버거를 붙일 수 없어요 |
| ad-hoc 서명 | 개발자 인증서를 쓰지 않는 로컬 서명이에요. 수정한 사본을 로컬에서 실행할 때 써요 |
| `CGEvent` tap | 시스템 이벤트 스트림에 끼워 넣는 리스너예요. 다른 앱의 키보드와 마우스 이벤트를 받을 수 있어요 |
| 리소스 포크, FinderInfo | HFS+/APFS 파일의 부가 데이터 스트림과 Finder 메타데이터 비트예요. Telegram은 이걸로 자기 bundle에 커스텀 아이콘을 설정해요 |
| Evidence ID | REA가 분석 출력마다 만드는 콘텐츠 해시예요. 결과를 다시 확인할 때 써요 |

## 연구 범위의 한계 [#limits]

- 이 사이트의 내용은 공개 소스 코드의 정적 분석과 macOS 13.0에 대한 한 차례의 바이너리 검증에서 나왔어요.
- 런타임 관측은 시작 화면과 로그인 화면만 다뤄요. 계정에 로그인하지 않았고 채팅 화면에도 들어가지 않았어요.
- iOS 부분은 전부 소스 분석이고 바이너리 검증은 하지 않았어요.
- 항목에 나오는 비공개 API 사용은 기술 연구 목적으로만 다뤄요. 가져다 쓰기 전에 [위험과 차용 가능성](/docs/risk/)을 먼저 참고해요.

## 데이터 출처 [#data]

사이트는 이런 자료로 만들어져요.

- 주제 노트 15개
- REA 검증 노트 1개
- 통합 리포트 1개

노트는 프로젝트 저장소의 `source-notes/` 디렉터리에 저장되어 있어요. `scripts/build_content.py`가 노트를 장별 MDX와 항목 데이터로 변환해요. 그래서 스크립트를 다시 실행하면 사이트 내용이 갱신돼요.
