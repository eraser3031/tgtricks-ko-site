# 리포트 개요

웹: /docs/ · Markdown: /docs/index.md

> 결론, 범위와 방법, 장 지도, 공통 패턴.

## 결론 [#conclusion]

이 리포트는 Telegram 공식 iOS 클라이언트와 macOS 클라이언트의 소스 코드를 분석해요. 두 플랫폼이 UIKit과 AppKit 위에서 택한 비정형적 기법을 정리했어요. 주제 노트 15편에 기법 300건을 담았고, 각 항목에는 고정 커밋 기준의 소스 행 번호가 붙어 있어요. 이 중 88건이 리포트 본문에 선정됐고(iOS 61건, macOS 27건), 사이트에서는 “리포트 선정”으로 표시해요.

iOS 클라이언트의 기법은 세 부류로 모여요.

1. **UIKit의 레이아웃·스크롤 스택 우회**: AsyncDisplayKit 노드, 보이지 않는 `UIScrollView`, π만큼 회전한 리스트로 UI를 직접 만들어요.
2. **시스템 비공개 객체 직접 조작**: 키보드 윈도우, `_UIPortalView`, `_UILiquidLensView`, 여러 비공개 KVC 키가 여기에 속해요.
3. **렌더링 파이프라인 자체 관리**: `IOSurface` 아틀라스, Metal 트리플 버퍼링, 공유 `DisplayLink`를 써요.

macOS 클라이언트의 TGUIKit은 AppKit으로 UIKit의 관례를 재현해요. 좌표계를 뒤집어 통일하고, 암시적 애니메이션을 끄고, `NSTableView`의 행 높이를 직접 관리하고, 키 윈도우가 아닌 윈도우로 메뉴를 직접 그려요.

비공개 API는 시스템 통합에 몰려 있어요. 예는 이래요.

- `CAEmitterBehavior` 파티클 동작
- 비공개 확대 커서
- `NSTitlebarView` 재배치
- `AVMovableView` 깊이 파기
- Dock 아이콘 리소스 포크
- 전역 `CGEvent` tap

소스 분석을 마친 뒤, 나는 macOS 클라우드 머신에서 REA로 공식 Telegram for macOS 13.0 바이너리를 분석해 보충 검증했어요. 결과는 이래요.

- 대조한 20개 항목 중 16개는 릴리스 빌드에서 직접 증거를 확인했어요.
- 2개는 버전 차이였어요.
- macOS 26.7에서는 비공개 경로 2건이 이미 시스템 구조와 맞지 않았어요.
- iOS 바이너리는 FairPlay로 암호화되어 있어서 검증하지 못했어요. 자세한 내용은 [REA 바이너리 검증](/docs/rea/)을 참고해요.

> **이 사이트 사용법**
>
> 조건별로 기법을 찾으려면 [기법 색인](/docs/explorer/)을 써요. 양 플랫폼의 해법을 비교하려면 [양 플랫폼 비교](/docs/compare/)를 써요. API 이름이나 파일 이름을 전문 검색하려면 ⌘K 또는 Ctrl K를 눌러요.

## 범위와 방법 [#scope]

