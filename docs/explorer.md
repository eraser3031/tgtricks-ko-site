# 기법 색인

웹: /docs/explorer/ · Markdown: /docs/explorer.md

> 기법 300개를 프레임워크, 소스 프로젝트, 검증 방식, 장, 태그로 골라 볼 수 있어요.

주제 노트 15편에 담긴 항목을 모두 모았어요. 필터를 고르면 주소창에 그대로 남아서, 걸러 낸 결과를 링크로 공유할 수 있어요. 예를 들어 ‘macOS · AppKit’ 플랫폼과 ‘비공개 API’ 태그를 함께 고르면, macOS에서 공개되지 않은 API에 기대는 항목만 볼 수 있어요. 항목 제목을 누르면 그 장으로 이동해서 코드 발췌와 증거 체인을 볼 수 있어요.

| 번호 | 플랫폼 | 장 | 제목 | 태그 | 바이너리 검증 |
| --- | --- | --- | --- | --- | --- |
| 01-01 | iOS · UIKit | 01 런타임과 비공개 API | [보안 텍스트 입력 레이어를 빌려 아무 레이어나 스크린샷에서 빼기](/docs/ios/01-runtime-private-api/#t01-01) | 비공개 API, Core Animation, 텍스트와 입력 | iOS 바이너리 미검증 |
| 01-02 | iOS · UIKit | 01 런타임과 비공개 API | [XOR 바이트열로 비공개 맞춤법 검사 컨트롤러를 숨기고 정확한 ABI로 호출하기](/docs/ios/01-runtime-private-api/#t01-02) | 비공개 API, 런타임 해킹 | iOS 바이너리 미검증 |
| 01-03 | iOS · UIKit | 01 런타임과 비공개 API | [`Info.plist` 결과를 위조해 App-Bound 도메인을 동적으로 추가·삭제하기](/docs/ios/01-runtime-private-api/#t01-03) | 비공개 API, 런타임 해킹 | iOS 바이너리 미검증 |
| 01-04 | iOS · UIKit | 01 런타임과 비공개 API | [비공개 키보드 씬 델리게이트에서 키보드 윈도우를 얻고 신버전의 함정 피하기](/docs/ios/01-runtime-private-api/#t01-04) | 비공개 API, 텍스트와 입력 | iOS 바이너리 미검증 |
| 01-05 | iOS · UIKit | 01 런타임과 비공개 API | [비공개 `cornerRadii`로 애니메이션 가능한 네 모서리 반경을 얻고 오프스크린 마스크 피하기](/docs/ios/01-runtime-private-api/#t01-05) | 비공개 API | iOS 바이너리 미검증 |
| 01-06 | iOS · UIKit | 01 런타임과 비공개 API | [상속된 메서드를 스위즐링하기 전에 대상 클래스에 먼저 안착시켜 부모 클래스 오염 막기](/docs/ios/01-runtime-private-api/#t01-06) | 비공개 API, 런타임 해킹 | iOS 바이너리 미검증 |
| 01-07 | iOS · UIKit | 01 런타임과 비공개 API | [`UIMenuItem`의 문자 렌더링을 아이콘 렌더링으로 바꿔치기](/docs/ios/01-runtime-private-api/#t01-07) | 비공개 API, 런타임 해킹, 텍스트와 입력 | iOS 바이너리 미검증 |
| 01-08 | iOS · UIKit | 01 런타임과 비공개 API | [비공개 `_scrollHeartbeat`에서 디스플레이 링크를 잡아 구형 시스템 스크롤에 최대 프레임률 강제하기](/docs/ios/01-runtime-private-api/#t01-08) | 비공개 API, 런타임 해킹, 제스처와 스크롤, 성능과 메모리 | iOS 바이너리 미검증 |
| 01-09 | iOS · UIKit | 01 런타임과 비공개 API | [비공개 메서드 시그니처를 먼저 읽고 `float`/`double` IMP 호출 규약 고르기](/docs/ios/01-runtime-private-api/#t01-09) | 비공개 API, 런타임 해킹 | iOS 바이너리 미검증 |
| 01-10 | iOS · UIKit | 01 런타임과 비공개 API | [스프링 애니메이션에 비공개 고프레임률 사유 코드 기록하기](/docs/ios/01-runtime-private-api/#t01-10) | 비공개 API, 성능과 메모리 | iOS 바이너리 미검증 |
| 01-11 | iOS · UIKit | 01 런타임과 비공개 API | [`NSInvocation`으로 비공개 블러 효과를 안전하게 탐지하고 구성하기](/docs/ios/01-runtime-private-api/#t01-11) | 비공개 API | iOS 바이너리 미검증 |
| 01-12 | iOS · UIKit | 01 런타임과 비공개 API | [비공개 `_UIPortalView`를 최소 프로토콜로 감싸고 히트 테스트 전달 끄기](/docs/ios/01-runtime-private-api/#t01-12) | 비공개 API | iOS 바이너리 미검증 |
| 01-13 | iOS · UIKit | 01 런타임과 비공개 API | [비공개 메서드의 전체 타입 인코딩을 검사한 뒤에야 IMP 교체하기](/docs/ios/01-runtime-private-api/#t01-13) | 비공개 API, 런타임 해킹, Core Animation | iOS 바이너리 미검증 |
| 01-14 | iOS · UIKit | 01 런타임과 비공개 API | [비공개 selector로 플로팅 윈도우가 상태 표시줄을 바꾸지도, 키보드 포커스를 빼앗지도 않게 하기](/docs/ios/01-runtime-private-api/#t01-14) | 비공개 API, 텍스트와 입력 | iOS 바이너리 미검증 |
| 01-15 | iOS · UIKit | 01 런타임과 비공개 API | [Debug 레코더가 UIKit 기반 클래스의 IMP를 직접 교체해 원시 스크롤 물리 데이터 수집하기](/docs/ios/01-runtime-private-api/#t01-15) | 비공개 API, 런타임 해킹, 제스처와 스크롤 | iOS 바이너리 미검증 |
| 01-16 | iOS · UIKit | 01 런타임과 비공개 API | [`UITextView` 타일 렌더링을 끄고 iOS 16 비공개 자동 수정 프롬프트 식별하기](/docs/ios/01-runtime-private-api/#t01-16) | 비공개 API, 텍스트와 입력, 제스처와 스크롤, 성능과 메모리 | iOS 바이너리 미검증 |
| 01-17 | iOS · UIKit | 01 런타임과 비공개 API | [구형 시스템은 KVC로 기기 방향을 강제하고, 신형 시스템은 씬 지오메트리 갱신 쓰기](/docs/ios/01-runtime-private-api/#t01-17) | 비공개 API | iOS 바이너리 미검증 |
| 01-18 | iOS · UIKit | 01 런타임과 비공개 API | [비공개 `UIWindow` 방향 콜백을 재정의해 레이아웃 중 회전 상태를 정확히 감싸기](/docs/ios/01-runtime-private-api/#t01-18) | 비공개 API, 런타임 해킹 | iOS 바이너리 미검증 |
| 01-19 | iOS · UIKit | 01 런타임과 비공개 API | [런타임 클래스 테이블을 해시로 색인해 비공개 클래스명이 바이너리에 들어가지 않게 하기](/docs/ios/01-runtime-private-api/#t01-19) | 비공개 API, 런타임 해킹 | iOS 바이너리 미검증 |
| 01-20 | iOS · UIKit | 01 런타임과 비공개 API | [장식 하위 클래스를 자동 생성하고 `alloc` 시점에 인스턴스마다 ISA 교체하기](/docs/ios/01-runtime-private-api/#t01-20) | 비공개 API, 런타임 해킹 | iOS 바이너리 미검증 |
| 01-21 | iOS · UIKit | 01 런타임과 비공개 API | [시스템 뒤로 가기 제스처 인식기의 ISA를 직접 고쳐 전체 화면 내비게이션 제스처 추가하기](/docs/ios/01-runtime-private-api/#t01-21) | 비공개 API, 런타임 해킹, 제스처와 스크롤 | iOS 바이너리 미검증 |
| 01-22 | iOS · UIKit | 01 런타임과 비공개 API | [`NSNotificationCenter`의 동기 전달을 가로챌 수 있는 미들웨어로 개조하기](/docs/ios/01-runtime-private-api/#t01-22) | 비공개 API, 런타임 해킹, 성능과 메모리 | iOS 바이너리 미검증 |
| 01-23 | iOS · UIKit | 01 런타임과 비공개 API | [스크린샷 중복 알림의 중복 제거 상태를 observer마다 두어 오버레이의 누락 보고 피하기](/docs/ios/01-runtime-private-api/#t01-23) | 비공개 API | iOS 바이너리 미검증 |
| 01-24 | iOS · UIKit | 01 런타임과 비공개 API | [`CAFilter`를 런타임 팩토리로만 쓰고 비공개 헤더 파일은 들이지 않기](/docs/ios/01-runtime-private-api/#t01-24) | 비공개 API, 런타임 해킹, Core Animation | iOS 바이너리 미검증 |
| 02-01 | iOS · UIKit | 02 윈도우, 키보드, 상태 표시줄 | [비공개 키보드 scene delegate로 키보드 윈도우 얻기](/docs/ios/02-window-keyboard-statusbar/#t02-01) | 비공개 API, 텍스트와 입력 | iOS 바이너리 미검증 |
| 02-02 | iOS · UIKit | 02 윈도우, 키보드, 상태 표시줄 | [클래스 이름 패턴으로 여러 세대의 시스템 키보드 뷰 맞추기](/docs/ios/02-window-keyboard-statusbar/#t02-02) | 비공개 API, 텍스트와 입력 | iOS 바이너리 미검증 |
| 02-03 | iOS · UIKit | 02 윈도우, 키보드, 상태 표시줄 | [앱 콘텐츠 대신 시스템 키보드 레이어를 직접 옮기기](/docs/ios/02-window-keyboard-statusbar/#t02-03) | 비공개 API, Core Animation, 텍스트와 입력 | iOS 바이너리 미검증 |
| 02-04 | iOS · UIKit | 02 윈도우, 키보드, 상태 표시줄 | [여러 좌표로 키보드 알림의 잘못된 높이 보정하기](/docs/ios/02-window-keyboard-statusbar/#t02-04) | 텍스트와 입력 | iOS 바이너리 미검증 |
| 02-05 | iOS · UIKit | 02 윈도우, 키보드, 상태 표시줄 | [first responder와 액세서리 높이로 정하는 전체 윈도우 제스처의 드래그 영역](/docs/ios/02-window-keyboard-statusbar/#t02-05) | 텍스트와 입력, 제스처와 스크롤 | iOS 바이너리 미검증 |
| 02-06 | iOS · UIKit | 02 윈도우, 키보드, 상태 표시줄 | [키보드 내림 제스처로 같은 손가락의 리스트 관성 명시적으로 취소하기](/docs/ios/02-window-keyboard-statusbar/#t02-06) | 텍스트와 입력, 제스처와 스크롤 | iOS 바이너리 미검증 |
| 02-07 | iOS · UIKit | 02 윈도우, 키보드, 상태 표시줄 | [선택 영역 변화를 위조해 텍스트 자동 수정 일으키기](/docs/ios/02-window-keyboard-statusbar/#t02-07) | 텍스트와 입력 | iOS 바이너리 미검증 |
| 02-08 | iOS · UIKit | 02 윈도우, 키보드, 상태 표시줄 | [시스템 키보드 윈도우를 first responder가 있는 표면으로 가로 평행 이동하기](/docs/ios/02-window-keyboard-statusbar/#t02-08) | 비공개 API, Core Animation, 텍스트와 입력 | iOS 바이너리 미검증 |
| 02-09 | iOS · UIKit | 02 윈도우, 키보드, 상태 표시줄 | [시스템 상태 표시줄을 캡처해 픽셀 단위로 다시 칠하기](/docs/ios/02-window-keyboard-statusbar/#t02-09) | 비공개 API | iOS 바이너리 미검증 |
| 02-10 | iOS · UIKit | 02 윈도우, 키보드, 상태 표시줄 | [런타임에 비공개 상태 표시줄 클래스 이름 이어 붙이기](/docs/ios/02-window-keyboard-statusbar/#t02-10) | 비공개 API, 런타임 해킹, 텍스트와 입력 | iOS 바이너리 미검증 |
| 02-11 | iOS · UIKit | 02 윈도우, 키보드, 상태 표시줄 | [공용 RunLoop로 상태 표시줄의 활성 요소 주기적으로 다시 캡처하기](/docs/ios/02-window-keyboard-statusbar/#t02-11) | 비공개 API | iOS 바이너리 미검증 |
| 02-12 | iOS · UIKit | 02 윈도우, 키보드, 상태 표시줄 | [여러 노드 트리의 상태 표시줄 결정을 하나의 루트 컨트롤러로 모으기](/docs/ios/02-window-keyboard-statusbar/#t02-12) |  | iOS 바이너리 미검증 |
| 02-13 | iOS · UIKit | 02 윈도우, 키보드, 상태 표시줄 | [키보드가 보이면 키보드 윈도우로 옮겨 붙는 전역 오버레이](/docs/ios/02-window-keyboard-statusbar/#t02-13) | 비공개 API, 텍스트와 입력 | iOS 바이너리 미검증 |
| 02-14 | iOS · UIKit | 02 윈도우, 키보드, 상태 표시줄 | [UIKit containment 대신 수동 생명주기를 쓰는 표시 컨텍스트](/docs/ios/02-window-keyboard-statusbar/#t02-14) |  | iOS 바이너리 미검증 |
| 02-15 | iOS · UIKit | 02 윈도우, 키보드, 상태 표시줄 | [시스템 편집 메뉴를 따로 처리하는 커스텀 `UIWindow.hitTest`](/docs/ios/02-window-keyboard-statusbar/#t02-15) |  | iOS 바이너리 미검증 |
| 02-16 | iOS · UIKit | 02 윈도우, 키보드, 상태 표시줄 | [상태 표시줄의 “맨 위로” 제스처를 가로채는 투명한 가짜 스크롤 뷰](/docs/ios/02-window-keyboard-statusbar/#t02-16) | 제스처와 스크롤 | iOS 바이너리 미검증 |
| 02-17 | iOS · UIKit | 02 윈도우, 키보드, 상태 표시줄 | [전체 화면 윈도우에만 기기 표로 측정한 세이프 에어리어 적용하기](/docs/ios/02-window-keyboard-statusbar/#t02-17) |  | iOS 바이너리 미검증 |
| 02-18 | iOS · UIKit | 02 윈도우, 키보드, 상태 표시줄 | [모델, 스케일 비율, 화면 크기로 경험적 기기 프로필 고르기](/docs/ios/02-window-keyboard-statusbar/#t02-18) |  | iOS 바이너리 미검증 |
| 02-19 | iOS · UIKit | 02 윈도우, 키보드, 상태 표시줄 | [다이내믹 아일랜드와 노치를 기종별로 잰 지오메트리 표로 그리기](/docs/ios/02-window-keyboard-statusbar/#t02-19) |  | iOS 바이너리 미검증 |
| 02-20 | iOS · UIKit | 02 윈도우, 키보드, 상태 표시줄 | [기기 지표 표 전체를 건너뛰는 듀얼 디스플레이 모델](/docs/ios/02-window-keyboard-statusbar/#t02-20) | 성능과 메모리 | iOS 바이너리 미검증 |
| 02-21 | iOS · UIKit | 02 윈도우, 키보드, 상태 표시줄 | [최신 Scene API와 구형 KVC를 모두 맞추는 방향 잠금](/docs/ios/02-window-keyboard-statusbar/#t02-21) | 비공개 API | iOS 바이너리 미검증 |
| 02-22 | iOS · UIKit | 02 윈도우, 키보드, 상태 표시줄 | [UIKit 상속 애니메이션 중 윈도우 레이어 프레임 애니메이션을 수동으로 하나 더하기](/docs/ios/02-window-keyboard-statusbar/#t02-22) |  | iOS 바이너리 미검증 |
| 03-01 | iOS · UIKit | 03 레이아웃과 애니메이션 | [비공개 `cornerRadii`로 마스크 없이 네 모서리를 각각 둥글게](/docs/ios/03-layout-animation/#t03-01) | 비공개 API | iOS 바이너리 미검증 |
| 03-02 | iOS · UIKit | 03 레이아웃과 애니메이션 | [UIKit 트랜잭션 안의 스프링 애니메이션 탈취](/docs/ios/03-layout-animation/#t03-02) | 비공개 API, 런타임 해킹, Core Animation, 성능과 메모리 | iOS 바이너리 미검증 |
| 03-03 | iOS · UIKit | 03 레이아웃과 애니메이션 | [비공개 `_solveForInput:`을 호출해 수동 보간과 시스템 스프링을 통일](/docs/ios/03-layout-animation/#t03-03) | 비공개 API, 런타임 해킹 | iOS 바이너리 미검증 |
| 03-04 | iOS · UIKit | 03 레이아웃과 애니메이션 | [공개되지 않은 UIKit 곡선 값 `7 << 16` 차용](/docs/ios/03-layout-animation/#t03-04) | 비공개 API, 텍스트와 입력 | iOS 바이너리 미검증 |
| 03-05 | iOS · UIKit | 03 레이아웃과 애니메이션 | [비공개 `_scrollHeartbeat`로 구형 시스템 스크롤을 고주사율로 강제](/docs/ios/03-layout-animation/#t03-05) | 비공개 API, 제스처와 스크롤 | iOS 바이너리 미검증 |
| 03-06 | iOS · UIKit | 03 레이아웃과 애니메이션 | [iOS 26 스프링에 비공개 고주사율 사유 코드 기록](/docs/ios/03-layout-animation/#t03-06) | 비공개 API | iOS 바이너리 미검증 |
| 03-07 | iOS · UIKit | 03 레이아웃과 애니메이션 | [저장된 기하량으로 `CALayer.frame`의 부동소수점 재시작 애니메이션 피하기](/docs/ios/03-layout-animation/#t03-07) | Core Animation | iOS 바이너리 미검증 |
| 03-08 | iOS · UIKit | 03 레이아웃과 애니메이션 | [프레젠테이션 레이어에서 중단된 레이아웃 애니메이션 이어받기](/docs/ios/03-layout-animation/#t03-08) |  | iOS 바이너리 미검증 |
| 03-09 | iOS · UIKit | 03 레이아웃과 애니메이션 | [frame을 두 개의 애니메이션으로 쪼개고 완료 신호를 합류시키기](/docs/ios/03-layout-animation/#t03-09) | 비공개 API | iOS 바이너리 미검증 |
| 03-10 | iOS · UIKit | 03 레이아웃과 애니메이션 | [additive 변위로 이미 갱신된 모델 frame 따라잡기](/docs/ios/03-layout-animation/#t03-10) | Core Animation | iOS 바이너리 미검증 |
| 03-11 | iOS · UIKit | 03 레이아웃과 애니메이션 | [수평·수직 기하량을 서로 다른 곡선으로 전환](/docs/ios/03-layout-animation/#t03-11) | 비공개 API | iOS 바이너리 미검증 |
| 03-12 | iOS · UIKit | 03 레이아웃과 애니메이션 | [`action(forKey:)`에서 암묵적 애니메이션 끄기와 마운트 추적을 동시에](/docs/ios/03-layout-animation/#t03-12) | Core Animation | iOS 바이너리 미검증 |
| 03-13 | iOS · UIKit | 03 레이아웃과 애니메이션 | [공유 DisplayLink 하나로 여러 프레임 레이트 요구를 처리](/docs/ios/03-layout-animation/#t03-13) | 비공개 API, 텍스트와 입력, 성능과 메모리 | iOS 바이너리 미검증 |
| 03-14 | iOS · UIKit | 03 레이아웃과 애니메이션 | [안정적인 정체성으로 UIKit 서브뷰를 재생성하지 않고 이동](/docs/ios/03-layout-animation/#t03-14) |  | iOS 바이너리 미검증 |
| 03-15 | iOS · UIKit | 03 레이아웃과 애니메이션 | [제거 중인 서브뷰가 움직이는 guide를 계속 추적하게 하기](/docs/ios/03-layout-animation/#t03-15) |  | iOS 바이너리 미검증 |
| 03-16 | iOS · UIKit | 03 레이아웃과 애니메이션 | [동등성, 환경, 크기로 렌더링 건너뛰기 조건 구성](/docs/ios/03-layout-animation/#t03-16) | 성능과 메모리 | iOS 바이너리 미검증 |
| 03-17 | iOS · UIKit | 03 레이아웃과 애니메이션 | [컴포넌트 컨텍스트를 UIView에 붙여 타입이 있는 상태 보존](/docs/ios/03-layout-animation/#t03-17) |  | iOS 바이너리 미검증 |
| 03-18 | iOS · UIKit | 03 레이아웃과 애니메이션 | [아무 연속 변환이나 기기 프레임 레이트로 샘플링한 키프레임으로 이산화](/docs/ios/03-layout-animation/#t03-18) | 성능과 메모리 | iOS 바이너리 미검증 |
| 03-19 | iOS · UIKit | 03 레이아웃과 애니메이션 | [모서리 비트맵을 미리 렌더링해 원자적으로 캐시하고 blit으로 합성](/docs/ios/03-layout-animation/#t03-19) | 비공개 API, 성능과 메모리 | iOS 바이너리 미검증 |
| 03-20 | iOS · UIKit | 03 레이아웃과 애니메이션 | [무작위 위상의 additive 미세 변위로 재정렬 항목의 동일 주기 흔들림 방지](/docs/ios/03-layout-animation/#t03-20) | 비공개 API, 텍스트와 입력 | iOS 바이너리 미검증 |
| 03-21 | iOS · UIKit | 03 레이아웃과 애니메이션 | [레이아웃 없는 `CATransformLayer`로 AsyncDisplayKit 적응](/docs/ios/03-layout-animation/#t03-21) | 비공개 API, Core Animation | iOS 바이너리 미검증 |
| 03-22 | iOS · UIKit | 03 레이아웃과 애니메이션 | [시뮬레이터 슬로 애니메이션을 UIKit과 명시적 CA 애니메이션에 함께 적용하기](/docs/ios/03-layout-animation/#t03-22) | 런타임 해킹 | iOS 바이너리 미검증 |
| 04-01 | iOS · UIKit | 04 리스트와 스크롤 | [보이지 않는 `UIScrollView`로 스크롤 물리만 가져오기](/docs/ios/04-listview-scroll/#t04-01) | 제스처와 스크롤 | iOS 바이너리 미검증 |
| 04-02 | iOS · UIKit | 04 리스트와 스크롤 | [20,000pt짜리 가짜 콘텐츠 높이로 아직 모르는 절반의 데이터 감당하기](/docs/ios/04-listview-scroll/#t04-02) | 제스처와 스크롤 | iOS 바이너리 미검증 |
| 04-03 | iOS · UIKit | 04 리스트와 스크롤 | [부모와 자식을 π만큼 두 번 회전해 뒤집힌 채팅의 글자 바로 세우기](/docs/ios/04-listview-scroll/#t04-03) | 제스처와 스크롤 | iOS 바이너리 미검증 |
| 04-04 | iOS · UIKit | 04 리스트와 스크롤 | [트랜잭션 큐로 비동기 레이아웃의 완료 순서 맞추기](/docs/ios/04-listview-scroll/#t04-04) | 성능과 메모리 | iOS 바이너리 미검증 |
| 04-05 | iOS · UIKit | 04 리스트와 스크롤 | [레이아웃은 비동기로, 노드 쓰기는 메인 큐에서 하기](/docs/ios/04-listview-scroll/#t04-05) | 성능과 메모리 | iOS 바이너리 미검증 |
| 04-06 | iOS · UIKit | 04 리스트와 스크롤 | [빠르게 스크롤할 때 가시 윈도우 갱신 트랜잭션을 하나만 걸기](/docs/ios/04-listview-scroll/#t04-06) | 제스처와 스크롤 | iOS 바이너리 미검증 |
| 04-07 | iOS · UIKit | 04 리스트와 스크롤 | [정지된 항목으로 삽입·삭제 뒤 뷰포트 고정하기](/docs/ios/04-listview-scroll/#t04-07) |  | iOS 바이너리 미검증 |
| 04-08 | iOS · UIKit | 04 리스트와 스크롤 | [자리 표시 노드로 삭제 중인 행의 공간 지키기](/docs/ios/04-listview-scroll/#t04-08) |  | iOS 바이너리 미검증 |
| 04-09 | iOS · UIKit | 04 리스트와 스크롤 | [`apparentHeight`로 실제 레이아웃 높이와 애니메이션 높이 나누기](/docs/ios/04-listview-scroll/#t04-09) |  | iOS 바이너리 미검증 |
| 04-10 | iOS · UIKit | 04 리스트와 스크롤 | [키보드와 inset 애니메이션을 `sublayerTransform` 하나로 바꾸기](/docs/ios/04-listview-scroll/#t04-10) | Core Animation, 텍스트와 입력 | iOS 바이너리 미검증 |
| 04-11 | iOS · UIKit | 04 리스트와 스크롤 | [뷰포트 바깥 500pt를 소프트 프리로드 윈도우로 쓰기](/docs/ios/04-listview-scroll/#t04-11) | 성능과 메모리 | iOS 바이너리 미검증 |
| 04-12 | iOS · UIKit | 04 리스트와 스크롤 | [플로팅 header로 상단·가장자리·하단 고정 한꺼번에 처리하기](/docs/ios/04-listview-scroll/#t04-12) |  | iOS 바이너리 미검증 |
| 04-13 | iOS · UIKit | 04 리스트와 스크롤 | [전체 행 높이를 몰라도 근사 스크롤바 그리기](/docs/ios/04-listview-scroll/#t04-13) | 제스처와 스크롤 | iOS 바이너리 미검증 |
| 04-14 | iOS · UIKit | 04 리스트와 스크롤 | [외부 속도를 UIKit처럼 감속하는 구간으로 주입하기](/docs/ios/04-listview-scroll/#t04-14) | 제스처와 스크롤 | iOS 바이너리 미검증 |
| 04-15 | iOS · UIKit | 04 리스트와 스크롤 | [`GridNode`에서 `UIScrollView`의 layer를 그리기 무효화에 반응하지 않게 바꾸기](/docs/ios/04-listview-scroll/#t04-15) | Core Animation, 제스처와 스크롤, 성능과 메모리 | iOS 바이너리 미검증 |
| 04-16 | iOS · UIKit | 04 리스트와 스크롤 | [`GridNode`에서 preload 윈도우를 먼저 잘라낸 뒤 노드 만들기](/docs/ios/04-listview-scroll/#t04-16) |  | iOS 바이너리 미검증 |
| 04-17 | iOS · UIKit | 04 리스트와 스크롤 | [그리드 애니메이션에서 화면 좌표를 먼저 고정하고 additive 이동하기](/docs/ios/04-listview-scroll/#t04-17) |  | iOS 바이너리 미검증 |
| 04-18 | iOS · UIKit | 04 리스트와 스크롤 | [선형 분할로 “앨범식” 균형 행 만들기](/docs/ios/04-listview-scroll/#t04-18) |  | iOS 바이너리 미검증 |
| 04-19 | iOS · UIKit | 04 리스트와 스크롤 | [희소 그리드에서 로드되지 않은 인덱스를 공백(hole)으로 보고 방향 정해 가져오기](/docs/ios/04-listview-scroll/#t04-19) |  | iOS 바이너리 미검증 |
| 04-20 | iOS · UIKit | 04 리스트와 스크롤 | [희소 그리드에서 cell 풀 대신 안정적인 id로 보이는 layer 관리하기](/docs/ios/04-listview-scroll/#t04-20) | Core Animation, 제스처와 스크롤, 성능과 메모리 | iOS 바이너리 미검증 |
| 04-21 | iOS · UIKit | 04 리스트와 스크롤 | [긴 그라데이션 이미지 한 장과 `contentsRect`로 전역 shimmer 구동하기](/docs/ios/04-listview-scroll/#t04-21) | Core Animation, 성능과 메모리 | iOS 바이너리 미검증 |
| 04-22 | iOS · UIKit | 04 리스트와 스크롤 | [`AsyncListComponent`에서 세로 `ListView`를 90° 회전해 재사용하기](/docs/ios/04-listview-scroll/#t04-22) | 제스처와 스크롤 | iOS 바이너리 미검증 |
| 04-23 | iOS · UIKit | 04 리스트와 스크롤 | [CoreList에서 IMP swizzle로 UIKit 물리의 기준값 녹화하기](/docs/ios/04-listview-scroll/#t04-23) | 비공개 API, 런타임 해킹, 제스처와 스크롤 | iOS 바이너리 미검증 |
| 05-01 | iOS · UIKit | 05 ASDK와 텍스트 | [시스템 선택 인터랙션을 설치하지 않고 선택 UI를 전부 직접 그리기](/docs/ios/05-asdk-text/#t05-01) | 비공개 API, 텍스트와 입력, 제스처와 스크롤 | iOS 바이너리 미검증 |
| 05-02 | iOS · UIKit | 05 ASDK와 텍스트 | [키보드에 “가상 줄바꿈”을 투영해 한글과 CJK의 문단 간 재조합 수정하기](/docs/ios/05-asdk-text/#t05-02) | 텍스트와 입력 | iOS 바이너리 미검증 |
| 05-03 | iOS · UIKit | 05 ASDK와 텍스트 | [IME 조합 텍스트와 인라인 예측을 “확정”과 “폐기”의 두 경로로 나누기](/docs/ios/05-asdk-text/#t05-03) | 텍스트와 입력, 제스처와 스크롤 | iOS 바이너리 미검증 |
| 05-04 | iOS · UIKit | 05 ASDK와 텍스트 | [보이지 않는 1×1 `NSTextAttachment`로 공간을 확보한 뒤 실제 애니메이션 이모지 뷰를 덮어 씌우기](/docs/ios/05-asdk-text/#t05-04) | 텍스트와 입력 | iOS 바이너리 미검증 |
| 05-05 | iOS · UIKit | 05 ASDK와 텍스트 | [구형 텍스트 노드는 `CTRunDelegate`로 커스텀 emoji를 하나의 조판 글리프로 위장하기](/docs/ios/05-asdk-text/#t05-05) | Core Animation, 텍스트와 입력 | iOS 바이너리 미검증 |
| 05-06 | iOS · UIKit | 05 ASDK와 텍스트 | [비동기 그리기는 원자적 sentinel로 오래된 프레임을 취소하고 메인 스레드에서 원자적으로 결과를 반영하기](/docs/ios/05-asdk-text/#t05-06) | Core Animation, 성능과 메모리 | iOS 바이너리 미검증 |
| 05-07 | iOS · UIKit | 05 ASDK와 텍스트 | [“기반 레이어 지우기 + 클리핑 재그리기 레이어”로 spoiler 텍스트를 이중 레이어로 렌더링하기](/docs/ios/05-asdk-text/#t05-07) | Core Animation, 텍스트와 입력, 성능과 메모리 | iOS 바이너리 미검증 |
| 05-08 | iOS · UIKit | 05 ASDK와 텍스트 | [TextKit 2는 RTL 커서 정지를 바로잡기 위해 일부러 `.selection` 기하를 사용하기](/docs/ios/05-asdk-text/#t05-08) | 텍스트와 입력, 제스처와 스크롤 | iOS 바이너리 미검증 |
| 05-09 | iOS · UIKit | 05 ASDK와 텍스트 | [`UITextView` 스크롤을 끄면서도 스크롤이 켜져 있다고 믿게 하기](/docs/ios/05-asdk-text/#t05-09) | 텍스트와 입력, 제스처와 스크롤 | iOS 바이너리 미검증 |
| 05-10 | iOS · UIKit | 05 ASDK와 텍스트 | [커스텀 잘림 token은 `CTLineCreateTruncatedLine`을 두 번 호출해 token이 잘려 나가지 않게 하기](/docs/ios/05-asdk-text/#t05-10) | 텍스트와 입력 | iOS 바이너리 미검증 |
| 05-11 | iOS · UIKit | 05 ASDK와 텍스트 | [CoreText 행 단위 “컷아웃” 조판을 직접 작성해 아바타나 말풍선 모서리 자리 확보하기](/docs/ios/05-asdk-text/#t05-11) | 텍스트와 입력 | iOS 바이너리 미검증 |
| 05-12 | iOS · UIKit | 05 ASDK와 텍스트 | [layer-backed 노드는 약한 프록시로 `CALayer.delegate` 댕글링 포인터 수정하기](/docs/ios/05-asdk-text/#t05-12) | Core Animation | iOS 바이너리 미검증 |
| 05-13 | iOS · UIKit | 05 ASDK와 텍스트 | [문자열 분할로 비공개 자동 수정 프롬프트 클래스를 참조하고 비공개 교체 메뉴 항목 비활성화하기](/docs/ios/05-asdk-text/#t05-13) | 비공개 API, 런타임 해킹, 텍스트와 입력 | iOS 바이너리 미검증 |
| 05-14 | iOS · UIKit | 05 ASDK와 텍스트 | [링크 하이라이트 윤곽을 mask로 재사용해 글자 모양 안에서만 흐르는 shimmer 구현하기](/docs/ios/05-asdk-text/#t05-14) | Metal / GPU, 텍스트와 입력 | iOS 바이너리 미검증 |
| 05-15 | iOS · UIKit | 05 ASDK와 텍스트 | [RichText spoiler는 표시 속성으로 글리프를 숨기고 안정적인 식별자로 파티클 레이어를 풀링하기](/docs/ios/05-asdk-text/#t05-15) | 텍스트와 입력 | iOS 바이너리 미검증 |
| 05-16 | iOS · UIKit | 05 ASDK와 텍스트 | [Markdown 붙여넣기를 두 개의 run-loop 실행 취소 그룹으로 나누고 중간 깜빡임 억제하기](/docs/ios/05-asdk-text/#t05-16) | 텍스트와 입력, 성능과 메모리 | iOS 바이너리 미검증 |
| 05-17 | iOS · UIKit | 05 ASDK와 텍스트 | [링크 하이라이트가 인접한 행의 사각형을 이어 붙여 하나의 연속된 둥근 윤곽으로 만들기](/docs/ios/05-asdk-text/#t05-17) |  | iOS 바이너리 미검증 |
| 05-18 | iOS · UIKit | 05 ASDK와 텍스트 | [텍스트 레이아웃은 영향을 주는 모든 조건을 비교하고, 일치하면 이전 `TextNodeLayout`을 그대로 재사용하기](/docs/ios/05-asdk-text/#t05-18) | 텍스트와 입력 | iOS 바이너리 미검증 |
| 05-19 | iOS · UIKit | 05 ASDK와 텍스트 | [tint 가능한 글자와 tint 불가능한 컬러 emoji를 겹쳐진 두 텍스트 노드로 분리하기](/docs/ios/05-asdk-text/#t05-19) | Core Animation, 텍스트와 입력, 성능과 메모리 | iOS 바이너리 미검증 |
| 05-20 | iOS · UIKit | 05 ASDK와 텍스트 | [AsyncDisplayKit의 TextKit 포크는 커스텀 컨테이너로 인용문 홈을 유지하고 커스텀 emoji 글리프를 건너뛰기](/docs/ios/05-asdk-text/#t05-20) | 텍스트와 입력 | iOS 바이너리 미검증 |
| 06-01 | iOS · UIKit | 06 내비게이션, 제스처와 컨텍스트 메뉴 | [비공개 `_UIPortalView`로 같은 콘텐츠를 레이어 간에 표시하기](/docs/ios/06-navigation-gesture-context/#t06-01) | 비공개 API | iOS 바이너리 미검증 |
| 06-02 | iOS · UIKit | 06 내비게이션, 제스처와 컨텍스트 메뉴 | [Portal로 원본 콘텐츠를 보관하고, 애니메이션 끝에서 명시적으로 돌려주기](/docs/ios/06-navigation-gesture-context/#t06-02) | 비공개 API | iOS 바이너리 미검증 |
| 06-03 | iOS · UIKit | 06 내비게이션, 제스처와 컨텍스트 메뉴 | [투명 배경에 1×1 이미지를 넣어 “보이지만 눌리지 않는” 메뉴 바깥 탭 고치기](/docs/ios/06-navigation-gesture-context/#t06-03) | 비공개 API, Core Animation | iOS 바이너리 미검증 |
| 06-04 | iOS · UIKit | 06 내비게이션, 제스처와 컨텍스트 메뉴 | [스케일이 없는 오버레이로 추출하기 전에, 조상의 스케일 보정하기](/docs/ios/06-navigation-gesture-context/#t06-04) |  | iOS 바이너리 미검증 |
| 06-05 | iOS · UIKit | 06 내비게이션, 제스처와 컨텍스트 메뉴 | [퇴장 중에 presentation layer로 착지점 다시 지정하기](/docs/ios/06-navigation-gesture-context/#t06-05) | Core Animation | iOS 바이너리 미검증 |
| 06-06 | iOS · UIKit | 06 내비게이션, 제스처와 컨텍스트 메뉴 | [전체 화면 오른쪽 스와이프 뒤로 가기: 히트 뷰 체인을 먼저 보고 방향 임계값으로 판정하기](/docs/ios/06-navigation-gesture-context/#t06-06) | 제스처와 스크롤 | iOS 바이너리 미검증 |
| 06-07 | iOS · UIKit | 06 내비게이션, 제스처와 컨텍스트 메뉴 | [뒤로 가기 진행도를 수동으로 구동하고, 키보드와 상태 표시줄 동기화하기](/docs/ios/06-navigation-gesture-context/#t06-07) | 텍스트와 입력 | iOS 바이너리 미검증 |
| 06-08 | iOS · UIKit | 06 내비게이션, 제스처와 컨텍스트 메뉴 | [픽셀에 정렬된 수동 기하로 내비게이션 시차와 그림자 구현하기](/docs/ios/06-navigation-gesture-context/#t06-08) | Core Animation, 성능과 메모리 | iOS 바이너리 미검증 |
| 06-09 | iOS · UIKit | 06 내비게이션, 제스처와 컨텍스트 메뉴 | [전환 중 내비게이션 바 두 개를 임시로 다시 붙이고, 클리핑 컨테이너로 잘라 내기](/docs/ios/06-navigation-gesture-context/#t06-09) | 성능과 메모리 | iOS 바이너리 미검증 |
| 06-10 | iOS · UIKit | 06 내비게이션, 제스처와 컨텍스트 메뉴 | [컨텍스트 길게 누르기: 시스템 왼쪽 가장자리를 먼저 피하고 제스처 트리 전체를 능동적으로 취소하기](/docs/ios/06-navigation-gesture-context/#t06-10) | 제스처와 스크롤 | iOS 바이너리 미검증 |
| 06-11 | iOS · UIKit | 06 내비게이션, 제스처와 컨텍스트 메뉴 | [인식기 하나로 탭, 더블 탭, 길게 누르기, 우클릭, 누르고 있기를 모두 처리하기](/docs/ios/06-navigation-gesture-context/#t06-11) | 제스처와 스크롤 | iOS 바이너리 미검증 |
| 06-12 | iOS · UIKit | 06 내비게이션, 제스처와 컨텍스트 메뉴 | [Peek를 원본 사각형에서 튀어나오게 하고, 활성화 모드에 따라 블러 전략 바꾸기](/docs/ios/06-navigation-gesture-context/#t06-12) | 제스처와 스크롤 | iOS 바이너리 미검증 |
| 06-13 | iOS · UIKit | 06 내비게이션, 제스처와 컨텍스트 메뉴 | [가장 안쪽 스크롤 뷰가 맨 위에 있을 때만 아래로 밀어 닫기 허용하기](/docs/ios/06-navigation-gesture-context/#t06-13) | 제스처와 스크롤 | iOS 바이너리 미검증 |
| 06-14 | iOS · UIKit | 06 내비게이션, 제스처와 컨텍스트 메뉴 | [최소화한 뒤 지연을 두고 살아 있는 컨트롤러를 스냅샷으로 교체하기](/docs/ios/06-navigation-gesture-context/#t06-14) |  | iOS 바이너리 미검증 |
| 06-15 | iOS · UIKit | 06 내비게이션, 제스처와 컨텍스트 메뉴 | [비공개 `CAFilter`로 스냅샷에 배경 블러 보충하기](/docs/ios/06-navigation-gesture-context/#t06-15) | 비공개 API, Core Animation | iOS 바이너리 미검증 |
| 06-16 | iOS · UIKit | 06 내비게이션, 제스처와 컨텍스트 메뉴 | [3D 변환으로 최소화된 페이지를 스크롤 가능한 카드 더미로 쌓기](/docs/ios/06-navigation-gesture-context/#t06-16) | Metal / GPU, 제스처와 스크롤 | iOS 바이너리 미검증 |
| 06-17 | iOS · UIKit | 06 내비게이션, 제스처와 컨텍스트 메뉴 | [iPad 분할 화면에서 실제 화면 가장자리의 안전 영역만 해당 창에 주기](/docs/ios/06-navigation-gesture-context/#t06-17) | 제스처와 스크롤 | iOS 바이너리 미검증 |
| 06-18 | iOS · UIKit | 06 내비게이션, 제스처와 컨텍스트 메뉴 | [햅틱 생성기를 스타일별로 캐시하고, 제스처가 시작될 때 예열하기](/docs/ios/06-navigation-gesture-context/#t06-18) | 제스처와 스크롤, 성능과 메모리 | iOS 바이너리 미검증 |
| 06-19 | iOS · UIKit | 06 내비게이션, 제스처와 컨텍스트 메뉴 | [갤러리 퇴장 때 원본 스냅샷이 이어받고, 애니메이션 세 개가 끝나기를 기다리기](/docs/ios/06-navigation-gesture-context/#t06-19) |  | iOS 바이너리 미검증 |
| 06-20 | iOS · UIKit | 06 내비게이션, 제스처와 컨텍스트 메뉴 | [반응 패널을 상단 오버스크롤로 확장하고, 스크롤 pan을 능동적으로 취소하기](/docs/ios/06-navigation-gesture-context/#t06-20) | 제스처와 스크롤 | iOS 바이너리 미검증 |
| 07-01 | iOS · UIKit | 07 시각 효과와 글래스 | [iOS 26의 비공개 Liquid Lens를 직접 구동하기](/docs/ios/07-visual-effects-glass/#t07-01) | 비공개 API, 런타임 해킹 | iOS 바이너리 미검증 |
| 07-02 | iOS · UIKit | 07 시각 효과와 글래스 | [비공개 Lens에 바운스 갱신과 위치 애니메이션 보완하기](/docs/ios/07-visual-effects-glass/#t07-02) | 비공개 API | iOS 바이너리 미검증 |
| 07-03 | iOS · UIKit | 07 시각 효과와 글래스 | [비공개 Lens 애니메이션 중 120 Hz 강제하기](/docs/ios/07-visual-effects-glass/#t07-03) | 비공개 API | iOS 바이너리 미검증 |
| 07-04 | iOS · UIKit | 07 시각 효과와 글래스 | [비공개 Lens가 없을 때 휘도→Alpha 변환으로 도려낸 글래스 흉내 내기](/docs/ios/07-visual-effects-glass/#t07-04) | 비공개 API, Core Animation | iOS 바이너리 미검증 |
| 07-05 | iOS · UIKit | 07 시각 효과와 글래스 | [스크린샷 대신 `_UIPortalView`로 원격 뷰 복제하기](/docs/ios/07-visual-effects-glass/#t07-05) | 비공개 API, 런타임 해킹, 텍스트와 입력 | iOS 바이너리 미검증 |
| 07-06 | iOS · UIKit | 07 시각 효과와 글래스 | [Portal 재바인딩 후 원래 위치에 다시 삽입해 렌더링 갱신 강제하기](/docs/ios/07-visual-effects-glass/#t07-06) | 비공개 API, Core Animation | iOS 바이너리 미검증 |
| 07-07 | iOS · UIKit | 07 시각 효과와 글래스 | [채팅 말풍선이 같은 동적 배경화면을 진짜로 “비춰” 보이기](/docs/ios/07-visual-effects-glass/#t07-07) | 비공개 API | iOS 바이너리 미검증 |
| 07-08 | iOS · UIKit | 07 시각 효과와 글래스 | [`UIVisualEffectView` 내부 필터를 분해해 제어 가능한 내비게이션 블러 얻기](/docs/ios/07-visual-effects-glass/#t07-08) | 비공개 API, Core Animation | iOS 바이너리 미검증 |
| 07-09 | iOS · UIKit | 07 시각 효과와 글래스 | [`CABackdropLayer`와 컬러 매트릭스로 구버전 글래스를 직접 만들기](/docs/ios/07-visual-effects-glass/#t07-09) | 비공개 API, Core Animation | iOS 바이너리 미검증 |
| 07-10 | iOS · UIKit | 07 시각 효과와 글래스 | [비공개 `CAMeshTransform`으로 backdrop 가장자리를 글래스처럼 굴절시키기](/docs/ios/07-visual-effects-glass/#t07-10) | 비공개 API | iOS 바이너리 미검증 |
| 07-11 | iOS · UIKit | 07 시각 효과와 글래스 | [가변 블러 반경을 너비 1픽셀짜리 mask에 인코딩하기](/docs/ios/07-visual-effects-glass/#t07-11) | 비공개 API | iOS 바이너리 미검증 |
| 07-12 | iOS · UIKit | 07 시각 효과와 글래스 | [물결은 변위 맵을 우선 쓰고, 없으면 메시 변형으로 폴백하기](/docs/ios/07-visual-effects-glass/#t07-12) | 비공개 API, 성능과 메모리 | iOS 바이너리 미검증 |
| 07-13 | iOS · UIKit | 07 시각 효과와 글래스 | [직접 그린 Liquid Glass에서 Metal로 비공개 변위 맵의 RG 인코딩 출력하기](/docs/ios/07-visual-effects-glass/#t07-13) | 비공개 API, Metal / GPU | iOS 바이너리 미검증 |
| 07-14 | iOS · UIKit | 07 시각 효과와 글래스 | [프레임마다 글래스를 직접 그리되, 화면에서 벗어나면 비싼 Surface를 즉시 해제하기](/docs/ios/07-visual-effects-glass/#t07-14) | 비공개 API | iOS 바이너리 미검증 |
| 07-15 | iOS · UIKit | 07 시각 효과와 글래스 | [Spoiler에서 비공개 `CAEmitterBehavior`로 반짝임과 인력 구현하기](/docs/ios/07-visual-effects-glass/#t07-15) | 비공개 API, 런타임 해킹, Core Animation, 텍스트와 입력 | iOS 바이너리 미검증 |
| 07-16 | iOS · UIKit | 07 시각 효과와 글래스 | [탭해서 spoiler를 드러낼 때 반전 방사형 mask로 파티클을 접점에서 터뜨리기](/docs/ios/07-visual-effects-glass/#t07-16) | 텍스트와 입력, 성능과 메모리 | iOS 바이너리 미검증 |
| 07-17 | iOS · UIKit | 07 시각 효과와 글래스 | [동적 그라데이션은 프레임마다 shader를 계산하는 것이 아니라 8단계를 미리 구워 `contents`를 애니메이션하기](/docs/ios/07-visual-effects-glass/#t07-17) | Core Animation, 성능과 메모리 | iOS 바이너리 미검증 |
| 07-18 | iOS · UIKit | 07 시각 효과와 글래스 | [여러 동적 배경화면이 공유 단계로 위상을 일치시키기](/docs/ios/07-visual-effects-glass/#t07-18) | 비공개 API | iOS 바이너리 미검증 |
| 07-19 | iOS · UIKit | 07 시각 효과와 글래스 | [Shimmer를 초광폭 mask와 가산 position으로 무한 스크롤하기](/docs/ios/07-visual-effects-glass/#t07-19) | 비공개 API, Core Animation, 제스처와 스크롤 | iOS 바이너리 미검증 |
| 07-20 | iOS · UIKit | 07 시각 효과와 글래스 | [말풍선 찢어진 가장자리에서 “1픽셀 오버슈트”로 타일링 이음새 없애기](/docs/ios/07-visual-effects-glass/#t07-20) | 비공개 API | iOS 바이너리 미검증 |
| 08-01 | iOS · UIKit | 08 렌더링과 성능 | [`IOSurface` 아틀라스 한 장으로 독립된 여러 레이어 담기](/docs/ios/08-rendering-perf/#t08-01) | Core Animation, Metal / GPU | iOS 바이너리 미검증 |
| 08-02 | iOS · UIKit | 08 렌더링과 성능 | [EDR 대형 레이어는 고정 아틀라스를 포기해 32 MB 낭비 피하기](/docs/ios/08-rendering-perf/#t08-02) |  | iOS 바이너리 미검증 |
| 08-03 | iOS · UIKit | 08 렌더링과 성능 | [처음 dirty가 된 subject에서만 그리기를 한 번 요청하기](/docs/ios/08-rendering-perf/#t08-03) |  | iOS 바이너리 미검증 |
| 08-04 | iOS · UIKit | 08 렌더링과 성능 | [비공개 텍스처를 트리플 버퍼링하고 GPU 완료 후 반납하기](/docs/ios/08-rendering-perf/#t08-04) | 비공개 API, Metal / GPU | iOS 바이너리 미검증 |
| 08-05 | iOS · UIKit | 08 렌더링과 성능 | [Objective-C 메서드 교체로 `MTLBinaryArchive` 크래시 피하기](/docs/ios/08-rendering-perf/#t08-05) | 비공개 API, Metal / GPU | 버전 차이 |
| 08-06 | iOS · UIKit | 08 렌더링과 성능 | [저장 마커와 `rename(2)`로 캐시 크래시 루프 막기](/docs/ios/08-rendering-perf/#t08-06) | 성능과 메모리 | iOS 바이너리 미검증 |
| 08-07 | iOS · UIKit | 08 렌더링과 성능 | [타이머 하나로 DCT 애니메이션 전체를 구동하고 배터리 상태에 따라 30 FPS로 낮추기](/docs/ios/08-rendering-perf/#t08-07) | 텍스트와 입력 | iOS 바이너리 미검증 |
| 08-08 | iOS · UIKit | 08 렌더링과 성능 | [같은 스티커는 한 번만 디코딩하고 여러 약한 대상으로 팬아웃하기](/docs/ios/08-rendering-perf/#t08-08) |  | iOS 바이너리 미검증 |
| 08-09 | iOS · UIKit | 08 렌더링과 성능 | [고정 백그라운드 큐 두 개로 병렬 디코딩하고 메인 스레드에 일괄 제출하기](/docs/ios/08-rendering-perf/#t08-09) | Core Animation, 성능과 메모리 | iOS 바이너리 미검증 |
| 08-10 | iOS · UIKit | 08 렌더링과 성능 | [DCT 캐시가 키프레임과 차분 프레임을 YUVA 평면으로 저장하기](/docs/ios/08-rendering-perf/#t08-10) | 성능과 메모리 | iOS 바이너리 미검증 |
| 08-11 | iOS · UIKit | 08 렌더링과 성능 | [“단일 비행(single-flight)” 요청으로 캐시 구축을 공유하고 구독자가 없으면 중단하기](/docs/ios/08-rendering-perf/#t08-11) | 성능과 메모리 | iOS 바이너리 미검증 |
| 08-12 | iOS · UIKit | 08 렌더링과 성능 | [더 높은 해상도의 캐시에서 더 작은 캐시를 즉시 파생하기](/docs/ios/08-rendering-perf/#t08-12) | 성능과 메모리 | iOS 바이너리 미검증 |
| 08-13 | iOS · UIKit | 08 렌더링과 성능 | [`DrawingContext`가 비트맵 버퍼의 소유권을 `CGImage`에 곧바로 넘기기](/docs/ios/08-rendering-perf/#t08-13) |  | iOS 바이너리 미검증 |
| 08-14 | iOS · UIKit | 08 렌더링과 성능 | [제자리 바이트로 IOSurface 속성이 있는 `CVPixelBuffer` 만들기](/docs/ios/08-rendering-perf/#t08-14) | Metal / GPU | iOS 바이너리 미검증 |
| 08-15 | iOS · UIKit | 08 렌더링과 성능 | [GIF 그리드는 디스플레이 링크 하나를 쓰고 두 프레임만 미리 읽기](/docs/ios/08-rendering-perf/#t08-15) |  | iOS 바이너리 미검증 |
| 08-16 | iOS · UIKit | 08 렌더링과 성능 | [각 레이어의 재생 시계를 믿지 않고 벽시계로 `CMSampleBuffer` 재생하기](/docs/ios/08-rendering-perf/#t08-16) | 텍스트와 입력, 성능과 메모리 | iOS 바이너리 미검증 |
| 08-17 | iOS · UIKit | 08 렌더링과 성능 | [작은 크기의 GIF는 디블로킹 필터를 끄고 디코딩 불가 파일은 차단하기](/docs/ios/08-rendering-perf/#t08-17) |  | iOS 바이너리 미검증 |
| 08-18 | iOS · UIKit | 08 렌더링과 성능 | [Lottie 캐시는 한 프레임 걸러 한 번씩 고정으로 래스터화하기](/docs/ios/08-rendering-perf/#t08-18) | 성능과 메모리 | iOS 바이너리 미검증 |
| 08-19 | iOS · UIKit | 08 렌더링과 성능 | [Subcodec이 여러 스프라이트를 인코딩 스트림 하나로 합치고 각자의 영역으로 다시 자르기](/docs/ios/08-rendering-perf/#t08-19) |  | iOS 바이너리 미검증 |
| 08-20 | iOS · UIKit | 08 렌더링과 성능 | [먼지 효과는 파티클 상태 갱신과 그리기를 모두 GPU에 두기](/docs/ios/08-rendering-perf/#t08-20) | Metal / GPU | iOS 바이너리 미검증 |
| 08-21 | iOS · UIKit | 08 렌더링과 성능 | [JPEG 디코딩은 TurboJPEG 빠른 경로를 고르고 vImage는 타일로 나누지 않게 하기](/docs/ios/08-rendering-perf/#t08-21) | 성능과 메모리 | iOS 바이너리 미검증 |
| 08-22 | iOS · UIKit | 08 렌더링과 성능 | [`AVSampleBufferDisplayLayer`를 재사용하고 암묵적 액션을 끄기](/docs/ios/08-rendering-perf/#t08-22) | Core Animation, 성능과 메모리 | iOS 바이너리 미검증 |
| 09-01 | iOS · UIKit | 09 채팅 화면 | [먼저 first responder를 넘기고, 그다음 이전 입력창 해제하기](/docs/ios/09-chat-screen/#t09-01) |  | iOS 바이너리 미검증 |
| 09-02 | iOS · UIKit | 09 채팅 화면 | [텍스트 전송 애니메이션에서 가로축과 세로축의 곡선 분리하기](/docs/ios/09-chat-screen/#t09-02) | 텍스트와 입력 | iOS 바이너리 미검증 |
| 09-03 | iOS · UIKit | 09 채팅 화면 | [미디어 스냅샷을 역방향 스케일로 말풍선에 인계하기](/docs/ios/09-chat-screen/#t09-03) |  | iOS 바이너리 미검증 |
| 09-04 | iOS · UIKit | 09 채팅 화면 | [녹음 구체가 직접 날아가 음성 말풍선으로 들어가기](/docs/ios/09-chat-screen/#t09-04) |  | iOS 바이너리 미검증 |
| 09-05 | iOS · UIKit | 09 채팅 화면 | [미디어 콜라주의 네 모서리마다 독립적으로 병합 형태 정하기](/docs/ios/09-chat-screen/#t09-05) |  | iOS 바이너리 미검증 |
| 09-06 | iOS · UIKit | 09 채팅 화면 | [리액션 영역에서 말풍선의 아래쪽 병합 변 바꾸기](/docs/ios/09-chat-screen/#t09-06) |  | iOS 바이너리 미검증 |
| 09-07 | iOS · UIKit | 09 채팅 화면 | [텍스트 측정값에 따라 늘어나되 최대 열두 줄까지인 입력창](/docs/ios/09-chat-screen/#t09-07) | 텍스트와 입력 | iOS 바이너리 미검증 |
| 09-08 | iOS · UIKit | 09 채팅 화면 | [녹음 버튼을 빠르게 탭하면 모드 바꾸고, 길게 눌러야 녹음 시작하기](/docs/ios/09-chat-screen/#t09-08) |  | iOS 바이너리 미검증 |
| 09-09 | iOS · UIKit | 09 채팅 화면 | [녹음 제스처에서 가로 이동, 취소 이동, 잠금을 나눠 보고하기](/docs/ios/09-chat-screen/#t09-09) | 제스처와 스크롤 | iOS 바이너리 미검증 |
| 09-10 | iOS · UIKit | 09 채팅 화면 | [미디어 패널이 허용될 때만 자동으로 텍스트 입력으로 되돌아가기](/docs/ios/09-chat-screen/#t09-10) | 텍스트와 입력 | iOS 바이너리 미검증 |
| 09-11 | iOS · UIKit | 09 채팅 화면 | [숨겨진 입력창의 높이를 컴포넌트 입력으로 받는 실체 키보드](/docs/ios/09-chat-screen/#t09-11) | 텍스트와 입력, 성능과 메모리 | iOS 바이너리 미검증 |
| 09-12 | iOS · UIKit | 09 채팅 화면 | [처음 나타날 때만 즉시 트리를 만드는 이모지 검색 오버레이](/docs/ios/09-chat-screen/#t09-12) | 텍스트와 입력 | iOS 바이너리 미검증 |
| 09-13 | iOS · UIKit | 09 채팅 화면 | [리스트 업데이트 전에 안정된 오프셋을 한 번만 샘플링하기](/docs/ios/09-chat-screen/#t09-13) |  | iOS 바이너리 미검증 |
| 09-14 | iOS · UIKit | 09 채팅 화면 | [읽지 않은 메시지 구분선이 화면 밖이면 강제로 다시 포커스하기](/docs/ios/09-chat-screen/#t09-14) | 성능과 메모리 | iOS 바이너리 미검증 |
| 09-15 | iOS · UIKit | 09 채팅 화면 | [답장 점프를 새 메시지 도착으로 위장하지 않기](/docs/ios/09-chat-screen/#t09-15) |  | iOS 바이너리 미검증 |
| 09-16 | iOS · UIKit | 09 채팅 화면 | [하이라이트, 인용, 답장 설정을 함께 싣는 점프 요청](/docs/ios/09-chat-screen/#t09-16) | 제스처와 스크롤 | iOS 바이너리 미검증 |
| 09-17 | iOS · UIKit | 09 채팅 화면 | [식별자로 에디터 서식 메뉴를 교체하고 시스템 메뉴의 나머지 항목은 유지하기](/docs/ios/09-chat-screen/#t09-17) |  | iOS 바이너리 미검증 |
| 09-18 | iOS · UIKit | 09 채팅 화면 | [녹음 중에는 인터랙티브 뒤로 가기 제스처 끄기](/docs/ios/09-chat-screen/#t09-18) | 제스처와 스크롤 | iOS 바이너리 미검증 |
| 10-01 | iOS · UIKit | 10 미디어, Stories, 프로필 화면 | [런타임으로 비공개 `CAEmitterBehavior` 호출하기](/docs/ios/10-media-stories-peerinfo/#t10-01) | 비공개 API, 런타임 해킹, Core Animation | iOS 바이너리 미검증 |
| 10-02 | iOS · UIKit | 10 미디어, Stories, 프로필 화면 | [비공개 `_verticalVelocity`로 두 번째 풀다운을 아바타 갤러리로 바꾸는 프로필 화면](/docs/ios/10-media-stories-peerinfo/#t10-02) | 비공개 API, 제스처와 스크롤 | iOS 바이너리 미검증 |
| 10-03 | iOS · UIKit | 10 미디어, Stories, 프로필 화면 | [NV12의 두 평면을 제로 카피로 Metal에 매핑하는 원형 비디오](/docs/ios/10-media-stories-peerinfo/#t10-03) | Metal / GPU, 성능과 메모리 | iOS 바이너리 미검증 |
| 10-04 | iOS · UIKit | 10 미디어, Stories, 프로필 화면 | [먼저 저해상도로 블러한 뒤 같은 커맨드 버퍼에서 합성하는 원형 비디오](/docs/ios/10-media-stories-peerinfo/#t10-04) | Metal / GPU | iOS 바이너리 미검증 |
| 10-05 | iOS · UIKit | 10 미디어, Stories, 프로필 화면 | [상한이 있는 `CVPixelBufferPool`로 백프레셔를 거는 원형 비디오](/docs/ios/10-media-stories-peerinfo/#t10-05) |  | iOS 바이너리 미검증 |
| 10-06 | iOS · UIKit | 10 미디어, Stories, 프로필 화면 | [처리가 끝나기 전까지 픽셀 버퍼와 텍스처를 명시적으로 살려 두기](/docs/ios/10-media-stories-peerinfo/#t10-06) | 비공개 API, Metal / GPU | iOS 바이너리 미검증 |
| 10-07 | iOS · UIKit | 10 미디어, Stories, 프로필 화면 | [블록 단위 히스토그램으로 국소 보정 LUT를 만드는 미디어 편집기](/docs/ios/10-media-stories-peerinfo/#t10-07) | 비공개 API, Metal / GPU | iOS 바이너리 미검증 |
| 10-08 | iOS · UIKit | 10 미디어, Stories, 프로필 화면 | [비공개 출력 텍스처를 재사용하고 불필요한 화면 지우기를 건너뛰는 보정 필터](/docs/ios/10-media-stories-peerinfo/#t10-08) | 비공개 API, Metal / GPU, 성능과 메모리 | iOS 바이너리 미검증 |
| 10-09 | iOS · UIKit | 10 미디어, Stories, 프로필 화면 | [내장 카메라를 전환할 때 이전 프리뷰를 멈추고 원형 반투명 블러 겹치기](/docs/ios/10-media-stories-peerinfo/#t10-09) | Metal / GPU | iOS 바이너리 미검증 |
| 10-10 | iOS · UIKit | 10 미디어, Stories, 프로필 화면 | [이중 전환 뷰로 썸네일과 전체 화면의 모서리 둥글기를 맞추는 GalleryUI](/docs/ios/10-media-stories-peerinfo/#t10-10) |  | iOS 바이너리 미검증 |
| 10-11 | iOS · UIKit | 10 미디어, Stories, 프로필 화면 | [퇴장할 때 복제 레이어의 래스터화 배율을 화면 배율로 재귀적으로 되돌리는 Stories](/docs/ios/10-media-stories-peerinfo/#t10-11) | 성능과 메모리 | iOS 바이너리 미검증 |
| 10-12 | iOS · UIKit | 10 미디어, Stories, 프로필 화면 | [모든 일시정지 원인을 재생 모드 함수 하나로 모으는 Stories](/docs/ios/10-media-stories-peerinfo/#t10-12) | 텍스트와 입력, 제스처와 스크롤 | iOS 바이너리 미검증 |
| 10-13 | iOS · UIKit | 10 미디어, Stories, 프로필 화면 | [탭이 팬에 양보하도록 명시하고, 스크롤 팬과만 함께 인식하는 Stories](/docs/ios/10-media-stories-peerinfo/#t10-13) | 비공개 API, 제스처와 스크롤 | iOS 바이너리 미검증 |
| 10-14 | iOS · UIKit | 10 미디어, Stories, 프로필 화면 | [현재 항목 앞뒤 두 장만 미리 로드하는 아바타 갤러리](/docs/ios/10-media-stories-peerinfo/#t10-14) | 비공개 API, 성능과 메모리 | iOS 바이너리 미검증 |
| 10-15 | iOS · UIKit | 10 미디어, Stories, 프로필 화면 | [전면과 후면 카메라를 전환할 때 첫 프레임을 보류하고 녹화 타임라인 보정하기](/docs/ios/10-media-stories-peerinfo/#t10-15) | 텍스트와 입력 | iOS 바이너리 미검증 |
| 11-01 | iOS · UIKit | 11 비 UI 코어 | [이중 가상 매핑으로 링 버퍼의 랩어라운드 없애기](/docs/ios/11-core-nonui/#t11-01) |  | 공유 모듈 간접 증거 |
| 11-02 | iOS · UIKit | 11 비 UI 코어 | [“소유권 펜스”와 3회 밸브로 공유 SQLite 보호하기](/docs/ios/11-core-nonui/#t11-02) |  | iOS 바이너리 미검증 |
| 11-03 | iOS · UIKit | 11 비 UI 코어 | [CPU 샘플링할 때 PAC를 직접 풀고 스택 순회를 제한하기](/docs/ios/11-core-nonui/#t11-03) |  | iOS 바이너리 미검증 |
| 11-04 | iOS · UIKit | 11 비 UI 코어 | [반쪽 트랜잭션으로 계속하지 않고 크래시를 택하는 알림 확장](/docs/ios/11-core-nonui/#t11-04) |  | iOS 바이너리 미검증 |
| 11-05 | iOS · UIKit | 11 비 UI 코어 | [하드 링크된 여러 경로를 하나의 캐시 축출 단위로 병합](/docs/ios/11-core-nonui/#t11-05) | 성능과 메모리 | iOS 바이너리 미검증 |
| 11-06 | iOS · UIKit | 11 비 UI 코어 | [행 ID 없는 테이블과 영속 구문으로 컴팩트한 ValueBox 구현](/docs/ios/11-core-nonui/#t11-06) | 성능과 메모리 | iOS 바이너리 미검증 |
| 11-07 | iOS · UIKit | 11 비 UI 코어 | [SQLite mmap을 명시적으로 끄고 영속성 트레이드오프를 분리](/docs/ios/11-core-nonui/#t11-07) | 성능과 메모리 | iOS 바이너리 미검증 |
| 11-08 | iOS · UIKit | 11 비 UI 코어 | [MTProxy와 Web 프록시는 주소 하나만 탐색](/docs/ios/11-core-nonui/#t11-08) |  | iOS 바이너리 미검증 |
| 11-09 | iOS · UIKit | 11 비 UI 코어 | [취소 가능한 지연 재귀로 지수 백오프 구현](/docs/ios/11-core-nonui/#t11-09) |  | iOS 바이너리 미검증 |
| 11-10 | iOS · UIKit | 11 비 UI 코어 | [만료될 때 큐 로컬 객체를 해제해 알림 파이프라인 전체를 취소](/docs/ios/11-core-nonui/#t11-10) | 텍스트와 입력 | iOS 바이너리 미검증 |
| 11-11 | iOS · UIKit | 11 비 UI 코어 | [`async`라는 이름의 큐 호출을 같은 큐 안에서 바로 실행하기](/docs/ios/11-core-nonui/#t11-11) | 성능과 메모리 | iOS 바이너리 미검증 |
| 11-12 | iOS · UIKit | 11 비 UI 코어 | [부팅 시각과 가동 시간으로 되돌릴 수 없는 자동 잠금 구현](/docs/ios/11-core-nonui/#t11-12) | 런타임 해킹, 텍스트와 입력 | iOS 바이너리 미검증 |
| 11-13 | iOS · UIKit | 11 비 UI 코어 | [메시지 인덱스를 먼저 모은 뒤 읽지 않음 상태를 한꺼번에 갱신](/docs/ios/11-core-nonui/#t11-13) | 성능과 메모리 | iOS 바이너리 미검증 |
| 11-14 | iOS · UIKit | 11 비 UI 코어 | [새로 시작한 VM에서 Metal 툴체인을 미리 마운트](/docs/ios/11-core-nonui/#t11-14) | Metal / GPU | iOS 바이너리 미검증 |
| 11-15 | iOS · UIKit | 11 비 UI 코어 | [Bazel 그래프에서 SwiftPM용 섀도 소스 트리 생성](/docs/ios/11-core-nonui/#t11-15) |  | iOS 바이너리 미검증 |
| 12-01 | macOS · AppKit | 12 TGUIKit 코어 | [키 윈도우가 아닌 윈도우로 전체 기능의 AppMenu를 직접 그리기](/docs/macos/12-tguikit-core/#t12-01) |  | 개별 검증 안 함 |
| 12-02 | macOS · AppKit | 12 TGUIKit 코어 | [글래스 효과의 내부 레이어와 필터 삭제하기](/docs/macos/12-tguikit-core/#t12-02) | Core Animation | 릴리스됨 · 계속 사용 가능 |
| 12-03 | macOS · AppKit | 12 TGUIKit 코어 | [우선순위로 Window 키보드 디스패치 재작성하기](/docs/macos/12-tguikit-core/#t12-03) | 텍스트와 입력 | 개별 검증 안 함 |
| 12-04 | macOS · AppKit | 12 TGUIKit 코어 | [조건부 `updateLayer`로 UIKit 변환 속성 흉내 내기](/docs/macos/12-tguikit-core/#t12-04) | Core Animation | 개별 검증 안 함 |
| 12-05 | macOS · AppKit | 12 TGUIKit 코어 | [기본 View에서 좌표 뒤집기를 통일하고 암시적 애니메이션 끄기](/docs/macos/12-tguikit-core/#t12-05) | Core Animation | 개별 검증 안 함 |
| 12-06 | macOS · AppKit | 12 TGUIKit 코어 | [빈 CAAction으로 계층 진입 알림 겸용하기](/docs/macos/12-tguikit-core/#t12-06) | Core Animation | 개별 검증 안 함 |
| 12-07 | macOS · AppKit | 12 TGUIKit 코어 | [비공개 KVC 키로 Layer 내용 색칠하기](/docs/macos/12-tguikit-core/#t12-07) | 비공개 API, 런타임 해킹, Core Animation | 릴리스됨 · 계속 사용 가능 |
| 12-08 | macOS · AppKit | 12 TGUIKit 코어 | [장식 View가 히트 테스트를 통과하게 하기](/docs/macos/12-tguikit-core/#t12-08) | Core Animation | 개별 검증 안 함 |
| 12-09 | macOS · AppKit | 12 TGUIKit 코어 | [실제 hit-test로 hover 판정 바로잡기](/docs/macos/12-tguikit-core/#t12-09) |  | 개별 검증 안 함 |
| 12-10 | macOS · AppKit | 12 TGUIKit 코어 | [`NSTrackingArea`로 비활성 윈도우 hover 처리하기](/docs/macos/12-tguikit-core/#t12-10) |  | 개별 검증 안 함 |
| 12-11 | macOS · AppKit | 12 TGUIKit 코어 | [CVDisplayLink 콜백을 메인 큐로 안전하게 넘기기](/docs/macos/12-tguikit-core/#t12-11) |  | 릴리스됨 · 계속 사용 가능 |
| 12-12 | macOS · AppKit | 12 TGUIKit 코어 | [DisplayLink 하나를 공유하고 요청에 따라 프레임 낮추기](/docs/macos/12-tguikit-core/#t12-12) | 성능과 메모리 | 릴리스됨 · 계속 사용 가능 |
| 12-13 | macOS · AppKit | 12 TGUIKit 코어 | [CAMediaTimingFunction의 3차 베지어를 직접 풀기](/docs/macos/12-tguikit-core/#t12-13) |  | 개별 검증 안 함 |
| 12-14 | macOS · AppKit | 12 TGUIKit 코어 | [내비게이션 push 전에 동적 하위 콘텐츠 동결하기](/docs/macos/12-tguikit-core/#t12-14) |  | 개별 검증 안 함 |
| 12-15 | macOS · AppKit | 12 TGUIKit 코어 | [스크롤 휠의 가로 delta를 취소 가능한 인터랙티브 내비게이션으로 바꾸기](/docs/macos/12-tguikit-core/#t12-15) | 제스처와 스크롤 | 개별 검증 안 함 |
| 12-16 | macOS · AppKit | 12 TGUIKit 코어 | [contentView 오버레이로 AppKit sheet 대체하기](/docs/macos/12-tguikit-core/#t12-16) |  | 개별 검증 안 함 |
| 12-17 | macOS · AppKit | 12 TGUIKit 코어 | [모달 컨테이너를 래스터화해 그림자 비용 낮추기](/docs/macos/12-tguikit-core/#t12-17) | 성능과 메모리 | 개별 검증 안 함 |
| 12-18 | macOS · AppKit | 12 TGUIKit 코어 | [트리거 컨트롤 위치에서 커스텀 Popover 확대하기](/docs/macos/12-tguikit-core/#t12-18) |  | 개별 검증 안 함 |
| 12-19 | macOS · AppKit | 12 TGUIKit 코어 | [Tooltip의 좌표를 반전하고 픽셀 정렬하기](/docs/macos/12-tguikit-core/#t12-19) | 제스처와 스크롤 | 개별 검증 안 함 |
| 12-20 | macOS · AppKit | 12 TGUIKit 코어 | [SimpleLayer를 복제해 배경화면 가로 타일링 구현하기](/docs/macos/12-tguikit-core/#t12-20) | Core Animation | 개별 검증 안 함 |
| 12-21 | macOS · AppKit | 12 TGUIKit 코어 | [전역 마우스 이동 폴링으로 Popover를 열어 두기](/docs/macos/12-tguikit-core/#t12-21) |  | 개별 검증 안 함 |
| 13-01 | macOS · AppKit | 13 목록, 스크롤, 텍스트 | [`CAEmitterLayer`의 비공개 동작으로 상호작용 가능한 투명 잉크 만들기](/docs/macos/13-table-scroll-text/#t13-01) | 비공개 API, Core Animation | 릴리스됨 · 계속 사용 가능 |
| 13-02 | macOS · AppKit | 13 목록, 스크롤, 텍스트 | [Core Text로 빈 오프셋을 되짚어 여러 줄 선택 영역의 끝부분 바로잡기](/docs/macos/13-table-scroll-text/#t13-02) | 텍스트와 입력 | 개별 검증 안 함 |
| 13-03 | macOS · AppKit | 13 목록, 스크롤, 텍스트 | [순수 offset 대신 안정적인 식별자로 스크롤 앵커 복원하기](/docs/macos/13-table-scroll-text/#t13-03) | 제스처와 스크롤 | 개별 검증 안 함 |
| 13-04 | macOS · AppKit | 13 목록, 스크롤, 텍스트 | [부드러운 스크롤에서 목표 지점을 저장하고 진행 중인 애니메이션을 동기적으로 마무리하기](/docs/macos/13-table-scroll-text/#t13-04) | 제스처와 스크롤 | 개별 검증 안 함 |
| 13-05 | macOS · AppKit | 13 목록, 스크롤, 텍스트 | [스티키 헤더를 스크롤 뷰와 같은 계층으로 옮기고 뒤집힌 목록에 맞추기](/docs/macos/13-table-scroll-text/#t13-05) | 제스처와 스크롤 | 개별 검증 안 함 |
| 13-06 | macOS · AppKit | 13 목록, 스크롤, 텍스트 | [스티키 헤더가 다음 헤더를 누를 때 y축 밀어내기와 계층을 직접 계산하기](/docs/macos/13-table-scroll-text/#t13-06) |  | 개별 검증 안 함 |
| 13-07 | macOS · AppKit | 13 목록, 스크롤, 텍스트 | [presentation layer로 행 이동을 받아내고 Core Animation을 수동으로 보완하기](/docs/macos/13-table-scroll-text/#t13-07) | Core Animation | 개별 검증 안 함 |
| 13-08 | macOS · AppKit | 13 목록, 스크롤, 텍스트 | [역순 메시지 스트림을 위한 독립적인 증분 diff 작성하기](/docs/macos/13-table-scroll-text/#t13-08) |  | 개별 검증 안 함 |
| 13-09 | macOS · AppKit | 13 목록, 스크롤, 텍스트 | [보이는 행만 재배치하고 수동 행 높이 알림으로 `NSTableView` 구동하기](/docs/macos/13-table-scroll-text/#t13-09) |  | 개별 검증 안 함 |
| 13-10 | macOS · AppKit | 13 목록, 스크롤, 텍스트 | [`TextNode`로 같은 입력의 Core Text layout 재사용하기](/docs/macos/13-table-scroll-text/#t13-10) | 텍스트와 입력, 성능과 메모리 | 개별 검증 안 함 |
| 13-11 | macOS · AppKit | 13 목록, 스크롤, 텍스트 | [`NSTextView`를 우회해 glyph run 단위로 직접 메시지 텍스트 그리기](/docs/macos/13-table-scroll-text/#t13-11) | 텍스트와 입력 | 개별 검증 안 함 |
| 13-12 | macOS · AppKit | 13 목록, 스크롤, 텍스트 | [임베디드 콘텐츠를 텍스트 흐름에 남겨 두고 독립 레이어로 렌더링하기](/docs/macos/13-table-scroll-text/#t13-12) | Core Animation, 텍스트와 입력 | 개별 검증 안 함 |
| 13-13 | macOS · AppKit | 13 목록, 스크롤, 텍스트 | [spoiler로 가릴 때 글자 위에 떠 있는 임베디드 레이어도 함께 숨기기](/docs/macos/13-table-scroll-text/#t13-13) | Core Animation | 개별 검증 안 함 |
| 13-14 | macOS · AppKit | 13 목록, 스크롤, 텍스트 | [입력창은 두 번째 TextKit 스택으로 높이를 측정해 표시 레이아웃 오염 피하기](/docs/macos/13-table-scroll-text/#t13-14) | 텍스트와 입력 | 개별 검증 안 함 |
| 13-15 | macOS · AppKit | 13 목록, 스크롤, 텍스트 | [`NSTextContainer`의 행 조각을 재정의해 인용 블록용 불규칙 여백 확보하기](/docs/macos/13-table-scroll-text/#t13-15) | 텍스트와 입력 | 개별 검증 안 함 |
| 13-16 | macOS · AppKit | 13 목록, 스크롤, 텍스트 | [커스텀 이모지는 `NSTextAttachment`가 아니라 glyph rect 위의 오버레이](/docs/macos/13-table-scroll-text/#t13-16) |  | 개별 검증 안 함 |
| 13-17 | macOS · AppKit | 13 목록, 스크롤, 텍스트 | [전송 키는 먼저 `hasMarkedText()`를 확인해 중국어 등 입력기 후보가 잘리지 않게 하기](/docs/macos/13-table-scroll-text/#t13-17) | 텍스트와 입력 | 개별 검증 안 함 |
| 13-18 | macOS · AppKit | 13 목록, 스크롤, 텍스트 | [입력 컨트롤마다 맞춤법·문법 검사 스위치를 따로 영속화하기](/docs/macos/13-table-scroll-text/#t13-18) |  | 개별 검증 안 함 |
| 13-19 | macOS · AppKit | 13 목록, 스크롤, 텍스트 | [가로 스크롤에서 세로 휠 delta를 곧바로 x로 매핑하고 경계에서 버블링하기](/docs/macos/13-table-scroll-text/#t13-19) | 제스처와 스크롤 | 개별 검증 안 함 |
| 13-20 | macOS · AppKit | 13 목록, 스크롤, 텍스트 | [숫자 카운터는 문자열을 여러 `TextView`로 쪼개 각각 스크롤해 들어오고 나가게 하기](/docs/macos/13-table-scroll-text/#t13-20) | 텍스트와 입력 | 개별 검증 안 함 |
| 13-21 | macOS · AppKit | 13 목록, 스크롤, 텍스트 | [자동 행 높이와 기본 배경을 강제로 끄고 `NSTableView`의 행 기하를 완전히 장악하기](/docs/macos/13-table-scroll-text/#t13-21) | Core Animation, 제스처와 스크롤 | 개별 검증 안 함 |
| 13-22 | macOS · AppKit | 13 목록, 스크롤, 텍스트 | [스크롤 목록 업데이트 시 가시 범위로 애니메이션 대상 거르기](/docs/macos/13-table-scroll-text/#t13-22) | 제스처와 스크롤 | 개별 검증 안 함 |
| 14-01 | macOS · AppKit | 14 비공개 시스템 통합 | [비공개 클래스 이름을 이어 붙이고 함수 포인터로 `CAEmitterBehavior` 호출하기](/docs/macos/14-private-system/#t14-01) | 비공개 API, 런타임 해킹, Core Animation | 릴리스됨 · 계속 사용 가능 |
| 14-02 | macOS · AppKit | 14 비공개 시스템 통합 | [리스닝 전용 `CGEvent` tap으로 전역 푸시투토크 구현하기](/docs/macos/14-private-system/#t14-02) | 텍스트와 입력 | 릴리스됨 · 계속 사용 가능 |
| 14-03 | macOS · AppKit | 14 비공개 시스템 통합 | [FinderInfo 비트와 리소스 포크로 Dock 아이콘을 영구히 바꾸기](/docs/macos/14-private-system/#t14-03) | 성능과 메모리 | 릴리스됨 · 계속 사용 가능 |
| 14-04 | macOS · AppKit | 14 비공개 시스템 통합 | [`AVMovableView`를 깊이 파고들어 숨겨진 전체 화면 버튼 클릭하기](/docs/macos/14-private-system/#t14-04) | Core Animation | 버전 차이 |
| 14-05 | macOS · AppKit | 14 비공개 시스템 통합 | [`NSTitlebarView` 내부 하위 뷰를 재배치해 50 pt 타이틀 바 확보하기](/docs/macos/14-private-system/#t14-05) | 비공개 API | 버전 차이 |
| 14-06 | macOS · AppKit | 14 비공개 시스템 통합 | [비공개 텍스트 메뉴 selector로 시스템 하위 메뉴 삭제하기](/docs/macos/14-private-system/#t14-06) | 비공개 API, 텍스트와 입력 | 시스템 구조 변경됨 |
| 14-07 | macOS · AppKit | 14 비공개 시스템 통합 | [비공개 대각선 크기 조절 커서 selector 직접 호출하기](/docs/macos/14-private-system/#t14-07) | 비공개 API | 릴리스됨 · 계속 사용 가능 |
| 14-08 | macOS · AppKit | 14 비공개 시스템 통합 | [비공개 `MKZoomSegmentedControl` 클래스 이름으로 지도 확대/축소 컨트롤 옮기기](/docs/macos/14-private-system/#t14-08) | 비공개 API | 시스템 구조 변경됨 |
| 14-09 | macOS · AppKit | 14 비공개 시스템 통합 | [`CATransaction`으로 시스템 전체 화면 전환을 감싸고 상태 발행 늦추기](/docs/macos/14-private-system/#t14-09) | Core Animation, 성능과 메모리 | 개별 검증 안 함 |
| 14-10 | macOS · AppKit | 14 비공개 시스템 통합 | [IOPS 요약과 `AppleSmartBattery` 레지스트리 비공개 키 함께 쓰기](/docs/macos/14-private-system/#t14-10) | 비공개 API | 릴리스됨 · 계속 사용 가능 |
| 14-11 | macOS · AppKit | 14 비공개 시스템 통합 | [Carbon 키보드 레이아웃으로 물리 키를 번역하고 비 Unicode 레이아웃은 다시 시도하기](/docs/macos/14-private-system/#t14-11) | 텍스트와 입력 | 릴리스됨 · 계속 사용 가능 |
| 14-12 | macOS · AppKit | 14 비공개 시스템 통합 | [블로킹될 수 있는 입력 모니터링 권한 요청을 메인 스레드 밖으로 옮기기](/docs/macos/14-private-system/#t14-12) | 성능과 메모리 | 개별 검증 안 함 |
| 14-13 | macOS · AppKit | 14 비공개 시스템 통합 | [POSIX 확장 속성으로 Finder 메타데이터 직접 읽고 쓰기](/docs/macos/14-private-system/#t14-13) | 성능과 메모리 | 개별 검증 안 함 |
| 14-14 | macOS · AppKit | 14 비공개 시스템 통합 | [`IOHIDSystem`에서 원시 유휴 나노초 값 읽기](/docs/macos/14-private-system/#t14-14) | 텍스트와 입력 | 릴리스됨 · 계속 사용 가능 |
| 14-15 | macOS · AppKit | 14 비공개 시스템 통합 | [M2 이후의 일반화된 `hw.model` 때문에 `system_profiler` 실행하기](/docs/macos/14-private-system/#t14-15) |  | 릴리스됨 · 계속 사용 가능 |
| 14-16 | macOS · AppKit | 14 비공개 시스템 통합 | [시그널 핸들러 없이 타임스탬프 두 개짜리 파일로 크래시 센티널 만들기](/docs/macos/14-private-system/#t14-16) |  | 개별 검증 안 함 |
| 14-17 | macOS · AppKit | 14 비공개 시스템 통합 | [인덱스를 전부 다시 만들지 않고 안정적인 ID로 Spotlight diff하기](/docs/macos/14-private-system/#t14-17) |  | 개별 검증 안 함 |
| 14-18 | macOS · AppKit | 14 비공개 시스템 통합 | [Dock 배지 문자열을 상태 표시줄 아이콘의 입력으로 쓰기](/docs/macos/14-private-system/#t14-18) | 비공개 API | 개별 검증 안 함 |
| 15-01 | macOS · AppKit | 15 렌더링과 앱 레이어 | [XOR 프레임 차분 후 LZFSE로 스티커 픽셀 캐시 압축하기](/docs/macos/15-rendering-app/#t15-01) | 성능과 메모리 | 릴리스됨 · 계속 사용 가능 |
| 15-02 | macOS · AppKit | 15 렌더링과 앱 레이어 | [같은 인라인 스티커가 디코딩과 재생 컨텍스트 하나를 공유하게 하기](/docs/macos/15-rendering-app/#t15-02) | Core Animation, 성능과 메모리 | 개별 검증 안 함 |
| 15-03 | macOS · AppKit | 15 렌더링과 앱 레이어 | [런 루프 단위로 여러 스티커 드로잉을 Metal 커맨드 버퍼 하나로 합치기](/docs/macos/15-rendering-app/#t15-03) | Metal / GPU | 개별 검증 안 함 |
| 15-04 | macOS · AppKit | 15 렌더링과 앱 레이어 | [GIF는 프레임별 `NSImage` 애니메이션 대신 `AVSampleBufferDisplayLayer`로 직접 프레임 공급하기](/docs/macos/15-rendering-app/#t15-04) | Core Animation | 개별 검증 안 함 |
| 15-05 | macOS · AppKit | 15 렌더링과 앱 레이어 | [통화 프레임 풀이 버퍼 다섯 개에 이르면 프레임을 버려 실시간 지연 우선 제어하기](/docs/macos/15-rendering-app/#t15-05) |  | 개별 검증 안 함 |
| 15-06 | macOS · AppKit | 15 렌더링과 앱 레이어 | [통화 배경은 먼저 색상 포맷을 통일하고, 다운샘플링한 뒤 컴퓨트 블러를 두 번 하기](/docs/macos/15-rendering-app/#t15-06) |  | 개별 검증 안 함 |
| 15-07 | macOS · AppKit | 15 렌더링과 앱 레이어 | [Thanos 효과로 뷰 스냅샷을 GPU 픽셀 파티클로 바꾸고 M2 이상에서만 켜기](/docs/macos/15-rendering-app/#t15-07) | Core Animation, Metal / GPU | 개별 검증 안 함 |
| 15-08 | macOS · AppKit | 15 렌더링과 앱 레이어 | [Thanos 파티클은 항목이 있고 보일 때만 전역 디스플레이 링크 구독하기](/docs/macos/15-rendering-app/#t15-08) | Metal / GPU, 성능과 메모리 | 개별 검증 안 함 |
| 15-09 | macOS · AppKit | 15 렌더링과 앱 레이어 | [CPU 블러는 세 색상 채널을 64비트 누적값에 넣고 슬라이딩 2차 차분 쓰기](/docs/macos/15-rendering-app/#t15-09) | 성능과 메모리 | 개별 검증 안 함 |
| 15-10 | macOS · AppKit | 15 렌더링과 앱 레이어 | [소프트웨어 그라디언트는 아주 작은 비트맵을 먼저 렌더링한 뒤 `contents` 애니메이션하기](/docs/macos/15-rendering-app/#t15-10) | Core Animation, 성능과 메모리 | 개별 검증 안 함 |
| 15-11 | macOS · AppKit | 15 렌더링과 앱 레이어 | [Shimmer는 압축된 경로 썸네일을 비동기 비트맵 마스크로 디코딩하기](/docs/macos/15-rendering-app/#t15-11) | 비공개 API, 성능과 메모리 | 개별 검증 안 함 |
| 15-12 | macOS · AppKit | 15 렌더링과 앱 레이어 | [음성 파문은 원시 볼륨을 지수 평활한 뒤 닫힌 베지어 곡선 만들기](/docs/macos/15-rendering-app/#t15-12) |  | 개별 검증 안 함 |
| 15-13 | macOS · AppKit | 15 렌더링과 앱 레이어 | [채팅 스티커는 크기에 따라 프레임 레이트를 낮추고, 썸네일로 깜빡임 없이 이어받기](/docs/macos/15-rendering-app/#t15-13) | Core Animation, 성능과 메모리 | 개별 검증 안 함 |
| 15-14 | macOS · AppKit | 15 렌더링과 앱 레이어 | [인라인 스티커는 콜드 스타트를 늦추되, 캐시 적중은 즉시 시작하게 하기](/docs/macos/15-rendering-app/#t15-14) | Core Animation, 성능과 메모리 | 개별 검증 안 함 |
| 15-15 | macOS · AppKit | 15 렌더링과 앱 레이어 | [인라인 스티커는 윈도우가 활성이고 스크롤 영역에 보일 때만 재생하기](/docs/macos/15-rendering-app/#t15-15) | 제스처와 스크롤 | 개별 검증 안 함 |
| 15-16 | macOS · AppKit | 15 렌더링과 앱 레이어 | [PiP는 시스템 PiP 컨트롤러에 의존하지 않고, Space를 넘나드는 비활성 패널로 흉내 내기](/docs/macos/15-rendering-app/#t15-16) | 텍스트와 입력 | 개별 검증 안 함 |
| 15-17 | macOS · AppKit | 15 렌더링과 앱 레이어 | [GIF 트랜스코더는 극단적으로 짧은 지연을 보정하고, `CVPixelBuffer`에 직접 그리기](/docs/macos/15-rendering-app/#t15-17) |  | 개별 검증 안 함 |
| 15-18 | macOS · AppKit | 15 렌더링과 앱 레이어 | [소프트웨어 비디오 미리 보기는 `CVPixelBuffer`를 잠근 뒤 vImage로 한 번에 스케일하기](/docs/macos/15-rendering-app/#t15-18) | 성능과 메모리 | 릴리스됨 · 계속 사용 가능 |
