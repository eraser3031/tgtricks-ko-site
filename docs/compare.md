# 양 플랫폼 비교

웹: /docs/compare/ · Markdown: /docs/compare.md

> 14개 주제로 Telegram-iOS와 TelegramSwift가 같은 문제를 어떻게 푸는지 나란히 비교해요.

두 클라이언트는 서로 다른 팀이 유지보수하지만 마주한 문제는 같아요. 긴 리스트, 역순 채팅, 리치 텍스트, 애니메이션 스티커, 시스템 메뉴, 키보드가 그 문제예요.

각 비교는 이 순서로 구성돼요.

1. 공통 문제를 먼저 적어요.
2. iOS와 macOS의 방식 및 관련 항목을 하나씩 나열해요.
3. 마지막에 차이를 정리해요.

왼쪽 목록이나 방향키로 주제를 바꾸고, 오른쪽 위에서 한쪽 플랫폼만 볼 수 있어요.

### 뷰 기초: 노드와 레이어

문제: 기본 뷰 스택에는 암시적 애니메이션(속성을 바꾸면 시스템이 저절로 붙이는 애니메이션)과 메인 스레드 그리기 비용이 따라와요. 두 플랫폼 모두 이 두 가지를 우회해야 해요.

- iOS(UIKit): AsyncDisplayKit 노드로 그리기를 백그라운드로 옮겨요. 무효화될 때마다 sentinel(작업이 아직 유효한지 가리는 카운터)을 증가시켜 이전 백그라운드 작업을 무효로 만들어요. layer-backed 노드(뷰 없이 `CALayer`만으로 동작하는 노드)는 약한 델리게이트를 써서 `CALayer.delegate`가 댕글링(이미 해제된 객체를 가리키는 상태)되는 것을 막아요. 관련 항목:
  - [#05-06 비동기 그리기는 원자적 sentinel로 오래된 프레임을 취소하고 메인 스레드에서 원자적으로 결과를 반영하기](/docs/ios/05-asdk-text/#t05-06)
  - [#05-12 layer-backed 노드는 약한 프록시로 `CALayer.delegate` 댕글링 포인터 수정하기](/docs/ios/05-asdk-text/#t05-12)
  - [#03-12 `action(forKey:)`에서 암묵적 애니메이션 끄기와 마운트 추적을 동시에](/docs/ios/03-layout-animation/#t03-12)
- macOS(AppKit): TGUIKit(TelegramSwift의 UI 라이브러리)은 비동기 그리기를 하지 않아요. 모든 `View`가 기본적으로 layer-backed이고, 좌표를 통일해 뒤집고, 암시적 애니메이션을 꺼요. 변환 계열 속성은 `updateLayer`에서 명시적으로 동기화해요. 계층 진입과 이탈은 빈 `CAAction`으로 알려요. 관련 항목:
  - [#12-05 기본 View에서 좌표 뒤집기를 통일하고 암시적 애니메이션 끄기](/docs/macos/12-tguikit-core/#t12-05)
  - [#12-04 조건부 `updateLayer`로 UIKit 변환 속성 흉내 내기](/docs/macos/12-tguikit-core/#t12-04)
  - [#12-06 빈 CAAction으로 계층 진입 알림 겸용하기](/docs/macos/12-tguikit-core/#t12-06)
- 차이: iOS의 핵심은 그리기를 메인 스레드 밖으로 옮기는 거예요. macOS의 핵심은 AppKit과 UIKit의 관습 차이를 메우고 Core Animation이 명시적 애니메이션만 실행하게 하는 거예요. 두 플랫폼 모두 "빈 action"으로 계층 이벤트를 가로채요.

### 긴 리스트: 직접 만들까, 넘겨받을까

문제: 채팅 목록은 역순, 알 수 없는 전체 높이, 대량의 비동기 레이아웃을 지원해야 해요. 시스템 리스트 컨트롤은 이 세 가지 요구를 직접 지원하지 않아요.

- iOS(UIKit): `ListView`는 `UITableView`를 쓰지 않아요. `UIScrollView`를 숨겨 두고 드래그, 감속, 바운스만 빌려 써요. 전체 높이를 모를 때는 가짜 콘텐츠 높이로 스크롤을 받쳐요. 역순 채팅은 부모와 자식에 π 회전을 두 번 줘서 구현해요. 관련 항목:
  - [#04-01 보이지 않는 `UIScrollView`로 스크롤 물리만 가져오기](/docs/ios/04-listview-scroll/#t04-01)
  - [#04-02 20,000pt짜리 가짜 콘텐츠 높이로 아직 모르는 절반의 데이터 감당하기](/docs/ios/04-listview-scroll/#t04-02)
  - [#04-03 부모와 자식을 π만큼 두 번 회전해 뒤집힌 채팅의 글자 바로 세우기](/docs/ios/04-listview-scroll/#t04-03)
- macOS(AppKit): `TableView`는 `NSTableView`를 유지하되 자동 행 높이와 기본 배경을 끄고 행 지오메트리를 직접 맡아요. 윈도우 폭이 바뀔 때는 보이는 행만 다시 배치해요. 고정(sticky) 헤더는 스크롤 뷰와 같은 계층에 둬요. 관련 항목:
  - [#13-21 자동 행 높이와 기본 배경을 강제로 끄고 `NSTableView`의 행 기하를 완전히 장악하기](/docs/macos/13-table-scroll-text/#t13-21)
  - [#13-09 보이는 행만 재배치하고 수동 행 높이 알림으로 `NSTableView` 구동하기](/docs/macos/13-table-scroll-text/#t13-09)
  - [#13-05 스티키 헤더를 스크롤 뷰와 같은 계층으로 옮기고 뒤집힌 목록에 맞추기](/docs/macos/13-table-scroll-text/#t13-05)
- 차이: iOS는 시스템의 스크롤 물리만 빌리고 레이아웃은 전부 직접 관리해요. macOS는 `NSTableView`의 재사용과 이벤트 처리를 유지하고 행 높이와 장식 뷰만 넘겨받아요.

### 차분 업데이트와 뷰포트 앵커링

문제: 메시지를 삽입하거나 삭제할 때 뷰포트가 튀면 안 돼요. 사라지는 행에도 전환 애니메이션이 필요해요.

- iOS(UIKit): 트랜잭션이 정지 항목(화면에서 제자리에 머물게 할 기준 항목)을 지정할 수 있어요. 새 레이아웃이 끝나면 모든 frame을 통째로 평행 이동해서 그 정지 항목이 이전 화면 위치로 돌아오게 해요. 삭제할 때는 이전 frame을 유지하는 자리표시 노드를 먼저 넣어요. 비동기 레이아웃이 끝나는 순서는 트랜잭션 큐가 직렬화해요. 관련 항목:
  - [#04-07 정지된 항목으로 삽입·삭제 뒤 뷰포트 고정하기](/docs/ios/04-listview-scroll/#t04-07)
  - [#04-08 자리 표시 노드로 삭제 중인 행의 공간 지키기](/docs/ios/04-listview-scroll/#t04-08)
  - [#04-04 트랜잭션 큐로 비동기 레이아웃의 완료 순서 맞추기](/docs/ios/04-listview-scroll/#t04-04)
- macOS(AppKit): `mergeListsStableWithUpdatesReversed`가 역순 데이터의 삭제, 삽입, 업데이트 인덱스를 만들어요. 업데이트 전에 보이는 행의 `stableId`와 화면 위치를 저장하고, 업데이트 후 이 저장값을 근거로 `NSClipView`의 y를 다시 계산해요. 행 이동은 `presentation()` 레이어에서 이어받아요. 관련 항목:
  - [#13-08 역순 메시지 스트림을 위한 독립적인 증분 diff 작성하기](/docs/macos/13-table-scroll-text/#t13-08)
  - [#13-03 순수 offset 대신 안정적인 식별자로 스크롤 앵커 복원하기](/docs/macos/13-table-scroll-text/#t13-03)
  - [#13-07 presentation layer로 행 이동을 받아내고 Core Animation을 수동으로 보완하기](/docs/macos/13-table-scroll-text/#t13-07)
- 차이: 두 플랫폼 모두 "안정 식별자와 앵커"를 쓰고 날 offset은 저장하지 않아요. iOS는 중간 레이아웃 상태에서 보정을 끝내요. macOS는 `NSTableView` 업데이트 전후에 한 번씩 측정해요.

### 텍스트 조판과 인라인 이모지

문제: 메시지 텍스트에는 커스텀 이모지, spoiler(눌러서 펼치기 전까지 가려 두는 텍스트), 임베디드 콘텐츠가 들어 있어요. 시스템 텍스트 컨트롤은 그리기 과정을 제어할 수 없어요.

- iOS(UIKit): `TextNode`가 Core Text 그리기를 직접 작성해요. 커스텀 emoji는 한 글자로 대체하고 `CTRunDelegate`를 묶어 폭을 확보해요. 실제 애니메이션 이모지는 그렇게 확보한 자리에 겹쳐 놓아요. spoiler는 이중 레이어로 렌더링해요. 관련 항목:
  - [#05-05 구형 텍스트 노드는 `CTRunDelegate`로 커스텀 emoji를 하나의 조판 글리프로 위장하기](/docs/ios/05-asdk-text/#t05-05)
  - [#05-04 보이지 않는 1×1 `NSTextAttachment`로 공간을 확보한 뒤 실제 애니메이션 이모지 뷰를 덮어 씌우기](/docs/ios/05-asdk-text/#t05-04)
  - [#05-07 “기반 레이어 지우기 + 클리핑 재그리기 레이어”로 spoiler 텍스트를 이중 레이어로 렌더링하기](/docs/ios/05-asdk-text/#t05-07)
- macOS(AppKit): TGUIKit의 `TextNode`는 `NSTextView`를 우회해서 줄별, `CTRun`별로 그려요. 임베디드 콘텐츠는 텍스트 흐름 안에 자리만 차지하고, 독립 레이어가 렌더링해요. 커스텀 이모지는 `NSTextAttachment`가 아니라 glyph rect 위의 오버레이 레이어예요. 관련 항목:
  - [#13-11 `NSTextView`를 우회해 glyph run 단위로 직접 메시지 텍스트 그리기](/docs/macos/13-table-scroll-text/#t13-11)
  - [#13-12 임베디드 콘텐츠를 텍스트 흐름에 남겨 두고 독립 레이어로 렌더링하기](/docs/macos/13-table-scroll-text/#t13-12)
  - [#13-16 커스텀 이모지는 `NSTextAttachment`가 아니라 glyph rect 위의 오버레이](/docs/macos/13-table-scroll-text/#t13-16)
- 차이: 두 플랫폼 모두 "자리 확보용 지오메트리와 독립 렌더링"을 택했어요. iOS에는 TextKit과 Core Text 두 경로가 모두 있어요. macOS의 메시지 텍스트는 Core Text만 거치고, 입력창에서만 TextKit을 써요.

### 입력기와 입력창

문제: 중국어, 일본어, 한국어 입력기에는 조합 중인 텍스트가 있어요. 직접 만든 입력창은 조기 확정이나 후보 잘림이 생기기 쉬워요.

- iOS(UIKit): 구조화된 문서가 키보드에 "가상 줄바꿈"을 투영해서 한국어와 CJK의 문단 간 재조합 문제를 고쳐요. 조합 텍스트와 인라인 예측은 "확정"과 "폐기"의 두 경로로 나뉘어요. 입력창은 텍스트 측정값에 따라 늘어나고 최대 12줄까지 커져요. 관련 항목:
  - [#05-02 키보드에 “가상 줄바꿈”을 투영해 한글과 CJK의 문단 간 재조합 수정하기](/docs/ios/05-asdk-text/#t05-02)
  - [#05-03 IME 조합 텍스트와 인라인 예측을 “확정”과 “폐기”의 두 경로로 나누기](/docs/ios/05-asdk-text/#t05-03)
  - [#09-07 텍스트 측정값에 따라 늘어나되 최대 열두 줄까지인 입력창](/docs/ios/09-chat-screen/#t09-07)
- macOS(AppKit): 엔터 키는 marked text(입력기가 조합 중인 텍스트)가 없을 때만 전송을 시작해요. 입력창은 두 번째 TextKit 스택으로 높이를 측정해서 표시 레이아웃이 오염되지 않게 해요. 인용 블록은 line fragment(텍스트 한 줄이 차지하는 영역)를 다시 써서 불규칙한 여백을 확보해요. 관련 항목:
  - [#13-17 전송 키는 먼저 `hasMarkedText()`를 확인해 중국어 등 입력기 후보가 잘리지 않게 하기](/docs/macos/13-table-scroll-text/#t13-17)
  - [#13-14 입력창은 두 번째 TextKit 스택으로 높이를 측정해 표시 레이아웃 오염 피하기](/docs/macos/13-table-scroll-text/#t13-14)
  - [#13-15 `NSTextContainer`의 행 조각을 재정의해 인용 블록용 불규칙 여백 확보하기](/docs/macos/13-table-scroll-text/#t13-15)
- 차이: iOS는 `UITextInput`을 직접 구현해야 해서 문제가 텍스트 투영에 몰려요. macOS는 `NSTextView`를 유지하니까 문제가 전송 시점과 측정 격리에 몰려요.

### 애니메이션 이어받기와 타이밍

문제: 애니메이션 도중 레이아웃이 다시 바뀌어도 뷰가 시작점으로 되돌아가면 안 돼요.

- iOS(UIKit): 새 레이아웃을 먼저 모델 레이어(최종 값을 담은 레이어)에 쓰고, presentation layer(지금 화면에 보이는 상태를 담은 레이어)의 현재 frame에서 명시적 애니메이션을 시작해요. 이동은 additive 애니메이션(현재 값에 더해지는 애니메이션)으로 모델 값을 따라잡아요. `UIView.animate`가 만든 시스템 애니메이션은 지정한 곡선으로 바뀌어요. 관련 항목:
  - [#03-08 프레젠테이션 레이어에서 중단된 레이아웃 애니메이션 이어받기](/docs/ios/03-layout-animation/#t03-08)
  - [#03-10 additive 변위로 이미 갱신된 모델 frame 따라잡기](/docs/ios/03-layout-animation/#t03-10)
  - [#03-02 UIKit 트랜잭션 안의 스프링 애니메이션 탈취](/docs/ios/03-layout-animation/#t03-02)
- macOS(AppKit): 리스트에 항목을 넣을 때 `presentation()`의 현재 위치를 읽어 애니메이션을 이어 가요. `DisplayLinkAnimator`가 `CAMediaTimingFunction`의 베지어 곡선을 직접 풀어요. 부드러운 스크롤은 목표를 하나만 저장하고 끝날 때 검증해요. 관련 항목:
  - [#13-07 presentation layer로 행 이동을 받아내고 Core Animation을 수동으로 보완하기](/docs/macos/13-table-scroll-text/#t13-07)
  - [#12-13 CAMediaTimingFunction의 3차 베지어를 직접 풀기](/docs/macos/12-tguikit-core/#t12-13)
  - [#13-04 부드러운 스크롤에서 목표 지점을 저장하고 진행 중인 애니메이션을 동기적으로 마무리하기](/docs/macos/13-table-scroll-text/#t13-04)
- 차이: 두 플랫폼 모두 "모델이 먼저 자리 잡고 애니메이션이 화면을 따라잡는다"는 원칙을 지켜요. iOS는 시스템 애니메이션의 파라미터를 가로챌 수 있어요. macOS에는 시스템 애니메이션 파라미터를 가로챌 진입점이 없어서, 수치 애니메이션을 디스플레이 링크가 직접 구동하는 경우가 더 많아요.

### 스티커와 다중 애니메이션 렌더링

문제: 한 화면에 수십 개의 애니메이션 스티커가 있을 수 있어요. 하나씩 디코딩하고 제출하면 CPU와 GPU가 바닥나요.

- iOS(UIKit): 타이머 하나가 활성 항목을 모두 구동하고, 저전력 상태에서는 30 FPS로 낮춰요. 같은 리소스는 한 번만 디코딩하고 여러 약한 대상으로 팬아웃(하나의 결과를 여러 대상에 나눠 보내기)해요. 고정 백그라운드 큐 두 개가 병렬로 디코딩한 뒤 메인 스레드로 한꺼번에 돌려보내 제출해요. 관련 항목:
  - [#08-07 타이머 하나로 DCT 애니메이션 전체를 구동하고 배터리 상태에 따라 30 FPS로 낮추기](/docs/ios/08-rendering-perf/#t08-07)
  - [#08-08 같은 스티커는 한 번만 디코딩하고 여러 약한 대상으로 팬아웃하기](/docs/ios/08-rendering-perf/#t08-08)
  - [#08-09 고정 백그라운드 큐 두 개로 병렬 디코딩하고 메인 스레드에 일괄 제출하기](/docs/ios/08-rendering-perf/#t08-09)
- macOS(AppKit): 같은 인라인 스티커는 디코딩 및 재생 컨텍스트 하나를 공유해요. 같은 런루프의 그리기 호출을 Metal 커맨드 버퍼 하나로 합쳐요. 픽셀 캐시는 먼저 XOR 프레임 차분(이전 프레임과 XOR해 달라진 부분만 남기는 방식)을 하고 LZFSE(Apple의 압축 알고리즘)로 압축해요. 관련 항목:
  - [#15-02 같은 인라인 스티커가 디코딩과 재생 컨텍스트 하나를 공유하게 하기](/docs/macos/15-rendering-app/#t15-02)
  - [#15-03 런 루프 단위로 여러 스티커 드로잉을 Metal 커맨드 버퍼 하나로 합치기](/docs/macos/15-rendering-app/#t15-03)
  - [#15-01 XOR 프레임 차분 후 LZFSE로 스티커 픽셀 캐시 압축하기](/docs/macos/15-rendering-app/#t15-01)
- 차이: 두 플랫폼은 "같은 키, 같은 컨텍스트"라는 발상을 공유해요. iOS의 캐시 형식은 DCT와 YUVA 평면이에요. macOS는 XOR 차분에 LZFSE를 더하고, 런루프 단위로 Metal 제출을 합쳐요.

### 파티클과 spoiler: CAEmitterBehavior

문제: spoiler의 "투명 잉크"에는 어트랙터(파티클을 끌어당기는 중심)와 수명에 따라 변하는 파티클이 필요해요. 공개 API인 `CAEmitterLayer`는 이런 파티클 동작을 제공하지 않아요.

- iOS(UIKit): 코드가 클래스 이름 `CAEmitterBehavior`와 selector `behaviorWithType:`을 이어 붙이고, IMP(메서드 구현 포인터)를 얻은 뒤 C 함수 포인터로 호출해요. 동작 객체는 KVC(문자열 키로 속성을 읽고 쓰는 방식)로 `emitterBehaviors`에 기록해요. 관련 항목:
  - [#10-01 런타임으로 비공개 `CAEmitterBehavior` 호출하기](/docs/ios/10-media-stories-peerinfo/#t10-01)
  - [#07-15 Spoiler에서 비공개 `CAEmitterBehavior`로 반짝임과 인력 구현하기](/docs/ios/07-visual-effects-glass/#t07-15)
  - [#05-15 RichText spoiler는 표시 속성으로 글리프를 숨기고 안정적인 식별자로 파티클 레이어를 풀링하기](/docs/ios/05-asdk-text/#t05-15)
- macOS(AppKit): TelegramSwift도 같은 이어 붙이기와 함수 포인터 방식을 써요. 여기에 `emitterBehaviors.randomAttractor0`의 비공개 속성까지 애니메이션해서 잉크가 계속 흐르게 해요. 임베디드 레이어는 spoiler 마스크가 덮일 때 함께 숨겨져요. 관련 항목:
  - [#14-01 비공개 클래스 이름을 이어 붙이고 함수 포인터로 `CAEmitterBehavior` 호출하기](/docs/macos/14-private-system/#t14-01)
  - [#13-01 `CAEmitterLayer`의 비공개 동작으로 상호작용 가능한 투명 잉크 만들기](/docs/macos/13-table-scroll-text/#t13-01)
  - [#13-13 spoiler로 가릴 때 글자 위에 떠 있는 임베디드 레이어도 함께 숨기기](/docs/macos/13-table-scroll-text/#t13-13)
- 차이: 두 플랫폼의 코드는 거의 같아요. REA는 macOS 13.0 바이너리에서 클래스 이름 조각과 KVC 경로를 찾았고, macOS 26.7에서 이 비공개 클래스를 여전히 만들 수 있음을 확인했어요. iOS 버전은 바이너리 검증을 하지 않았어요.

### 반투명 블러와 시각 효과

문제: 시스템 블러 뷰의 색조와 계층은 제어할 수 없어요. 디자인에는 더 깔끔하거나 특별한 글래스 효과가 필요해요.

- iOS(UIKit): 코드가 `UIVisualEffectView`의 내부 필터를 분해해서 `gaussianBlur`와 `colorSaturate`만 남겨요. 구형 시스템에서는 비공개 `CABackdropLayer`에 컬러 매트릭스를 직접 얹어요. iOS 26에서는 비공개 `_UILiquidLensView`를 직접 구동해요. 관련 항목:
  - [#07-08 `UIVisualEffectView` 내부 필터를 분해해 제어 가능한 내비게이션 블러 얻기](/docs/ios/07-visual-effects-glass/#t07-08)
  - [#07-09 `CABackdropLayer`와 컬러 매트릭스로 구버전 글래스를 직접 만들기](/docs/ios/07-visual-effects-glass/#t07-09)
  - [#07-01 iOS 26의 비공개 Liquid Lens를 직접 구동하기](/docs/ios/07-visual-effects-glass/#t07-01)
- macOS(AppKit): `VisualEffect`는 `updateLayer`가 끝날 때마다 반투명 블러 하위 트리를 정리해서 backdrop 레이어와 필터 두 가지만 남겨요. 레이어 착색에는 공개되지 않은 KVC 키 `contentsMultiplyColor`를 써요. 관련 항목:
  - [#12-02 글래스 효과의 내부 레이어와 필터 삭제하기](/docs/macos/12-tguikit-core/#t12-02)
  - [#12-07 비공개 KVC 키로 Layer 내용 색칠하기](/docs/macos/12-tguikit-core/#t12-07)
- 차이: 두 플랫폼 모두 화이트리스트(남길 것만 허용하는 목록)로 시스템 내부 레이어를 지워요. iOS는 더 멀리 나가 비공개 레이어와 Lens 뷰를 직접 만들어요. macOS 26.7에서 새로 추가된 `sdrNormalize` 필터는 화이트리스트에 걸려 지워져요. 화면에 미치는 영향은 미검증이에요.

### 메뉴와 컨텍스트 메뉴

문제: 시스템 메뉴의 스타일과 내용은 채팅 앱의 요구를 채우지 못해요. 아이콘 메뉴 항목, 서식 하위 메뉴, 커스텀 외형이 그런 예예요.

- iOS(UIKit): 코드가 메뉴 제목에 보이지 않는 접미사를 덧붙이고 `UILabel`의 그리기를 교환해서 글자를 아이콘으로 바꿔요. 에디터의 서식 메뉴는 식별자 기준으로 교체해요. 투명한 메뉴 배경에는 1×1 이미지를 채워 넣어서 메뉴 바깥 탭 문제를 고쳐요. 관련 항목:
  - [#01-07 `UIMenuItem`의 문자 렌더링을 아이콘 렌더링으로 바꿔치기](/docs/ios/01-runtime-private-api/#t01-07)
  - [#09-17 식별자로 에디터 서식 메뉴를 교체하고 시스템 메뉴의 나머지 항목은 유지하기](/docs/ios/09-chat-screen/#t09-17)
  - [#06-03 투명 배경에 1×1 이미지를 넣어 “보이지만 눌리지 않는” 메뉴 바깥 탭 고치기](/docs/ios/06-navigation-gesture-context/#t06-03)
- macOS(AppKit): `AppMenu`는 `NSMenu`의 네이티브 팝업을 쓰지 않고, 메뉴 단계마다 키 윈도우가 아닌 윈도우를 만들어 직접 그려요. 시스템 텍스트 우클릭 메뉴는 비공개 selector `_shareServiceSelected:`로 알아보고 하위 메뉴를 지워요. 관련 항목:
  - [#12-01 키 윈도우가 아닌 윈도우로 전체 기능의 AppMenu를 직접 그리기](/docs/macos/12-tguikit-core/#t12-01)
  - [#14-06 비공개 텍스트 메뉴 selector로 시스템 하위 메뉴 삭제하기](/docs/macos/14-private-system/#t14-06)
- 차이: iOS는 시스템 메뉴 내부에 패치를 해요. macOS는 아예 `NSMenu`를 버리고 윈도우를 직접 만들어요. macOS 26.7의 공유 항목은 이미 `_performStandardShareMenuItem:`을 써서, 앞서 말한 `_shareServiceSelected:`로 거르는 방식은 동작하지 않을 수 있어요.

### 윈도우, 키보드, 이벤트 디스패치

문제: 시스템 키보드 윈도우와 responder 체인(이벤트를 받을 객체를 차례로 찾아가는 연결)은 앱이 제어할 수 없어요. 그런데 채팅 입력, 전역 오버레이, 단축키는 모두 여기에 의존해요.

- iOS(UIKit): 코드가 비공개 키보드 씬 델리게이트로 키보드 윈도우를 얻고 시스템 키보드 레이어를 직접 움직여요. 키보드가 보일 때는 전역 오버레이를 키보드 윈도우에 다시 붙여요. 관련 항목:
  - [#02-01 비공개 키보드 scene delegate로 키보드 윈도우 얻기](/docs/ios/02-window-keyboard-statusbar/#t02-01)
  - [#02-03 앱 콘텐츠 대신 시스템 키보드 레이어를 직접 옮기기](/docs/ios/02-window-keyboard-statusbar/#t02-03)
  - [#02-13 키보드가 보이면 키보드 윈도우로 옮겨 붙는 전역 오버레이](/docs/ios/02-window-keyboard-statusbar/#t02-13)
- macOS(AppKit): `Window.sendEvent`가 전역 핸들러를 먼저 호출한 뒤 우선순위에 따라 키 입력을 나눠 줘요. 비활성 윈도우의 hover는 `.activeInActiveApp` tracking area에 기대요. 전역 푸시 투 토크(누르고 말하기)에는 리스닝 전용 `CGEvent` tap(이벤트를 가로채지 않고 듣기만 하는 감시기)을 써요. 관련 항목:
  - [#12-03 우선순위로 Window 키보드 디스패치 재작성하기](/docs/macos/12-tguikit-core/#t12-03)
  - [#12-10 `NSTrackingArea`로 비활성 윈도우 hover 처리하기](/docs/macos/12-tguikit-core/#t12-10)
  - [#14-02 리스닝 전용 `CGEvent` tap으로 전역 푸시투토크 구현하기](/docs/macos/14-private-system/#t14-02)
- 차이: iOS의 난점은 시스템 비공개 윈도우를 얻는 거예요. macOS의 난점은 responder 체인이 부족하다는 거라서, Telegram은 우선순위 디스패치를 직접 만들고 필요하면 앱 경계를 넘어 이벤트를 리스닝해요.

### 시스템 UI 개조: 상태 표시줄과 타이틀 바

문제: 시스템 상태 표시줄, 타이틀 바, 임베디드 컨트롤의 위치는 고정되어 있어서 커스텀 UI와 맞추기 어려워요.

- iOS(UIKit): 코드가 런타임에 비공개 상태 표시줄 클래스 이름을 이어 붙이고, 상태 표시줄을 캡처해서 픽셀 단위로 다시 착색해요. 투명한 가짜 스크롤 뷰가 "상태 표시줄을 탭해 맨 위로 이동" 제스처를 넘겨받아요. 관련 항목:
  - [#02-10 런타임에 비공개 상태 표시줄 클래스 이름 이어 붙이기](/docs/ios/02-window-keyboard-statusbar/#t02-10)
  - [#02-09 시스템 상태 표시줄을 캡처해 픽셀 단위로 다시 칠하기](/docs/ios/02-window-keyboard-statusbar/#t02-09)
  - [#02-16 상태 표시줄의 “맨 위로” 제스처를 가로채는 투명한 가짜 스크롤 뷰](/docs/ios/02-window-keyboard-statusbar/#t02-16)
- macOS(AppKit): 코드가 비공개 `NSTitlebarView`를 재귀적으로 찾아 내부 서브뷰를 재배치하고 50 pt 타이틀 바를 확보해요. 클래스 이름으로 `AVMovableView`를 깊이 파고들어 전체 화면 버튼을 클릭하고, `MKZoomSegmentedControl`을 옮겨요. 관련 항목:
  - [#14-05 `NSTitlebarView` 내부 하위 뷰를 재배치해 50 pt 타이틀 바 확보하기](/docs/macos/14-private-system/#t14-05)
  - [#14-04 `AVMovableView`를 깊이 파고들어 숨겨진 전체 화면 버튼 클릭하기](/docs/macos/14-private-system/#t14-04)
  - [#14-08 비공개 `MKZoomSegmentedControl` 클래스 이름으로 지도 확대/축소 컨트롤 옮기기](/docs/macos/14-private-system/#t14-08)
- 차이: 두 플랫폼 모두 클래스 이름으로 시스템 뷰 트리에서 비공개 뷰를 찾아요. 이 클래스 이름들은 macOS 13.0 바이너리에서 찾을 수 있어요. macOS 26.7에서는 `MKZoomSegmentedControl`이 지도 뷰 트리에 없어서 코드가 조용히 건너뛰어요.

### 디스플레이 링크와 프레임레이트

문제: 여러 애니메이션이 각자 갱신을 요청하면 전력이 낭비돼요. 반면 고주사율 화면에서는 핵심 애니메이션 때 120 Hz로 올려야 해요.

- iOS(UIKit): 싱글턴이 `CADisplayLink`(화면 주사율에 맞춰 콜백을 부르는 타이머)를 하나만 갖고, 모든 요청 중 가장 높은 FPS를 골라요. 구형 시스템에서는 KVC로 `UIScrollView` 내부의 `_scrollHeartbeat`를 얻어 스크롤을 풀 프레임으로 강제해요. 비공개 Lens 애니메이션 동안에는 120 Hz로 고정해요. 관련 항목:
  - [#03-13 공유 DisplayLink 하나로 여러 프레임 레이트 요구를 처리](/docs/ios/03-layout-animation/#t03-13)
  - [#03-05 비공개 `_scrollHeartbeat`로 구형 시스템 스크롤을 고주사율로 강제](/docs/ios/03-layout-animation/#t03-05)
  - [#07-03 비공개 Lens 애니메이션 중 120 Hz 강제하기](/docs/ios/07-visual-effects-glass/#t07-03)
- macOS(AppKit): `CVDisplayLink` 콜백은 실시간 스레드에서 데이터만 쌓고 메인 큐로 넘겨요. 공유 디스플레이 링크 드라이버는 실제 주사 주기에 맞춰 낮은 프레임레이트 요청을 감속해요. 파티클 레이어는 보일 때만 공유 디스플레이 링크를 구독해요. 관련 항목:
  - [#12-11 CVDisplayLink 콜백을 메인 큐로 안전하게 넘기기](/docs/macos/12-tguikit-core/#t12-11)
  - [#12-12 DisplayLink 하나를 공유하고 요청에 따라 프레임 낮추기](/docs/macos/12-tguikit-core/#t12-12)
  - [#15-08 Thanos 파티클은 항목이 있고 보일 때만 전역 디스플레이 링크 구독하기](/docs/macos/15-rendering-app/#t15-08)
- 차이: 두 플랫폼 모두 디스플레이 링크를 공유 드라이버 하나로 모았어요. iOS는 비공개 진입점으로 시스템 스크롤의 프레임레이트를 높여요. macOS는 비공개 API가 필요 없고, 핵심은 `CVDisplayLink`를 스레드 안전하게 메인 큐로 연결하는 거예요.

### 내비게이션과 전환

문제: 기본 내비게이션 전환은 키보드, 상태 표시줄, 내비게이션 바를 동기화하지 못하고 트랙패드 제스처 뒤로 가기도 지원하지 않아요.

- iOS(UIKit): 전체 화면 오른쪽 스와이프 뒤로 가기는 먼저 히트 뷰 체인을 조회하고, 방향 임계값으로 판정해요. 뒤로 가기 진행률이 키보드와 상태 표시줄을 동시에 구동해요. 전환 중에는 내비게이션 바 두 개를 임시로 다시 붙이고 클리핑 컨테이너로 잘라 나눠요. 관련 항목:
  - [#06-06 전체 화면 오른쪽 스와이프 뒤로 가기: 히트 뷰 체인을 먼저 보고 방향 임계값으로 판정하기](/docs/ios/06-navigation-gesture-context/#t06-06)
  - [#06-07 뒤로 가기 진행도를 수동으로 구동하고, 키보드와 상태 표시줄 동기화하기](/docs/ios/06-navigation-gesture-context/#t06-07)
  - [#06-09 전환 중 내비게이션 바 두 개를 임시로 다시 붙이고, 클리핑 컨테이너로 잘라 내기](/docs/ios/06-navigation-gesture-context/#t06-09)
- macOS(AppKit): push 전에 새 페이지의 동적 콘텐츠를 동결해요. 가로 휠 delta가 취소 가능한 인터랙티브 뒤로 가기로 바뀌어요. 모달은 `beginSheet`를 쓰지 않고 `contentView` 위에 덮어씌워요. 관련 항목:
  - [#12-14 내비게이션 push 전에 동적 하위 콘텐츠 동결하기](/docs/macos/12-tguikit-core/#t12-14)
  - [#12-15 스크롤 휠의 가로 delta를 취소 가능한 인터랙티브 내비게이션으로 바꾸기](/docs/macos/12-tguikit-core/#t12-15)
  - [#12-16 contentView 오버레이로 AppKit sheet 대체하기](/docs/macos/12-tguikit-core/#t12-16)
- 차이: 두 플랫폼 모두 전환 진행률을 수동으로 구동해요. iOS의 입력은 터치 제스처예요. macOS의 입력은 트랙패드 휠 delta라서 뒤로 가기 제스처를 가로 스크롤과 구분해야 해요.

## AppKit 고유의 난점 [#appkit]

다음 문제들은 macOS 쪽에만 있고 iOS 쪽에는 대응하는 항목이 없어요.

- **뒤집힌 좌표.** TGUIKit은 `flip = true`로 통일하고 `updateLayer`에서 동기화해서 여러 곳에서 좌표계가 갈라지는 것을 막아요. [#12-05](/docs/macos/12-tguikit-core/#t12-05)를 참고하세요.
- **`NSTableView` 행 지오메트리.** 코드는 자동 행 높이를 끄고 `noteHeightOfRow`를 수동으로 호출하며, 고정 헤더가 밀려나는 계산도 직접 해요. [#13-21](/docs/macos/13-table-scroll-text/#t13-21)과 [#13-06](/docs/macos/13-table-scroll-text/#t13-06)을 참고하세요.
- **비활성 윈도우의 hover.** `NSTrackingArea`가 `.activeInActiveApp`과 `.assumeInside`를 써서 윈도우가 포커스를 잃어도 호버 상태를 계속 갱신해요. [#12-10](/docs/macos/12-tguikit-core/#t12-10)을 참고하세요.
- **윈도우 키보드 디스패치.** 코드가 우선순위 체인과 전역 핸들러를 직접 만들어 기본 responder 체인을 덮어써요. [#12-03](/docs/macos/12-tguikit-core/#t12-03)을 참고하세요.
- **명시적 Core Animation 관리.** `SimpleLayer`의 빈 action이 암시적 애니메이션을 끄고, 핵심 전환에서는 애니메이션을 수동으로 보충해요. [#12-06](/docs/macos/12-tguikit-core/#t12-06)을 참고하세요.