| 항목 | Telegram-iOS | TelegramSwift(macOS) |
| --- | --- | --- |
| 저장소 | [TelegramMessenger/Telegram-iOS](https://github.com/TelegramMessenger/Telegram-iOS) | [overtake/TelegramSwift](https://github.com/overtake/TelegramSwift) |
| 분석 커밋 | `f1dd7a2d`(2026-10-07) | `579cebbf`(2025-07-29, `master` 최신) |
| 소스 규모 | Swift 파일 5,493개, ObjC/ObjC++ 파일 913개 | Swift 파일 1,265개, ObjC 파일 88개 |
| 주제 노트 | 01–11(11편) | 12–15(4편) |
| 바이너리 검증 | 하지 않음(App Store IPA 암호화) | 공식 13.0(build 283305) |

분석은 세 단계로 했어요.

1. **소스 정독.** 하위 작업 15개가 주제별로 소스를 병렬로 읽었어요. 그중 09번 주제(채팅 화면)는 따로 보충했어요. 모든 기법에 “무엇을 했나, 왜 했나, 어떻게 구현했나, 위험”을 적고 소스 행 번호를 붙였어요.
2. **병합과 선정.** 나는 주제 노트 15편을 합치고 중복을 없앤 뒤, 기법의 독창성과 차용 가능성을 기준으로 리포트 본문 항목 88건을 골랐어요.
3. **바이너리 검증.** 나는 macOS 26.7 클라우드 머신에서 REA 6.7.0과 Rizin 0.9.1로 릴리스 바이너리를 분석하고, 시스템에 비공개 클래스와 selector가 아직 있는지 실제로 재 봤어요.

## 장 지도 [#chapters]

각 장은 주제 노트 한 편에 해당해요. 오른쪽 숫자는 “항목 수 · 선정 수”예요.

**iOS · UIKit(Telegram-iOS)**

- [01 런타임과 비공개 API](/docs/ios/01-runtime-private-api/): 메서드 스위즐링, 비공개 클래스 이름 난독화, XOR 바이트열, 정밀한 ABI 호출. (24건, 선정 6)
- [02 윈도우, 키보드, 상태 표시줄](/docs/ios/02-window-keyboard-statusbar/): 키보드 윈도우 장악, 키보드 높이 보정, 상태 표시줄과 윈도우 계층 위조. (22건, 선정 6)
- [03 레이아웃과 애니메이션](/docs/ios/03-layout-animation/): 비공개 모서리 반경, 시스템 스프링 가로채기, additive 애니메이션, 모델 선반영. (22건, 선정 6)
- [04 리스트와 스크롤](/docs/ios/04-listview-scroll/): 보이지 않는 UIScrollView, π 회전 역순 채팅, 트랜잭션식 레이아웃. (23건, 선정 6)
- [05 ASDK와 텍스트](/docs/ios/05-asdk-text/): AsyncDisplayKit 비동기 드로잉, 직접 그린 선택 영역, IME 수정. (20건, 선정 5)
- [06 내비게이션, 제스처, 컨텍스트 메뉴](/docs/ios/06-navigation-gesture-context/): _UIPortalView 레이어 간 표시, 컨텍스트 메뉴, 제스처 중재. (20건, 선정 5)
- [07 시각 효과와 글래스](/docs/ios/07-visual-effects-glass/): 비공개 Liquid Lens, CABackdropLayer, 가변 블러. (20건, 선정 5)
- [08 렌더링과 성능](/docs/ios/08-rendering-perf/): IOSurface 아틀라스, Metal 트리플 버퍼링, 공유 DisplayLink. (22건, 선정 4)
- [09 채팅 화면](/docs/ios/09-chat-screen/): 입력창 포커스 인계, 메시지 전환, 리스트 도달 판정. (18건, 선정 8)
- [10 미디어, Stories, 프로필 화면](/docs/ios/10-media-stories-peerinfo/): 파티클 동작, 제로 카피 비디오 텍스처, 프로필 화면 풀다운 갤러리. (15건, 선정 5)
- [11 비 UI 코어](/docs/ios/11-core-nonui/): 이중 가상 매핑 링 버퍼, 공유 SQLite 보호, 샘플러. (15건, 선정 5)

**macOS · AppKit(TelegramSwift)**

- [12 TGUIKit 코어](/docs/macos/12-tguikit-core/): AppKit으로 UIKit 재현: 무키 윈도우 메뉴, 키보드 디스패치, 레이어 변환. (21건, 선정 6)
- [13 리스트, 스크롤, 텍스트](/docs/macos/13-table-scroll-text/): NSTableView 행 높이 직접 관리, 스크롤 앵커, Core Text 선택 영역. (22건, 선정 7)
- [14 비공개 시스템 통합](/docs/macos/14-private-system/): CAEmitterBehavior, CGEvent tap, Dock 리소스 포크, 시스템 뷰 트리 개조. (18건, 선정 7)
- [15 렌더링과 앱 계층](/docs/macos/15-rendering-app/): LZFSE 스티커 캐시, 공유 디코딩 컨텍스트, Metal 일괄 제출. (18건, 선정 7)

## 공통 패턴 [#patterns]

**애니메이션과 전환.** 두 클라이언트 모두 표시 레이어를 진실의 원천으로 삼아요.

- 레이아웃은 모델 레이어에 먼저 쓰고, 명시적 애니메이션이 화면을 뒤따라가게 해요.
- `presentation()`과 additive 애니메이션이 중단된 애니메이션을 이어받아요.
- 안정적인 식별자와 지연 소멸이 diff 전환을 유지해요.

**플랫폼별 접근.**

- iOS는 `UIScrollView`의 물리 특성과 비공개 윈도우로 새 스택을 조립해요.
- macOS는 UIKit 관용구를 재현하고 Core Animation의 수명 주기를 명시적으로 관리해요.
- 두 플랫폼 모두 Portal 방식으로 픽셀을 재사용해요.
- 텍스트와 스티커는 모두 “자리 표시 지오메트리와 별도 렌더링” 방식을 써요.

**렌더링과 리스트.**

- 렌더링은 `IOSurface`, 픽셀 풀, 필요할 때 제출하는 방식에 기대요.
- 리스트는 회전하거나 뒤집은 좌표로 앵커 의미를 지켜요.
- 알고리즘과 캐시는 대체로 비공개 API에 기대지 않아요.
- 재생과 디코딩 컨텍스트를 공유해 중복 비용을 줄여요.

**시스템 차이와 비공개 API.**

- 상태 표시줄 프록시와 키보드 프록시가 시스템 간 차이를 격리해요.
- 복잡한 전환은 먼저 Portal을 미러링한 다음 실제 뷰 트리를 돌려줘요.
- 비공개 진입점은 꼭 필요한 곳에서만 쓰고, 가능한 한 ABI를 스스로 점검해요.

## 항목 형식 [#format]

장 페이지의 각 기법은 같은 구조예요.

- **메타 정보**(제목 아래): 항목 번호, 소스 프로젝트, 프레임워크, 검증 방식과 태그, 그리고 고정 커밋을 가리키는 소스 링크예요.
- **본문**: “무엇을 했나”, “왜 이렇게 했나”, “핵심 코드 발췌”, “대가와 위험” 순서예요.
- **증거 체인**(마지막): 소스 확인 여부, 리포트 요약, 바이너리 검증 결과를 모았어요.

| 표시 | 의미 |
| --- | --- |
| 리포트 선정 | 이 항목이 리포트 본문에 들어갔다는 뜻이에요. 번호는 `01-1` 형태예요 |
| 소스 분석 | 고정 커밋에서 한 줄씩 확인했어요 |
| REA 바이너리 | 13.0 릴리스 빌드의 arm64 슬라이스에서 문자열, import, 클래스 테이블 증거를 찾았어요 |
| macOS 26.7 실측 | 점검 프로그램으로 비공개 클래스, selector, 뷰 트리가 아직 있는지 확인했어요 |
| REA 런타임 관측 | LLDB에서 13.0 사본을 40초간 실행하며 브레이크포인트 호출을 기록했어요 |
| 비공개 API, 런타임 해킹 | 비공개 인터페이스에 기대거나 런타임 동작을 고쳐 쓰니까, 차용하기 전에 [위험 페이지](/docs/risk/)를 먼저 봐요 |

- [기법 색인](/docs/explorer/): 프레임워크, 프로젝트, 태그, 검증 방식으로 기법 300건을 걸러요.
- [양 플랫폼 비교](/docs/compare/): 주제 14개 묶음에서 UIKit과 AppKit의 해법을 나란히 비교해요.
- [REA 바이너리 검증](/docs/rea/): 20개 항목의 대조 결과, 컴파일 산출물 분석, 런타임 관측.
- [위험과 차용 가능성](/docs/risk/): 비공개 API 위험 등급과 안전하게 차용할 수 있는 방법.
