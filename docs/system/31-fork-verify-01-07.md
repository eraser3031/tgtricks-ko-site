# macOS fork 장별 검증 (01–07장)

웹: /docs/system/31-fork-verify-01-07/ · Markdown: /docs/system/31-fork-verify-01-07.md

> 01장부터 07장까지가 overtake Telegram iOS fork에서 어떻게 구현되어 있는지, 그리고 버전 차이가 어떤지 검증해요.

> 시스템 설계 · 검증과 보충 · 보충 31 · 도표 0 · 코드 발췌 3 · 인용 43 · 소스 기준선: [TelegramMessenger/Telegram-iOS@f1dd7a2d](https://github.com/TelegramMessenger/Telegram-iOS/tree/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838), [overtake/TelegramSwift@579cebbf](https://github.com/overtake/TelegramSwift/tree/579cebbf0c01fd41b712eff3647fa7f69db9665d)
>
> 검증 대상: [전체 아키텍처와 모듈 계층](/docs/system/01-architecture/), [리액티브와 동시성 모델](/docs/system/02-reactive-concurrency/), [MTProto 전송 계층](/docs/system/03-mtproto-transport/), [인증 키와 다중 데이터 센터](/docs/system/04-auth-keys-datacenters/), [TL 직렬화와 API 호출 계층](/docs/system/05-tl-api-layer/), [업데이트 동기화: pts, qts, seq와 getDifference](/docs/system/06-update-sync/), [메시지 전송 파이프라인과 오프라인 작업 큐](/docs/system/07-message-send-pipeline/)
>
> 미검증 사항:
> - fork에서 TL schema 입력 파일, SwiftTL 전체 재생성 명령, CI 호출을 확인하지 못했어요.
> - macOS 빌드를 실행하지 않았기 때문에 Signal async bridge의 실제 배포 타깃 호환성은 여전히 미검증이에요.
> - fork의 오디오 녹음기에서 MessageMediaPreuploadManager로 이어지는 직접 호출 체인은 행 단위로 추적하지 않았어요.

## 검증 기준 [#검증-기준]

이 보충은 `overtake/Telegram-iOS`의 고정 커밋 `a24bbe45`를 대조해요. 판정 기준은 이래요.

- “동일”: 대상 동작이 fork에 있고, 선택한 핵심 소스 파일이 `f1dd7a2`(업스트림 체크아웃)와 내용이 같다는 뜻이에요.
- “차이 있음”: 양쪽 소스 증거를 함께 보여 줘요.
- “fork에 없음”: fork에서 체크아웃한 트리 전체를 뒤져도 그 모듈이나 식별자를 찾지 못했다는 뜻이에요.

차이는 대부분 버전이 달라서 생겨요. fork의 커밋 날짜는 업스트림 체크아웃보다 앞서요. 명시적인 `#if os(macOS)` 분기를 빼면, 파일 이름만으로 macOS 전용 수정을 추정할 수 없어요.

## 01장 전체 아키텍처와 모듈 계층 [#01장-전체-아키텍처와-모듈-계층]

| 원래 장의 핵심 결론 | fork 위치와 판정 |
|---|---|
| `TelegramCore`가 API, 전송, Signal, Postbox를 조립해요. | **동일**. fork의 target은 `TelegramApi`, `MtProtoKit`, `SwiftSignalKit`, `Postbox`에 직접 의존해요. [\[1\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/BUILD#L3-L31) |
| `TelegramEngine`이 `Account` 하나를 기반으로 지연 초기화되는 도메인 진입점을 제공해요. | **동일**. fork의 `peers`, `messages`, `data` 등의 진입점은 모두 같은 `account`에서 만들어져요. [\[2\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/TelegramEngine/TelegramEngine.swift#L4-L76) |
| `Account`는 단일 계정의 저장소, 네트워크, 백그라운드 관리자를 조립하는 루트예요. | **동일**. `postbox`, `network`, `stateManager`, `pendingMessageManager`를 갖고 있어요. [\[3\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/Account/Account.swift#L973-L1003) |
| `supplementary` 계정은 포그라운드 서비스를 줄여요. | **동일**. fork는 스토리 구독 컨텍스트를 만들지 않고, `!supplementary`를 작업 관리자에 넘겨요. [\[4\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/Account/Account.swift#L1064-L1112) |
| Rust 엔진은 이 버전에서 전환할 수 있는 상위 네트워크 구현이에요. | **fork에 없음**. fork의 Core BUILD에는 `MtProtoKit`만 나열되어 있어요. 업스트림은 `WebProxyTransport`를 선언하고 `RustNetworkEngine`을 만들어요. [\[1\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/BUILD#L3-L31) [\[40\]](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/submodules/MTProtoRustEngine/Sources/MTProtoRustEngine/RustNetworkEngine.swift#L35-L54) [\[42\]](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/submodules/TelegramCore/BUILD#L14-L32) |

fork의 아키텍처 줄기는 여전히 Account, Postbox, Network, Engine이에요. macOS 조건 분기도 실제로 있어요. 예를 들어 테마 포맷이 `macos`와 `palette`로 바뀌어요. 이 분기는 공유 Core를 플랫폼에 맞춘 것이에요. TelegramSwift 바깥 계층이 Core를 중복 구현한다는 증거는 아니에요. [\[43\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/Themes.swift#L1-L14)

## 02장 리액티브와 동시성 모델 [#02장-리액티브와-동시성-모델]

| 원래 장의 핵심 결론 | fork 위치와 판정 |
|---|---|
| `Signal.start`는 구독할 때 generator를 실행해요. | **동일**. fork는 먼저 `Subscriber`를 만들고 `generator`를 호출한 뒤 마지막에 disposable을 바인딩해요. [\[5\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/SSignalKit/SwiftSignalKit/Source/Signal.swift#L63-L75) |
| `MetaDisposable.set`은 교체된 항목을 해제하거나, 늦게 도착한 항목을 즉시 해제해요. | **동일**. fork는 mutex로 상태를 보호하고, 잠금을 푼 뒤 해제를 실행해요. [\[6\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/SSignalKit/SwiftSignalKit/Source/Disposable.swift#L107-L149) |
| `Promise.get`은 현재 값을 재생하고, 구독자를 제거하는 disposable을 돌려줘요. | **동일**. fork는 잠금 밖에서 현재 값을 전달하고, 취소할 때 `Bag`에서 클로저를 제거해요. [\[7\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/SSignalKit/SwiftSignalKit/Source/Promise.swift#L51-L77) |
| `Queue.async`는 소속 큐에서는 동기로 실행되고, `justDispatch`만 항상 비동기예요. | **동일**. 두 메서드는 fork에서도 이 의미를 유지해요. [\[8\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/SSignalKit/SwiftSignalKit/Source/Queue.swift#L50-L78) |
| `Signal`은 Swift async를 브리지할 수 있고, Task 취소를 disposable로 매핑해요. | **동일**. fork는 최소 macOS 10.15를 표기하고, 취소 핸들러에서 `dispose`를 호출해요. [\[9\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/SSignalKit/SwiftSignalKit/Source/Signal.swift#L123-L135) |

위 SwiftSignalKit 파일 다섯 개는 업스트림과 글자 단위로 같아요. Signal, 취소, 큐에 관한 원래 장의 주요 결론은 그대로 둘 수 있어요. macOS 빌드나 런타임 테스트는 아직 실행하지 않았어요. 그래서 async 브리지의 배포 타깃 호환성은 여전히 미검증이에요.

## 03장 MTProto 전송 계층 [#03장-mtproto-전송-계층]

| 원래 장의 핵심 결론 | fork 위치와 판정 |
|---|---|
| `MTProto`가 서비스, 세션, 전송, 연결 상태를 갖고 있어요. | **동일**. fork의 private 필드는 여전히 `_messageServices`, `_sessionInfo`, `_transport`로 역할을 나눠요. [\[10\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/MtProtoKit/Sources/MTProto.m#L107-L165) |
| 클라이언트 `msg_id`는 전역 시간으로 만들고 4의 배수로 맞춰요. | **동일**. fork는 같은 단조성 검사와 정렬 루프를 유지해요. [\[11\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/MtProtoKit/Sources/MTSessionInfo.m#L91-L108) |
| 의미 있는 메시지는 `seq_no`를 두 단계 올리고, 의미 없는 메시지는 올리지 않아요. | **동일**. fork의 `takeSeqNo:`는 원래 장과 일치해요. [\[12\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/MtProtoKit/Sources/MTSessionInfo.m#L278-L291) |
| 컨테이너는 하위 메시지 식별자 매핑을 저장해서 컨테이너 수준 장애를 복구해요. | **동일**. fork는 `msg_container`를 쓰고 `addContainerMessageIdMapping`을 호출해요. [\[13\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/MtProtoKit/Sources/MTProto.m#L1476-L1520) |
| 연결 종료는 1초, 4초, 8초의 단계식 백오프를 써요. | **동일**. fork의 임계값은 여전히 5번째와 20번째예요. [\[14\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/MtProtoKit/Sources/MTTcpConnectionBehaviour.m#L49-L77) |
| Rust와 WebProxy의 엔진 전환은 이 버전의 전송 경로에 속해요. | **fork에 없음**. fork에는 이 두 모듈이나 식별자가 없어요. 업스트림의 Rust factory는 명시적으로 `RustNetworkEngine`을 돌려주고, 업스트림 Core도 `WebProxyTransport`에 의존해요. [\[40\]](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/submodules/MTProtoRustEngine/Sources/MTProtoRustEngine/RustNetworkEngine.swift#L35-L54) [\[42\]](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/submodules/TelegramCore/BUILD#L14-L32) |

따라서 MTProto의 Objective-C 세션, 컨테이너, 백오프 분석은 macOS fork에 그대로 쓸 수 있어요. 원래 장의 Rust, WebProxy, 이중 엔진 전환 단락은 이 fork에 적용되지 않아요. Rust, WebProxy, 이중 엔진 전환이 업스트림에서 나중에 발전한 것이라는 판단은 추론이에요. fork 커밋이 더 이르고, fork에서 보이는 유일한 전송 구현이 여전히 `MtProtoKit`이기 때문이에요.

## 04장 인증 키와 다중 데이터 센터 [#04장-인증-키와-다중-데이터-센터]

| 원래 장의 핵심 결론 | fork 위치와 판정 |
|---|---|
| `MTContext`는 데이터 센터와 selector의 복합 키로 인증 정보를 저장해요. | **동일**. fork는 64비트 복합 키를 만들고 이 키로 인증 정보 테이블을 인덱싱해요. [\[15\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/MtProtoKit/Sources/MTContext.m#L117-L142) |
| 인증 핸드셰이크에는 공개 키 대기, PQ, DH, 키 확인 단계가 있어요. | **동일**. fork는 다섯 상태의 `MTDatacenterAuthStage`를 유지해요. [\[16\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/MtProtoKit/Sources/MTDatacenterAuthMessageService.m#L95-L125) |
| 임시 주 연결과 미디어 연결은 서로 다른 selector를 쓰고, 영구 키가 있으면 바인딩해요. | **동일**. fork는 두 임시 selector에 각각 미디어 역할을 설정하고 `auth.bindTempAuthKey` 페이로드로 바인딩해요. [\[17\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/MtProtoKit/Sources/MTDatacenterAuthAction.m#L69-L89) [\[18\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/MtProtoKit/Sources/MTBindKeyMessageService.m#L55-L91) |
| 교차 DC 인증은 먼저 export하고 import한 뒤, 성공하면 대상 token을 써요. | **동일**. fork의 transfer action이 이 순서를 유지해요. [\[19\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/MtProtoKit/Sources/MTDatacenterTransferAuthAction.m#L87-L166) |
| supplementary 네트워크는 임시 키를 갱신하지 않아요. | **차이 있음**. fork는 `MTContext`를 만든 뒤 갱신 스위치가 없어요. [\[20\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/Network/Network.swift#L472-L508) 업스트림은 곧바로 `context.refreshesTemporaryKeys = !supplementary`를 기록해요. [\[21\]](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/submodules/TelegramCore/Sources/Network/Network.swift#L529-L532) |
| `FILE_MIGRATE_X`에는 다운로드 전용 폴백 재실행이 있어요. | **fork에 없음**. fork는 `auth.sendCode` 분기에서만 PHONE, USER, NETWORK 마이그레이션을 매칭해요. [\[22\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/Authorization.swift#L195-L210) |

- fork의 인증, DH, 바인딩, 교차 센터 인증 주 경로는 성립해요.
- 임시 키 갱신 스위치는 업스트림에서 나중에 생긴 동작이에요. 여기서 거꾸로 fork의 supplementary 네트워크를 추정할 수 없어요.
- `FILE_MIGRATE_X`의 다운로드 폴백은 여전히 미검증이에요. 범용 자동 재실행이라고 부를 수 없어요.

## 05장 TL 직렬화와 API 호출 계층 [#05장-tl-직렬화와-api-호출-계층]

| 원래 장의 핵심 결론 | fork 위치와 판정 |
|---|---|
| `TelegramApi`는 독립 target으로 기존 Sources를 컴파일해요. | **동일**. fork target은 여전히 `glob`으로 생성된 Swift 파일을 모아요. [\[23\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramApi/BUILD#L3-L15) |
| 생성된 함수는 설명, 요청 Buffer, 강타입 파서를 돌려줘요. | **동일**. fork의 `users.getUsers`는 이 3요소 튜플 형태를 유지해요. [\[24\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramApi/Sources/Api39.swift#L12068-L12076) |
| `Network.request`는 응답을 파싱하고, 타입이 맞지 않으면 `TL_VERIFICATION_ERROR`를 돌려줘요. | **동일**. fork는 파싱 결과가 `nil`이거나 타입 cast가 실패하면 여전히 code 500을 돌려줘요. [\[25\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/Network/Network.swift#L1140-L1177) |
| 현재 TL layer는 230이에요. | **차이 있음**. fork의 `currentLayer()`는 210을 돌려줘요. [\[26\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/Serialization.swift#L211-L218) 업스트림은 230을 돌려줘요. [\[27\]](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/submodules/TelegramCore/Sources/State/Serialization.swift#L261-L264) |
| 생성 출력은 Api0부터 Api43까지 총 44개 조각이에요. | **차이 있음**. fork에는 Api0부터 Api39까지 총 40개 조각만 있어요. 업스트림에는 `Api43.swift`가 이미 있어요. [\[24\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramApi/Sources/Api39.swift#L12068-L12076) [\[41\]](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/submodules/TelegramApi/Sources/Api43.swift#L1-L18) |
| checkout에 검증할 수 있는 TL schema 입력과 전체 생성 명령이 있어요. | **fork에 없음**. fork에서는 `.tl` 입력 파일이나 target 안의 생성 명령을 확인하지 못했어요. |

```swift
public func currentLayer() -> UInt {
    return 210
}
```
[\[26\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/Serialization.swift#L211-L218)

fork의 함수 3요소 튜플과 Network 경계는 성립해요. 하지만 wire schema는 layer 210이에요. layer 230, 44개 조각, 새 constructor를 언급하는 원래 장의 서술은 모두 fork에 그대로 쓸 수 없어요. 이 차이는 버전이 달라서 생긴 거예요. macOS 전용 수정의 증거가 아니에요.

## 06장 업데이트 동기화: pts, qts, seq와 getDifference [#06장-업데이트-동기화-pts-qts-seq와-getdifference]

| 원래 장의 핵심 결론 | fork 위치와 판정 |
|---|---|
| 계정 상태는 `pts`, `qts`, `date`, `seq`를 영속화해요. | **동일**. fork의 `AuthorizedAccountState.State` 정의와 인코딩은 네 필드를 모두 유지해요. [\[28\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/SyncCore/SyncCore_AuthorizedAccountState.swift#L12-L36) |
| 상태 관리자는 일곱 종류의 operation으로 동기화를 직렬로 구성해요. | **동일**. fork는 차이 폴링, 수집, 처리, 이벤트, 비동기 최종 상태 operation을 유지해요. [\[29\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/AccountStateManager.swift#L7-L14) |
| 입력은 먼저 pts, qts, 기타 업데이트로 그룹을 나눠요. | **동일**. fork의 `groupUpdates`는 여전히 이 세 가지 그룹을 만들어요. [\[30\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/UpdateGroup.swift#L196-L225) |
| 연속된 접두부만 커밋하고, 첫 번째 공백(hole) 뒤의 pts 업데이트는 보류해요. | **동일**. fork는 `pts - pts_count`를 명시적으로 비교하고 공백 항목을 배열에 추가해요. [\[31\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/AccountStateManagementUtils.swift#L571-L605) |
| `getDifference`는 현재 체크포인트를 쓰고 오래된 스냅샷을 거부해요. | **동일**. fork는 초기 상태와 요청 시점 상태를 비교한 뒤에야 재생해요. [\[32\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/AccountStateManager.swift#L841-L865) |

06장의 핵심 동기화 불변식은 fork에서도 성립해요. fork와 업스트림의 파일은 여전히 행 수가 다르지만, 검증한 상태 필드, 큐, 연속 접두부, 오래된 스냅샷 보호는 바뀌지 않았어요.

## 07장 메시지 전송 파이프라인과 오프라인 작업 큐 [#07장-메시지-전송-파이프라인과-오프라인-작업-큐]

| 원래 장의 핵심 결론 | fork 위치와 판정 |
|---|---|
| 큐에 등록할 때 먼저 `Unsent`가 붙은 로컬 메시지를 쓰고 random id를 만들어요. | **동일**. fork는 `Unsent`와 `OutgoingMessageInfoAttribute.uniqueId`를 함께 설정해요. [\[33\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/PendingMessages/EnqueueMessage.swift#L557-L570) |
| 같은 peer의 앞선 미디어 업로드가 뒤따르는 미디어 업로드를 막아요. | **동일**. fork는 `MessageId` 기준으로 이전 context가 `uploading` 상태인지 검사해요. [\[34\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/PendingMessageManager.swift#L366-L385) |
| 상태 머신은 전송하기 전에 입력 게이트를 거쳐요. | **차이 있음**. fork에는 `waitingToBeSent`와 `sending`만 있어요. [\[35\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/PendingMessageManager.swift#L28-L48) 업스트림은 `waitingForSendGate`, `waitingForForwardSendGate`, `waitingForNewTopic`을 추가로 정의해요. [\[36\]](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/submodules/TelegramCore/Sources/State/PendingMessageManager.swift#L27-L36) |
| `sendMessage`는 random id, 유료 스타 코인, 제안 게시물 파라미터를 유지해요. | **동일**. fork는 여전히 `randomId`, `allowPaidStars`, `suggestedPost`를 전달해요. [\[37\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/PendingMessageManager.swift#L1539-L1546) |
| `sendMessage`는 `scheduleRepeatPeriod`와 `richMessage`도 지원해요. | **차이 있음**. fork의 호출에는 이 두 파라미터가 없어요. [\[37\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/PendingMessageManager.swift#L1539-L1546) 업스트림 호출에는 둘 다 추가되어 있어요. [\[38\]](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/submodules/TelegramCore/Sources/State/PendingMessageManager.swift#L1947-L1954) |
| 영수증은 `updateMessageID.randomId`로 로컬 발신 속성과 매칭해요. | **동일**. fork는 매칭한 뒤 해당 server id로 응답 메시지를 골라요. [\[39\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/ApplyUpdateMessage.swift#L61-L102) |

```swift
case waitingToBeSent(groupId: Int64?, content: PendingMessageUploadedContentAndReuploadInfo)
case sending(groupId: Int64?)
```
[\[35\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/PendingMessageManager.swift#L28-L48)

```swift
case waitingToBeSent(groupId: Int64?, content: PendingMessageUploadedContentAndReuploadInfo)
case waitingForSendGate(groupId: Int64?, content: PendingMessageUploadedContentAndReuploadInfo)
case waitingForForwardSendGate
case sending(groupId: Int64?)
```
[\[36\]](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/submodules/TelegramCore/Sources/State/PendingMessageManager.swift#L27-L36)

07장의 서술은 이렇게 처리해요.

- “메시지가 곧 큐”라는 점, 미디어 직렬화, random id 매핑: 그대로 둘 수 있어요.
- 원래 장의 입력 게이트: fork의 사실로 쓸 수 없어요.
- 새 schema 파라미터: fork의 호출 시그니처에 맞춰 써야 해요.

## 원래 장에 대한 수정 [#원래-장에-대한-수정]

아래는 원래 장마다 고칠 부분이에요. 각 항목은 고칠 위치와 바꿀 내용을 적고, 해결 정도는 **해결됨**이나 **부분 해결**로 표시해요.

- `01-architecture.md`의 “macOS 공유는 Git 서브모듈을 사용한다” 마지막 단락 → “현재 서브모듈이 체크아웃되어 있지 않아 행 단위로 검증할 수 없다”를 삭제해요. “고정된 fork를 체크아웃했으며, Core의 Account, Engine, 기초 의존성을 검증했다. Rust 엔진은 fork에 없다”로 바꿔요. 이로써 fork 핵심 차이의 미검증 사항이 풀렸고, 결과는 **해결됨**이에요.
- `02-reactive-concurrency.md`의 “Objective-C 대응 계층과 작업 스레드” 마지막 단락 → “공유 구현의 버전이 다를 수 있다”를 “이 보충이 검증한 Signal, Disposable, Promise, Queue, Signal_Dispatch 파일은 동일하다”로 바꿔요. “macOS 빌드를 실행하지 않았다”는 제한은 그대로 둬요. 이로써 버전을 알 수 없던 사항이 풀렸고, 결과는 **부분 해결**이에요.
- `03-mtproto-transport.md`의 “MTProtoRustEngine의 공존과 전환” → “업스트림 구현이며 a24bbe45 fork에는 적용되지 않음”으로 표시해요. fork에서 Rust나 WebProxy 폴백을 실행할 수 있다는 표현은 삭제해요. 이는 fork까지 일반화한 원래 장의 결론과 **모순**돼요.
- `04-auth-keys-datacenters.md` 도입부의 fork 미체크아웃 설명 → fork의 DH, 바인딩, transfer 경로를 검증했다는 내용으로 바꿔요. supplementary 임시 키 갱신 스위치는 업스트림에서만 보인다는 점을 보충해요. FILE_MIGRATE 다운로드 폴백 미검증 항목은 그대로 두고, 결과는 **부분 해결**이에요.
- `05-tl-api-layer.md`의 “Layer, flags, schema 진화”에 나오는 layer 230과 44개 조각 → fork의 layer 210과 40개 조각으로 바꿔요. fork layer 미검증 항목은 삭제해요. schema 입력과 전체 재생성 명령은 여전히 체크아웃되지 않았고, 결과는 **부분 해결**이에요.
- `06-update-sync.md` 도입부의 “fork 버전이 다를 수 있으므로 재확인 필요” → 4요소 상태, operation 큐, 공백, 차이 요청을 항목별로 검증했다는 내용으로 바꿔요. 이 사항은 **해결됨**이에요.
- `07-message-send-pipeline.md`의 macOS 버전 면책 문구와 “전송 전에 입력 게이트가 하나 더 있다” → macOS 버전 면책 문구는 검증했다는 내용으로, “전송 전에 입력 게이트가 하나 더 있다”는 문장은 “업스트림에서 추가된 동작이며 fork에서는 확인되지 않음”으로 바꿔요. fork에서 오디오 녹음기가 사전 업로드 관리자를 직접 호출하는 경로는 여전히 미검증이고, 결과는 **부분 해결**이에요.

## 참고 [#references]

- \[1\] [Fork TelegramCore dependencies](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/BUILD#L3-L31)
- \[2\] [Fork TelegramEngine facades](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/TelegramEngine/TelegramEngine.swift#L4-L76)
- \[3\] [Fork Account ownership](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/Account/Account.swift#L973-L1003)
- \[4\] [Fork supplementary account setup](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/Account/Account.swift#L1064-L1112)
- \[5\] [Fork Signal start](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/SSignalKit/SwiftSignalKit/Source/Signal.swift#L63-L75)
- \[6\] [Fork MetaDisposable](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/SSignalKit/SwiftSignalKit/Source/Disposable.swift#L107-L149)
- \[7\] [Fork Promise replay](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/SSignalKit/SwiftSignalKit/Source/Promise.swift#L51-L77)
- \[8\] [Fork Queue dispatch](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/SSignalKit/SwiftSignalKit/Source/Queue.swift#L50-L78)
- \[9\] [Fork Signal async bridge](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/SSignalKit/SwiftSignalKit/Source/Signal.swift#L123-L135)
- \[10\] [Fork MTProto state](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/MtProtoKit/Sources/MTProto.m#L107-L165)
- \[11\] [Fork MTProto message id](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/MtProtoKit/Sources/MTSessionInfo.m#L91-L108)
- \[12\] [Fork MTProto sequence number](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/MtProtoKit/Sources/MTSessionInfo.m#L278-L291)
- \[13\] [Fork MTProto container](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/MtProtoKit/Sources/MTProto.m#L1476-L1520)
- \[14\] [Fork TCP reconnect backoff](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/MtProtoKit/Sources/MTTcpConnectionBehaviour.m#L49-L77)
- \[15\] [Fork data center auth key](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/MtProtoKit/Sources/MTContext.m#L117-L142)
- \[16\] [Fork auth stages](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/MtProtoKit/Sources/MTDatacenterAuthMessageService.m#L95-L125)
- \[17\] [Fork temporary auth roles](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/MtProtoKit/Sources/MTDatacenterAuthAction.m#L69-L89)
- \[18\] [Fork temporary key binding](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/MtProtoKit/Sources/MTBindKeyMessageService.m#L55-L91)
- \[19\] [Fork data center transfer](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/MtProtoKit/Sources/MTDatacenterTransferAuthAction.m#L87-L166)
- \[20\] [Fork network context setup](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/Network/Network.swift#L472-L508)
- \[21\] [Upstream temporary key refresh control](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/submodules/TelegramCore/Sources/Network/Network.swift#L529-L532)
- \[22\] [Fork authorization migration handling](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/Authorization.swift#L195-L210)
- \[23\] [Fork TelegramApi build target](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramApi/BUILD#L3-L15)
- \[24\] [Fork generated getUsers](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramApi/Sources/Api39.swift#L12068-L12076)
- \[25\] [Fork Network request parser](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/Network/Network.swift#L1140-L1177)
- \[26\] [Fork TL layer](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/Serialization.swift#L211-L218)
- \[27\] [Upstream TL layer](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/submodules/TelegramCore/Sources/State/Serialization.swift#L261-L264)
- \[28\] [Fork update state fields](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/SyncCore/SyncCore_AuthorizedAccountState.swift#L12-L36)
- \[29\] [Fork update operations](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/AccountStateManager.swift#L7-L14)
- \[30\] [Fork update grouping](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/UpdateGroup.swift#L196-L225)
- \[31\] [Fork contiguous pts updates](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/AccountStateManagementUtils.swift#L571-L605)
- \[32\] [Fork getDifference snapshot check](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/AccountStateManager.swift#L841-L865)
- \[33\] [Fork unsent message enqueue](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/PendingMessages/EnqueueMessage.swift#L557-L570)
- \[34\] [Fork media upload ordering](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/PendingMessageManager.swift#L366-L385)
- \[35\] [Fork pending message states](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/PendingMessageManager.swift#L28-L48)
- \[36\] [Upstream pending message states](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/submodules/TelegramCore/Sources/State/PendingMessageManager.swift#L27-L36)
- \[37\] [Fork send message parameters](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/PendingMessageManager.swift#L1539-L1546)
- \[38\] [Upstream send message parameters](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/submodules/TelegramCore/Sources/State/PendingMessageManager.swift#L1947-L1954)
- \[39\] [Fork sent message correlation](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/ApplyUpdateMessage.swift#L61-L102)
- \[40\] [Upstream Rust network engine factory](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/submodules/MTProtoRustEngine/Sources/MTProtoRustEngine/RustNetworkEngine.swift#L35-L54)
- \[41\] [Upstream Api43 generated output](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/submodules/TelegramApi/Sources/Api43.swift#L1-L18)
- \[42\] [Upstream Core WebProxy dependency](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/submodules/TelegramCore/BUILD#L14-L32)
- \[43\] [Fork macOS theme branch](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/Themes.swift#L1-L14)
