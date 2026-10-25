# 위험과 차용 가능성

웹: /docs/risk/ · Markdown: /docs/risk.md

> 주요 비공개 API와 hack의 위험 등급, 13.0 릴리스 빌드와 macOS 26.7에서의 상태, 그리고 안전하게 차용할 수 있는 방법이에요.

## 위험 등급 [#levels]

위험 등급은 두 단계예요.

- **높음**: 비공개 클래스, 비공개 메서드, 내부 계층 구조에 기대요. 시스템이 바뀌면 동작하지 않을 수 있고, App Store 심사에도 영향을 줄 수 있어요.
- **중간**: 비공개 KVC 키, 내부 이름, 프로세스 수준의 메서드 교체에 기대요. 다만 실패해도 영향이 제한적이거나 조용히 건너뛰어요.

iOS 항목의 “현재 상태” 열은 모두 “미검증”이에요. iOS 바이너리가 FairPlay(Apple의 바이너리 암호화)로 암호화되어 있기 때문이에요. macOS 항목의 상태는 [REA 바이너리 검증](/docs/rea/)에서 가져왔어요.

| API 또는 기법 | 플랫폼 | 용도 | 위험 등급 | 현재 상태 |
| --- | --- | --- | --- | --- |
| `_UIPortalView` | iOS | 레이어를 가로질러 실제 콘텐츠를 미러링 | 높음 | 미검증(iOS 바이너리 암호화) |
| `CALayer.cornerRadii` KVC | iOS | 네 모서리 둥글기 애니메이션 | 높음 | 미검증(iOS 바이너리 암호화) |
| `_scrollHeartbeat` KVC | iOS | 스크롤 고주사율 하트비트 | 높음 | 미검증(iOS 바이너리 암호화) |
| `_solveForInput:` | iOS | 시스템 스프링 값 계산기 | 높음 | 미검증(iOS 바이너리 암호화) |
| `WKWebsiteDataStoreReinitializeAppBoundDomains` | iOS | 동적 App-Bound 도메인 | 높음 | 미검증(iOS 바이너리 암호화) |
| `_verticalVelocity` KVC | iOS | 프로필 화면 두 번째 당겨내리기 판정 | 높음 | 미검증(iOS 바이너리 암호화) |
| `_UILiquidLensView` | iOS | 시스템 Liquid Lens | 높음 | 미검증(iOS 바이너리 암호화) |
| `CAFilter`/`luminanceToAlpha`/`variableBlur` | iOS | 글래스/가변 블러 | 높음 | 미검증(iOS 바이너리 암호화) |
| `CABackdropLayer` | iOS | 구버전 글래스 backdrop | 높음 | 미검증(iOS 바이너리 암호화) |
| `FileManager` IMP 교체 | iOS/macOS | `MTLBinaryArchive` 크래시 회피 | 중간 | 13.0 릴리스됨; 시작 시 트리거되지 않음 |
| `CAEmitterBehavior` | macOS | 인력/수명 주기 동작 | 높음 | 13.0 릴리스됨; 26.7에서 사용 가능 |
| `contentsMultiplyColor` KVC | macOS | 레이어 색조 적용 | 중간 | 13.0 릴리스됨; 26.7에서 사용 가능 |
| `NSVisualEffectView` 내부 레이어와 필터 | macOS | 간소화된 블러 글래스 | 중간 | 13.0 릴리스됨; 26.7에서도 구조 일치, `sdrNormalize`가 추가됨 |
| `NSCursor` 비공개 대각선 커서 | macOS | 윈도우 크기 조절 커서 | 높음 | 13.0 릴리스됨; 26.7에서도 구현되어 있음 |
| `NSTitlebarView` 하위 레이어 재배치 | macOS | 타이틀 바 커스터마이징 | 높음 | 13.0 릴리스됨; 26.7에서도 구조가 남아 있음 |
| `AVMovableView` 깊이 탐색 버튼 | macOS | 플레이어 전체 화면 | 높음 | 13.0 릴리스됨; 26.7에서 클래스는 남아 있으나 버튼 경로는 미검증 |
| `_shareServiceSelected:` 메뉴 필터링 | macOS | 시스템 하위 메뉴 제거 | 중간 | 13.0 릴리스됨; 26.7에서 이름이 바뀌어 동작하지 않을 수 있음 |
| `MKZoomSegmentedControl` | macOS | 지도 확대/축소 컨트롤 이동 | 중간 | 13.0 릴리스됨; 26.7에서 뷰 트리에 없어 조용히 건너뜀 |
| FinderInfo/리소스 포크 | macOS | Dock 아이콘 교체 | 높음 | 13.0 릴리스됨; 실행되지 않음 |

## 안전하게 차용할 수 있는 방법 [#borrow]

아래 일곱 가지 방법은 공개 API나 순수 알고리즘만 써요. 비공개 인터페이스에 기대지 않아서 다른 UIKit, AppKit 프로젝트로 바로 옮길 수 있어요.

1. UI 전환의 “모델 먼저 반영 + additive”와 `presentationLayer`에서 이어받는 애니메이션 연속성.
2. 목록의 “안정적인 식별자 + 앵커 + 플레이스홀더” 차등 갱신과 스크롤 앵커링 전략.
3. 공유 `DisplayLink/Timer`로 여러 애니메이션의 프레임 레이트 요구를 합치고 필요에 따라 프레임을 낮추는 방식.
4. `IOSurface`/픽셀 풀과 백프레셔 임계값을 이용한 리소스 게이트 설계.
5. 텍스트의 “자리 확보용 기하 + 독립 렌더링” 첨부/이모지 아키텍처.
6. 기기/윈도우 안전 영역과 노치 기하를 표로 구동하는 크기 적응(공개 API + 데이터 표).
7. 비동기 그리기의 원자적 취소, 메인 스레드에서의 원자적 반영, 명시적 수명 주기 관리.

## 재사용 가능한 코드 두 조각 [#snippets]

**iOS Shimmer**

그라데이션 이미지 한 장을 모든 shimmer 레이어가 공유하고 `contentsRect`만 애니메이션해요. 같은 비트맵을 쓰므로 프레임마다 다시 그릴 필요가 없어요. 출처: [SparseItemGrid.swift#L131-L145](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/submodules/SparseItemGrid/Sources/SparseItemGrid.swift#L131-L145).

```swift title="SparseItemGrid.swift"
// iOS Shimmer: 공유 그라디언트 이미지 + contentsRect
let animation = CABasicAnimation(keyPath: "contentsRect.origin.y")
animation.fromValue = 1.0 as NSNumber
animation.toValue = -1.0 as NSNumber
animation.isAdditive = true
animation.repeatCount = .infinity
animation.duration = 0.8
layer.add(animation, forKey: "shimmer")
```
**iOS 알림 미들웨어**

`NSNotificationCenter`의 동기 전달을 가로챌 수 있는 미들웨어로 바꿔요. 핸들러는 알림을 가로채거나, 미루거나, 삼킬 수 있어요. 이 코드는 메서드 스위즐링을 쓰기 때문에 “중간” 위험에 속해요. 출처: [NotificationCenterUtils.m#L19-L68](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/submodules/UIKitRuntimeUtils/Source/UIKitRuntimeUtils/NotificationCenterUtils.m#L19-L68).

```objc title="NotificationCenterUtils.m"
// iOS 전역 알림 미들웨어: 가로채기/지연/삼키기 가능
if ([NSThread isMainThread]) {
  for (NotificationHandlerBlock handler in notificationHandlers()) {
    if (handler(aName, anObject, aUserInfo, ^{ /* pass */ })) { return; }
  }
}
```
