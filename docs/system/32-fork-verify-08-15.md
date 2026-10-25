# macOS fork 장별 검증(08–15장)

웹: /docs/system/32-fork-verify-08-15/ · Markdown: /docs/system/32-fork-verify-08-15.md

> 08장부터 15장까지의 핵심 결론이 TelegramSwift 고정 fork에서 어떤 구현 상태인지 검증해요.

> 시스템 설계 · 검증과 보충 · 보충 32 · 도표 0 · 코드 발췌 6 · 인용 52 · 소스 기준선: [TelegramMessenger/Telegram-iOS@f1dd7a2d](https://github.com/TelegramMessenger/Telegram-iOS/tree/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838), [overtake/TelegramSwift@579cebbf](https://github.com/overtake/TelegramSwift/tree/579cebbf0c01fd41b712eff3647fa7f69db9665d)
>
> 검증 대상: [Postbox 스토리지 엔진](/docs/system/08-postbox-storage/), [Postbox 뷰와 메시지 이력 모델](/docs/system/09-postbox-views/), [미디어 리소스, 다운로드/업로드, 캐시](/docs/system/10-media-cache/), [종단 간 암호화와 로컬 보안](/docs/system/11-secret-chats-security/), [푸시, 확장 프로세스와 다중 프로세스 협업](/docs/system/12-push-extensions/), [통화 시스템](/docs/system/13-calls/), [다중 계정, 로그인, 구성 동기화](/docs/system/14-accounts-auth-config/), [macOS 클라이언트의 시스템 계층](/docs/system/15-macos-system-layer/)
>
> 미검증 사항:
> - fork의 메시지 이력 공백(hole) 채우기가 실패한 뒤 업스트림처럼 두 번째 RPC 재시도가 없는지(업스트림 maxRetries는 0) 한 줄씩 대조하지 않았어요.
> - 알림 서비스 실행 시 사용 가능한 메모리 상한을 시스템이 어떻게 부과하는지.
> - macOS의 실제 서명, App Group entitlement, 배포 채널, 빌드 결과.

이 보충은 `overtake/Telegram-iOS`의 `a24bbe45f9861f736a79916a50898512f751e0dc`를 대조해요. 이 커밋은 업스트림 `f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838`보다 앞서요. 판정 라벨은 이렇게 읽어요.

- “동일”: 핵심 메커니즘과 인터페이스가 fork 소스에서 보인다는 뜻이에요.
- “차이 있음”: 코드 경로나 상수가 다르다는 뜻이에요.
- “fork에 없음”: 이번에 해당 구현에서 그 메커니즘을 찾지 못했다는 뜻이에요.

## 08장 Postbox 저장 엔진 [#08장-postbox-저장-엔진]

|핵심 결론|fork 위치와 판정|
|---|---|
|단일 큐 트랜잭션은 begin, 클로저, beforeCommit, commit 순서로 실행돼요|`Postbox.internalTransaction`이 이 순서를 유지해요. 동일. [\[1\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/Postbox/Sources/Postbox.swift#L3058-L3075)|
|논리 테이블은 `t`에 정수 ID를 붙인 BLOB 또는 Int64 키 테이블로 매핑돼요|`SqliteValueBox.createTable`이 두 가지 테이블 생성 경로를 유지해요. 동일. [\[2\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/Postbox/Sources/SqliteValueBox.swift#L717-L730)|
|데이터베이스는 WAL, NORMAL 동기화, 메모리 임시 테이블을 써요|fork도 같은 pragma 묶음을 설정해요. 동일. [\[3\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/Postbox/Sources/SqliteValueBox.swift#L447-L462)|
|계정 Postbox의 현재 schema는 25이며 24에서 25로의 업그레이드를 포함해요|fork도 `currentUserVersion`을 25로 정의해요. 동일. [\[4\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/Postbox/Sources/Postbox.swift#L1484-L1550)|
|카운트할 수 있는 열기 실패가 연속 세 번일 때만 전체 데이터베이스 파일 묶음을 삭제해요|fork는 열기에 실패하면 바로 주 데이터베이스 파일 삭제를 시도해요. 세 번의 임계값도 동반 파일 묶음 함수도 없어요. 차이 있음. [\[5\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/Postbox/Sources/SqliteValueBox.swift#L275-L312)[\[6\]](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/submodules/Postbox/Sources/SqliteValueBox.swift#L261-L304)|

fork의 트랜잭션 순서는 업스트림 결론과 일치해요.

```swift
self.valueBox.begin()
let transaction = Transaction(queue: self.queue, postbox: self)
self.afterBegin(transaction: transaction)
let result = f(transaction)
let (updatedTransactionState, updatedMasterClientId) = self.beforeCommit(currentTransaction: transaction)
transaction.disposed = true
self.valueBox.commit()
```
[\[1\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/Postbox/Sources/Postbox.swift#L3058-L3075)

복구 전략은 가져다 쓸 수 없어요. 둘의 동작은 이래요.

- fork: `Database` 생성에 실패하면 `path`에 대해 `removeItem`만 호출하고 바로 `preconditionFailure`를 실행해요.
- 업스트림: 나중에 연속 실패 횟수를 기록하고, 임계값 3에 이르면 주 파일, WAL, SHM, 카운트 파일을 삭제해요. [\[5\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/Postbox/Sources/SqliteValueBox.swift#L275-L312)[\[6\]](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/submodules/Postbox/Sources/SqliteValueBox.swift#L261-L304)

업스트림의 이 방식은 시기가 더 늦은 업스트림 커밋에만 나타나요. 그래서 macOS 전용 수정이 아니라 업스트림의 후속 발전이라고 추론해요.

## 09장 Postbox 뷰와 메시지 이력 모델 [#09장-postbox-뷰와-메시지-이력-모델]

|핵심 결론|fork 위치와 판정|
|---|---|
|`MessageId`는 peer, namespace, id로 구성돼요|fork가 동일한 3단 식별자를 정의해요. 동일. [\[7\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/Postbox/Sources/Message.swift#L3-L35)|
|`MessageIndex`는 timestamp, namespace, id, peer 순으로 정렬돼요|fork의 비교자 순서가 같아요. 동일. [\[7\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/Postbox/Sources/Message.swift#L3-L35)|
|공백(hole) 삭제는 교차하는 구간의 나머지를 다시 만들어요|`MessageHistoryHoleIndexTable.removeInternal`이 `IndexSet`을 써요. 동일. [\[8\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/Postbox/Sources/MessageHistoryHoleIndexTable.swift#L428-L469)|
|이력 뷰는 외부 트랜잭션 reset과 로컬 트랜잭션 replay를 구분해요|`MutableMessageHistoryView`가 두 진입점을 유지해요. 동일. [\[9\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/Postbox/Sources/MessageHistoryView.swift#L305-L490)|
|Quick Reply 네임스페이스는 Local 6, Cloud 7이에요|fork에서는 실제로 Cloud 5, Local 6이고 6까지만 정의되어 있어요. 차이 있음. [\[10\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/SyncCore/SyncCore_Namespaces.swift#L4-L22)|
|macOS는 combined view로 루트 그룹의 읽지 않은 메시지 수를 집계해요|`MainViewController`가 `.unreadCounts`를 구독하고 `messageCount`를 누적해요. 동일. [\[11\]](https://github.com/overtake/TelegramSwift/blob/579cebbf0c01fd41b712eff3647fa7f69db9665d/Telegram-Mac/MainViewController.swift#L439-L451)|

원래 장(09장)의 Quick Reply 수치는 틀렸어요.

- fork와 현재 업스트림 모두 Cloud를 5, Local을 6으로 정해요.
- 업스트림은 추가로 ephemeral 7과 8, welcome message 9와 10을 넣었어요. fork에는 이 후속 공간들이 없어요. [\[10\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/SyncCore/SyncCore_Namespaces.swift#L4-L22)[\[12\]](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/submodules/TelegramCore/Sources/SyncCore/SyncCore_Namespaces.swift#L4-L30)

커밋 시점으로 보면, 추가된 공간은 업스트림의 후속 발전이라고 추론해요.

```swift
public static let ScheduledCloud: Int32 = 3
public static let ScheduledLocal: Int32 = 4
public static let QuickReplyCloud: Int32 = 5
public static let QuickReplyLocal: Int32 = 6
```
[\[10\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/SyncCore/SyncCore_Namespaces.swift#L4-L22)

공백(hole) 채우기가 실패한 뒤의 장기 백오프는 여전히 확인되지 않았어요. fork에서는 뷰와 공백 인덱스의 로컬 재생은 확인할 수 있지만, 실패한 뒤 독립적으로 재시도를 스케줄링하는지는 확인할 수 없어요.

## 10장 미디어 리소스, 다운로드 업로드와 캐시 [#10장-미디어-리소스-다운로드-업로드와-캐시]

|핵심 결론|fork 위치와 판정|
|---|---|
|같은 `MediaResourceId`는 하나의 file context를 공유해요|fork의 `fileContexts`가 리소스 식별자 기준으로 컨텍스트와 참조 카운트를 다시 써요. 동일. [\[13\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/Postbox/Sources/MediaBox.swift#L595-L630)|
|완전한 파일, `_partial`, `.meta`는 각각 세 종류의 로컬 상태를 나타내요|fork는 컨텍스트를 만들 때 세 경로를 건드려요. 동일. [\[13\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/Postbox/Sources/MediaBox.swift#L595-L630)|
|읽기 측 범위는 우선순위에 따라 하나의 pending fetch로 합쳐져요|`MediaBoxFileContextV2Impl`이 materialized range 요청을 유지해요. 동일. [\[14\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/Postbox/Sources/MediaBoxFileContextV2Impl.swift#L330-L350)|
|한 번에 완전한 파일 구간을 읽을 수 있는 크기는 최대 64 MiB예요|`internal_resourceData`가 64 MiB 상한을 유지해요. 동일. [\[15\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/Postbox/Sources/MediaBox.swift#L698-L725)|
|CDN 리디렉션은 key와 IV를 먼저 검증한 뒤 CDN으로 전환해요|fork는 `.switchToCdn`을 직접 구성해요. 검증 함수를 호출하지 않아요. 차이 있음. [\[16\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/Network/MultipartFetch.swift#L370-L380)[\[17\]](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/submodules/TelegramCore/Sources/Network/MultipartFetch.swift#L372-L389)|
|macOS는 공유 FetchManager를 쓰고, 계정 수준의 취소 가능한 정리기가 있어요|패키지가 공유 코어에 의존하고, `AccountContext`가 `FetchManagerImpl`을 만들고, `AccountClearCache`가 run과 cancel을 제공해요. 동일. [\[18\]](https://github.com/overtake/TelegramSwift/blob/579cebbf0c01fd41b712eff3647fa7f69db9665d/packages/FetchManager/Package.swift#L6-L34)[\[19\]](https://github.com/overtake/TelegramSwift/blob/579cebbf0c01fd41b712eff3647fa7f69db9665d/Telegram-Mac/AccountContext.swift#L703-L713)[\[20\]](https://github.com/overtake/TelegramSwift/blob/579cebbf0c01fd41b712eff3647fa7f69db9665d/Telegram-Mac/ClearCache.swift#L154-L218)|

fork와 업스트림 모두 초기 `parsedPartHashes`를 비워요. 차이는 업스트림이 비운 뒤 key와 IV를 `Data`로 바꿔서 유효하지 않은 재료를 거부한다는 점이에요. fork는 key와 IV를 CDN 상태 머신에 바로 넘겨요.

```swift
parsedPartHashes.removeAll()
return .fail(.switchToCdn(id: dcId, token: fileToken.makeData(), key: encryptionKey.makeData(), iv: encryptionIv.makeData(), partHashes: parsedPartHashes))
```
[\[16\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/Network/MultipartFetch.swift#L370-L380)

업스트림에 새로 추가된 호출은 다음과 같아요. 커밋 시점으로 보면, 이는 후속으로 입력 검증을 강화한 것이고 macOS 전용 분기가 아니라고 추론해요.

```swift
let key = encryptionKey.makeData()
let iv = encryptionIv.makeData()
if !cdnRedirectKeyMaterialIsValid(encryptionKey: key, encryptionIv: iv) {
    return .fail(.fatal)
}
```
[\[17\]](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/submodules/TelegramCore/Sources/Network/MultipartFetch.swift#L372-L389)

원래 장의 분할, 범위, 캐시에 관한 주된 결론은 fork에도 적용돼요. 용량을 비울 때 엄밀한 LRU나 LFU를 쓰는지는 이번 fork 읽기로도 확인되지 않았어요.

## 11장 종단 간 암호화와 로컬 보안 [#11장-종단-간-암호화와-로컬-보안]

|핵심 결론|fork 위치와 판정|
|---|---|
|`SecretChatState`는 role, embedded state, keychain, 지문, 자기 파괴 초 수를 가져요|fork의 타입과 영속화 필드가 일치해요. 동일. [\[21\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/SyncCore/SyncCore_SecretChatState.swift#L581-L609)|
|수신 핸드셰이크는 256바이트 b를 생성하고 initial handshake accept를 큐에 넣어요|fork는 `SecRandomCopyBytes`를 호출한 뒤 아웃바운드 작업에 써요. 동일. [\[22\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/SecretChats/UpdateSecretChat.swift#L97-L117)|
|Secret Chat v2는 최소 12바이트의 무작위 패딩을 넣고 정렬해요|fork의 암호화 분기가 그 루프를 유지해요. 동일. [\[23\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/SecretChats/SecretChatEncryption.swift#L286-L310)|
|PFS의 request, accept, commit, abort는 모두 작업 로그가 처리해요|fork가 이 action들과 rekey state를 유지해요. 동일. [\[24\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/ManagedSecretChatOutgoingOperations.swift#L146-L155)|
|메시지 100개 또는 일주일이 되면 자동으로 PFS를 시작해요|fork에서는 그 임계값을 시작 작업으로 바꾸는 스케줄 지점을 찾지 못했어요. fork에 없음.|
|기기 전용 데이터베이스 key 파일은 원자적으로 쓰이고 프로세스 잠금이 있어요|fork는 `atomically:false`로 `.tempkey`를 쓰고 잠금이 없어요. 차이 있음. [\[25\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/BuildConfig/Sources/BuildConfig.m#L361-L381)[\[26\]](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/submodules/BuildConfig/Sources/BuildConfig.m#L380-L434)|
|macOS 지문 화면은 공유 `SecretChatState`를 읽어요|`SecretChatKeyViewController`가 SHA1과 SHA256을 합쳐요. 동일. [\[27\]](https://github.com/overtake/TelegramSwift/blob/579cebbf0c01fd41b712eff3647fa7f69db9665d/Telegram-Mac/SecretChatKeyViewController.swift#L38-L123)|

원래 장이 말한 원자적 쓰기와 약 2초의 자문 잠금은 더 새로운 업스트림 구현에 속해요. a24 fork를 설명하는 데는 쓸 수 없어요. fork의 실제 코드는 다음과 같아요.

```objc
int result = SecRandomCopyBytes(kSecRandomDefault, randomData.length, [randomData mutableBytes]);
if (currentData != nil && currentData.length == 32) {
    [currentData getBytes:randomData.mutableBytes length:32];
}
assert(result == 0);
resultData = randomData;
[resultData writeToFile:filePath atomically:false];
```
[\[25\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/BuildConfig/Sources/BuildConfig.m#L361-L381)

업스트림은 나중에 잠금을 쓰고 원자적 쓰기로 바꿨어요. [\[26\]](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/submodules/BuildConfig/Sources/BuildConfig.m#L380-L434) 메인 앱과 확장이 동시에 처음으로 key를 만들 때 복구되는 방식이 달라요. 이 차이는 업스트림의 후속 수정이라고 추론해요.

## 12장 푸시, 확장 프로세스와 다중 프로세스 협업 [#12장-푸시-확장-프로세스와-다중-프로세스-협업]

|핵심 결론|fork 위치와 판정|
|---|---|
|알림 서비스는 App Group의 `telegram-data`에서 임시 AccountManager를 열어요|fork도 여전히 group 경로, 임시 연결, 캐시 닫기, 오류로 인한 데이터베이스 삭제 금지를 써요. 동일. [\[28\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/Telegram/NotificationService/Sources/NotificationService.swift#L696-L718)|
|알림 복호화는 DC auth key가 아니라 `master-notification-secret`을 써요|fork가 같은 이름의 key, 256바이트 무작위 재료, payload 검증을 정의해요. 동일. [\[29\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/Account/Account.swift#L786-L889)|
|읽기와 쓰기 트랜잭션은 각각 deferred와 immediate를 쓰고 WAL checkpoint는 passive예요|fork의 `SqliteValueBox`가 이 분기를 유지해요. 동일. [\[30\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/Postbox/Sources/SqliteValueBox.swift#L528-L550)|
|VoIP push는 먼저 CallKit에 보고하고 그다음 계정 상태 관리자에 넘겨요|fork의 AppDelegate가 두 호출과 2초의 백그라운드 연장을 유지해요. 동일. [\[31\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramUI/Sources/AppDelegate.swift#L2160-L2243)|
|알림 서비스 연결의 총 예산은 25초, poll 예산은 15초예요|fork에는 `connectionDeadline`이나 `pollBudget`이 없어요. 보이는 미디어와 소리 fetch는 10초 타임아웃이에요. 차이 있음. [\[32\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/Telegram/NotificationService/Sources/NotificationService.swift#L1534-L1645)[\[33\]](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/Telegram/NotificationService/Sources/NotificationService.swift#L1521-L1543)|
|알림 서비스에는 고정된 시스템 수준 메모리 상한이 있어요|fork 소스는 고정 메모리 상한을 정의하지 않아요. fork에 없음.|

타임아웃은 fork와 업스트림이 달라요.

- fork의 미디어 경로: 실제로 미디어와 소리에 각각 10초 timeout을 적용해요. [\[32\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/Telegram/NotificationService/Sources/NotificationService.swift#L1534-L1645)
- 업스트림: poll 분기에 handler 시점부터 계산하는 25초 연결 마감과 15초 poll 상한이 추가되었어요. [\[33\]](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/Telegram/NotificationService/Sources/NotificationService.swift#L1521-L1543)

이 25초와 15초는 업스트림의 후속 시간 제한 전략으로 적어야 해요. macOS fork에서 실제로 확인한 것으로 다루면 안 돼요.

## 13장 통화 시스템 [#13장-통화-시스템]

|핵심 결론|fork 위치와 판정|
|---|---|
|1:1 채팅 세션에는 requesting, ringing, active, dropping, terminated 등의 명시적 상태가 있어요|fork가 같은 상태 머신과 conference 전환 상태를 유지해요. 동일. [\[34\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/CallSessionManager.swift#L81-L110)|
|DH 확인은 key id와 `keyVisualHash`를 계산해요|fork는 gA hash, 공유 key, 시각 해시를 써요. 동일. [\[35\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/CallSessionManager.swift#L509-L555)|
|API 시그널링은 먼저 버퍼링하고 수신기가 준비되면 전달해요|fork가 `enqueuedSignalingData`를 유지해요. 동일. [\[35\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/CallSessionManager.swift#L509-L555)|
|그룹 통화 회의 E2E에는 체인 블록과 나간 멤버 정리가 있어요|fork에 `ConferenceCallE2EContext`와 cleanup 호출이 있어요. 동일. [\[36\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/ConferenceCallE2EContext.swift#L328-L417)|
|macOS는 Core 세션이 active가 된 뒤에야 `OngoingCallContext`를 만들어요|`PCallSession`이 active 분기에서 key, 연결, 기기, P2P 매개변수를 전달해요. 동일. [\[37\]](https://github.com/overtake/TelegramSwift/blob/579cebbf0c01fd41b712eff3647fa7f69db9665d/Telegram-Mac/PCallSession.swift#L1014-L1027)|
|로컬에서 통화를 끝내면 서버에 실제 통화 시간을 보고해요|fork는 2001 epoch의 `CFAbsoluteTime`에서 Unix start date를 빼고 0으로 잘라내요. 차이 있음. [\[38\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/CallSessionManager.swift#L841-L855)[\[39\]](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/submodules/TelegramCore/Sources/State/CallSessionManager.swift#L848-L858)|
|`tgcalls` 서브모듈이 체크아웃되지 않아 미디어 소스를 읽을 수 없어요|현재 작업 트리에는 `tgcalls`가 체크아웃되어 있고 `TelegramMedia`가 `TgVoipWebrtc`에 의존해요. 원래 주장은 현재 체크아웃과 모순돼요. [\[40\]](https://github.com/overtake/TelegramSwift/blob/579cebbf0c01fd41b712eff3647fa7f69db9665d/packages/PrivateCallScreen/Package.swift#L18-L37)|

fork의 통화 시간 코드는 다음과 같아요.

```swift
case let .active(id, accessHash, beginTimestamp, _, _, _, _, _, _, _, _, _):
    let duration = max(0, Int32(CFAbsoluteTimeGetCurrent()) - beginTimestamp)
```
[\[38\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/CallSessionManager.swift#L841-L855)

업스트림 코드는 epoch(시간 기준점)가 일치하지 않는다는 점을 이미 명시했고, 서버 근사 시간을 쓰도록 바꿨어요. [\[39\]](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/submodules/TelegramCore/Sources/State/CallSessionManager.swift#L848-L858)

그래서 원래 장의 “종료 경로가 통화한 초 수를 싣는다”는 말은, fork에 대해서는 필드가 전달된다는 것만 설명할 뿐 값이 올바르다는 것은 보장하지 못해요. 네 가지 이모지의 구체적인 매핑은 fork에서 읽은 Swift와 macOS 호출 계층에서 여전히 찾지 못했어요.

## 14장 다중 계정, 로그인과 설정 동기화 [#14장-다중-계정-로그인과-설정-동기화]

|핵심 결론|fork 위치와 판정|
|---|---|
|`AccountRecordId`는 무작위 64비트 디렉터리 식별자이고 레코드에 임시 세션 ID가 있어요|fork의 구조와 무작위 생성 방식이 일치해요. 동일. [\[41\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/AccountManager/AccountRecord.swift#L28-L71)|
|디렉터리 레코드 삭제는 `secure:false`를 써요|fork의 record table이 그 삭제 매개변수를 유지해요. 동일. [\[42\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/AccountManager/AccountManagerRecordTable.swift#L42-L51)|
|AccountManager는 직렬 트랜잭션에서 beforeCommit 뒤에 ValueBox를 커밋해요|fork가 그 커밋 경계를 유지해요. 동일. [\[43\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/AccountManager/AccountManagerImpl.swift#L174-L266)|
|인증 전과 인증 후 상태는 같은 account factory가 복원해요|fork의 `accountWithId`가 두 종류의 상태 분기를 유지해요. 동일. [\[44\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/Account/Account.swift#L251-L365)|
|전화 인증 코드는 `auth.sendCode`를 거치고, 비밀번호 요구는 `SESSION_PASSWORD_NEEDED`에서 변환돼요|fork가 같은 호출과 오류 분기를 유지해요. 동일. [\[45\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/Authorization.swift#L174-L241)|
|App config는 hash로 가져오고, 변경이 없거나 오류이면 캐시를 덮어쓰지 않으며, 매시간 suspend aware로 폴링해요|fork의 `managedAppConfigurationUpdates`가 같아요. 동일. [\[46\]](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/ManagedAppConfigurationUpdates.swift#L7-L54)|
|macOS AuthController는 UI만 담당하고 인증 동작은 공유 코어로 들어가요|`UnauthorizedAccount`의 state view를 저장하고 `authorizeWithCode`를 호출해요. 동일. [\[47\]](https://github.com/overtake/TelegramSwift/blob/579cebbf0c01fd41b712eff3647fa7f69db9665d/Telegram-Mac/AuthController.swift#L247-L266)|

이 장에서 fork와 다른 부분은 주로 이후의 API 필드와 생성 코드의 형태이고, 계정 모델은 아니에요. 기존의 “fork 미체크아웃” 제한은 삭제해야 해요. `cleanupAccount`가 각 종류의 디렉터리와 미디어 파일을 하나씩 삭제하는 부분은 이 보충에서 펼치지 않았어요. 그래서 그 세부 사항은 계속 미검증 사항으로 남겨요.

## 15장 macOS 클라이언트의 시스템 계층 [#15장-macos-클라이언트의-시스템-계층]

|핵심 결론|fork 또는 TelegramSwift 위치와 판정|
|---|---|
|TelegramSwift는 package 경로로 fork의 TelegramCore, Postbox, SSignalKit을 참조해요|`PrivateCallScreen` 매니페스트가 이 상대 경로 의존성을 직접 선언해요. 동일. [\[40\]](https://github.com/overtake/TelegramSwift/blob/579cebbf0c01fd41b712eff3647fa7f69db9665d/packages/PrivateCallScreen/Package.swift#L18-L37)|
|컨테이너 루트는 App Group에서 오며, 비어 있는 새 디렉터리에서 옛 내용을 마이그레이션해요|`ApiEnvironment.containerURL`과 `migrate`가 그 조건을 유지해요. 동일. [\[48\]](https://github.com/overtake/TelegramSwift/blob/579cebbf0c01fd41b712eff3647fa7f69db9665d/packages/ApiCredentials/Sources/ApiCredentials/Config.swift#L23-L57)|
|계정 전환은 `SharedAccountContext.switchToAccount`로 현재 레코드를 갱신해요|macOS 코드는 AccountManager 트랜잭션에서 current id를 설정해요. 동일. [\[49\]](https://github.com/overtake/TelegramSwift/blob/579cebbf0c01fd41b712eff3647fa7f69db9665d/Telegram-Mac/SharedAccountContext.swift#L634-L668)|
|데스크톱 알림은 alert, badge, sound를 요청하고 peer와 최대 메시지 ID에 따라 정리해요|`UNUserNotifications`의 권한 요청과 정리 로직이 일치해요. 동일. [\[50\]](https://github.com/overtake/TelegramSwift/blob/579cebbf0c01fd41b712eff3647fa7f69db9665d/Telegram-Mac/UNUserNotifications.swift#L340-L364)|
|Sparkle 래퍼는 `TelegramUpdater`에 있어요|`AppUpdateViewController`가 여전히 `SUUpdaterPrivate` 래퍼를 정의해요. 동일. [\[51\]](https://github.com/overtake/TelegramSwift/blob/579cebbf0c01fd41b712eff3647fa7f69db9665d/Telegram-Mac/AppUpdateViewController.swift#L21-L45)|
|크래시 표식은 두 타임스탬프의 차이가 10초 미만이면 비정상으로 판정해요|`CrashHandler`가 이 알고리즘을 유지해요. 동일. [\[52\]](https://github.com/overtake/TelegramSwift/blob/579cebbf0c01fd41b712eff3647fa7f69db9665d/packages/CrashHandler/Sources/CrashHandler/CrashHandler.swift#L11-L34)|
|`submodules/telegram-ios`와 `submodules/tgcalls`는 초기화되지 않았어요|둘 다 현재 읽을 수 있고 커밋은 각각 a24와 5ddc예요. 원래 주장은 현재 체크아웃과 모순돼요. [\[40\]](https://github.com/overtake/TelegramSwift/blob/579cebbf0c01fd41b712eff3647fa7f69db9665d/packages/PrivateCallScreen/Package.swift#L18-L37)|

로컬에서 검증해 보니 TelegramSwift가 고정한 두 코어 서브모듈(telegram-ios와 tgcalls)이 이미 있어요. 그래서 15장은 “Gitlink만으로 추론”에서 “고정된 fork 소스를 읽었으나 더 새로운 업스트림과 같다고 볼 수는 없음”으로 바꿀 수 있어요. 이렇게 하면 08, 10, 11, 12, 13, 14장에 있던 같은 fork 미체크아웃 제한도 함께 풀려요.

## 원래 장들에 대한 수정 [#원래-장들에-대한-수정]

- `08-postbox-storage.md`
  - 16행의 “서브모듈 미초기화” → “a24 fork 체크아웃됨, 해당 커밋 기준으로 검증 필요”로 바꿔요. 이 항목은 해결되었어요.
  - 연속 세 번 열기 실패 시 삭제하는 안전장치는 업스트림 전용으로 표시하고, fork에는 적용되지 않아요. 결과는 원문의 macOS 외삽과 모순돼요.
- `09-postbox-views.md`
  - 12행의 Quick Reply 번호 → Cloud 5, Local 6으로 바꿔요. 이 항목은 해결되었고, 원문과 모순돼요.
  - 393행의 미체크아웃 설명도 fork 체크아웃됨으로 바꿔요. 이 항목은 해결되었어요.
  - 공백(hole) 채우기의 장기 백오프는 여전히 미검증이에요.
- `10-media-cache.md`
  - 427~437행의 서브모듈 미체크아웃과 패키지 해석 불가 추론 → 미체크아웃 전제를 삭제하고 미빌드 결론은 유지해요. 결과는 부분 해결이에요.
  - CDN key와 IV 검증은 더 새로운 업스트림 구현으로 한정해요.
- `11-secret-chats-security.md`
  - 65, 370, 460, 484행 → fork가 체크아웃되었어요. 이 항목은 해결되었어요.
  - 데이터베이스 key 파일은 fork에서 비원자적 쓰기이고 자문 잠금이 없어요. 이 항목은 원문과 모순돼요.
  - PFS 자동 시작 임계값은 여전히 미검증이에요.
- `12-push-extensions.md`
  - 140, 215, 468행 → 25초와 15초는 더 새로운 업스트림에만 표시해요.
  - 고정 메모리 상한은 여전히 미검증이에요.
  - fork 미초기화 설명은 삭제해요.
  - 결과는 부분 해결이에요.
- `13-calls.md`
  - 436, 444행 → 두 서브모듈 미체크아웃 설명을 삭제해요. 이 항목은 해결되었어요.
  - 종료 시간 설명은 fork가 시간 필드를 전달하지만 현재의 계산은 0으로 잘라낸다는 것으로 바꿔요. 이 항목의 결과는 원문 결론이 불완전하다는 거예요.
- `14-accounts-auth-config.md`
  - 6, 158행 → fork 체크아웃됨으로 바꾸고 “버전이 업스트림과 다름”은 유지해요. 결과는 해결이에요.
  - `cleanupAccount`가 파일을 하나씩 삭제하는 범위는 여전히 미검증이에요.
- `15-macos-system-layer.md`
  - 8, 14, 440, 446, 450행 → telegram-ios와 tgcalls 모두 읽을 수 있는 것으로 바꿔요.
  - 서명, App Group entitlement, 실제 배포 flavor, 빌드 실행 결과는 미검증으로 유지해요.
  - 결과는 fork 가독성 항목이 해결되었다는 거예요.

## 참고 자료 [#references]

- \[1\] [fork Postbox transaction](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/Postbox/Sources/Postbox.swift#L3058-L3075)
- \[2\] [fork ValueBox table creation](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/Postbox/Sources/SqliteValueBox.swift#L717-L730)
- \[3\] [fork SQLite pragmas](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/Postbox/Sources/SqliteValueBox.swift#L447-L462)
- \[4\] [fork Postbox schema version](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/Postbox/Sources/Postbox.swift#L1484-L1550)
- \[5\] [fork database open failure](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/Postbox/Sources/SqliteValueBox.swift#L275-L312)
- \[6\] [upstream database open failure valve](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/submodules/Postbox/Sources/SqliteValueBox.swift#L261-L304)
- \[7\] [fork message identity](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/Postbox/Sources/Message.swift#L3-L35)
- \[8\] [fork message hole removal](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/Postbox/Sources/MessageHistoryHoleIndexTable.swift#L428-L469)
- \[9\] [fork message history view replay](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/Postbox/Sources/MessageHistoryView.swift#L305-L490)
- \[10\] [fork message namespaces](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/SyncCore/SyncCore_Namespaces.swift#L4-L22)
- \[11\] [macOS unread combined view](https://github.com/overtake/TelegramSwift/blob/579cebbf0c01fd41b712eff3647fa7f69db9665d/Telegram-Mac/MainViewController.swift#L439-L451)
- \[12\] [upstream extended namespaces](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/submodules/TelegramCore/Sources/SyncCore/SyncCore_Namespaces.swift#L4-L30)
- \[13\] [fork MediaBox file contexts](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/Postbox/Sources/MediaBox.swift#L595-L630)
- \[14\] [fork range request merge](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/Postbox/Sources/MediaBoxFileContextV2Impl.swift#L330-L350)
- \[15\] [fork 64 MiB media read](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/Postbox/Sources/MediaBox.swift#L698-L725)
- \[16\] [fork CDN redirect](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/Network/MultipartFetch.swift#L370-L380)
- \[17\] [upstream CDN key validation](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/submodules/TelegramCore/Sources/Network/MultipartFetch.swift#L372-L389)
- \[18\] [macOS FetchManager package](https://github.com/overtake/TelegramSwift/blob/579cebbf0c01fd41b712eff3647fa7f69db9665d/packages/FetchManager/Package.swift#L6-L34)
- \[19\] [macOS FetchManager construction](https://github.com/overtake/TelegramSwift/blob/579cebbf0c01fd41b712eff3647fa7f69db9665d/Telegram-Mac/AccountContext.swift#L703-L713)
- \[20\] [macOS cache cleaner](https://github.com/overtake/TelegramSwift/blob/579cebbf0c01fd41b712eff3647fa7f69db9665d/Telegram-Mac/ClearCache.swift#L154-L218)
- \[21\] [fork secret chat state](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/SyncCore/SyncCore_SecretChatState.swift#L581-L609)
- \[22\] [fork secret handshake](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/SecretChats/UpdateSecretChat.swift#L97-L117)
- \[23\] [fork secret chat padding](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/SecretChats/SecretChatEncryption.swift#L286-L310)
- \[24\] [fork PFS operations](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/ManagedSecretChatOutgoingOperations.swift#L146-L155)
- \[25\] [fork device encryption key file](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/BuildConfig/Sources/BuildConfig.m#L361-L381)
- \[26\] [upstream locked atomic key file](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/submodules/BuildConfig/Sources/BuildConfig.m#L380-L434)
- \[27\] [macOS secret key view](https://github.com/overtake/TelegramSwift/blob/579cebbf0c01fd41b712eff3647fa7f69db9665d/Telegram-Mac/SecretChatKeyViewController.swift#L38-L123)
- \[28\] [fork notification account setup](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/Telegram/NotificationService/Sources/NotificationService.swift#L696-L718)
- \[29\] [fork notification key and payload decryption](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/Account/Account.swift#L786-L889)
- \[30\] [fork transaction modes](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/Postbox/Sources/SqliteValueBox.swift#L528-L550)
- \[31\] [fork VoIP push CallKit handoff](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramUI/Sources/AppDelegate.swift#L2160-L2243)
- \[32\] [fork notification media timeouts](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/Telegram/NotificationService/Sources/NotificationService.swift#L1534-L1645)
- \[33\] [upstream notification poll budgets](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/Telegram/NotificationService/Sources/NotificationService.swift#L1521-L1543)
- \[34\] [fork call state](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/CallSessionManager.swift#L81-L110)
- \[35\] [fork call versions and signaling](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/CallSessionManager.swift#L509-L555)
- \[36\] [fork conference cleanup](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/ConferenceCallE2EContext.swift#L328-L417)
- \[37\] [macOS active call context](https://github.com/overtake/TelegramSwift/blob/579cebbf0c01fd41b712eff3647fa7f69db9665d/Telegram-Mac/PCallSession.swift#L1014-L1027)
- \[38\] [fork call duration](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/CallSessionManager.swift#L841-L855)
- \[39\] [upstream call duration fix](https://github.com/TelegramMessenger/Telegram-iOS/blob/f1dd7a2dbd02cbbf513e75d5695d8d36d1cf5838/submodules/TelegramCore/Sources/State/CallSessionManager.swift#L848-L858)
- \[40\] [macOS fork core dependencies](https://github.com/overtake/TelegramSwift/blob/579cebbf0c01fd41b712eff3647fa7f69db9665d/packages/PrivateCallScreen/Package.swift#L18-L37)
- \[41\] [fork account record](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/AccountManager/AccountRecord.swift#L28-L71)
- \[42\] [fork account record deletion](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/AccountManager/AccountManagerRecordTable.swift#L42-L51)
- \[43\] [fork account manager transaction](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/AccountManager/AccountManagerImpl.swift#L174-L266)
- \[44\] [fork account factory](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/Account/Account.swift#L251-L365)
- \[45\] [fork authorization send code](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/Authorization.swift#L174-L241)
- \[46\] [fork app config polling](https://github.com/overtake/Telegram-iOS/blob/a24bbe45f9861f736a79916a50898512f751e0dc/submodules/TelegramCore/Sources/State/ManagedAppConfigurationUpdates.swift#L7-L54)
- \[47\] [macOS auth state view](https://github.com/overtake/TelegramSwift/blob/579cebbf0c01fd41b712eff3647fa7f69db9665d/Telegram-Mac/AuthController.swift#L247-L266)
- \[48\] [macOS group container migration](https://github.com/overtake/TelegramSwift/blob/579cebbf0c01fd41b712eff3647fa7f69db9665d/packages/ApiCredentials/Sources/ApiCredentials/Config.swift#L23-L57)
- \[49\] [macOS account switch](https://github.com/overtake/TelegramSwift/blob/579cebbf0c01fd41b712eff3647fa7f69db9665d/Telegram-Mac/SharedAccountContext.swift#L634-L668)
- \[50\] [macOS notification authorization and cleanup](https://github.com/overtake/TelegramSwift/blob/579cebbf0c01fd41b712eff3647fa7f69db9665d/Telegram-Mac/UNUserNotifications.swift#L340-L364)
- \[51\] [macOS Sparkle updater wrapper](https://github.com/overtake/TelegramSwift/blob/579cebbf0c01fd41b712eff3647fa7f69db9665d/Telegram-Mac/AppUpdateViewController.swift#L21-L45)
- \[52\] [macOS crash marker](https://github.com/overtake/TelegramSwift/blob/579cebbf0c01fd41b712eff3647fa7f69db9665d/packages/CrashHandler/Sources/CrashHandler/CrashHandler.swift#L11-L34)
