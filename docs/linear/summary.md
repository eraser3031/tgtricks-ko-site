# 요약

웹: /docs/linear/summary/ · Markdown: /docs/linear/summary.md

> 긴 본문을 10분 안에 읽을 수 있게 핵심만 추린 요약이에요.

> 이 문서는 Wenzhao Hu의 [Reverse Engineering Linear's Sync Engine](https://github.com/wzhudev/reverse-linear-sync-engine/blob/45dc91e78e03300b69cc088057a28c42cee35a41/SUMMARY.md)을 한국어로 옮기고 말투를 다듬은 번역이에요. 원문과 번역 모두 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.ko) 라이선스를 따라요.

이 요약은 Linear 동기화 엔진(LSE)을 리버스 엔지니어링한 연구를 정리한 글이에요. LSE가 어떻게 동작하는지 개념 위주로 훑어볼 수 있고, 소스 코드를 이해하기 위한 출발점으로 삼기 좋아요.

## 들어가며 [#introduction]

LSE의 핵심 개념은 이래요.

![introduction to LSE](https://github.com/wzhudev/reverse-linear-sync-engine/raw/45dc91e78e03300b69cc088057a28c42cee35a41/imgs/introduction.png)

**모델**

`Issue`, `Team`, `Organization`, `Comment` 같은 엔티티를 LSE에서는 **모델**이라고 불러요. 모델은 **프로퍼티**와 다른 모델에 대한 **참조**를 가져요. 이 중 상당수는 (**MobX**로) 관찰할 수 있어서, 값이 바뀌면 뷰가 자동으로 업데이트돼요. 한마디로 모델과 프로퍼티에는 LSE 안에서 어떻게 동작할지를 정하는 **메타데이터**가 들어 있어요.

모델은 **로컬 데이터베이스**(IndexedDB)나 서버에서 불러올 수 있어요. 일부 모델은 **부분 로딩**을 지원해서, 필요할 때 로컬 데이터베이스에서 불러오거나 서버에서 데이터를 더 가져올 수 있어요. 불러온 모델은 **오브젝트 풀**에 저장해요. 오브젝트 풀은 **UUID**로 모델을 찾는 커다란 맵이에요.

모델은 지연 **하이드레이션**할 수 있어요. 프로퍼티에 접근할 때 비로소 프로퍼티를 불러온다는 뜻이에요. 꼭 필요한 데이터만 불러오기 때문에 성능을 높이는 데 특히 유용해요.

모델, 프로퍼티, 참조에 하는 추가, 삭제, 업데이트, 보관 같은 작업은 **트랜잭션**으로 묶어요. 트랜잭션은 서버로 보내 서버에서 실행하고, 그 뒤 **델타 패킷**으로 연결된 모든 클라이언트에 브로드캐스트해요. 이렇게 하면 여러 클라이언트의 데이터가 일관되게 유지돼요.

**트랜잭션**

서버로 보내는 작업은 **트랜잭션**으로 포장해요. 트랜잭션은 서버에서만 **독점적으로** 실행하도록 만들었고, 실패하면 클라이언트에서 **되돌릴 수 있게** 설계했어요. 클라이언트가 서버와 연결이 끊기면 트랜잭션을 IndexedDB에 임시로 **캐시**해 두었다가, 연결이 복구되면 자동으로 다시 보내요.

트랜잭션에는 **싱크 ID**가 붙어요. 싱크 ID는 계속 커지기만 하는 숫자라서 작업 순서를 정확하게 지켜 줘요. 이 숫자는 모든 클라이언트의 일관성을 유지하는 데 중요해요.

트랜잭션은 **실행 취소**와 **다시 실행**을 지원하는 데도 핵심 역할을 해요. 덕분에 실시간 협업 중에도 변경과 수정을 매끄럽게 할 수 있어요.

**델타 패킷**

서버는 트랜잭션을 실행한 뒤, 트랜잭션을 보낸 클라이언트를 포함해 모든 클라이언트에 **델타 패킷**을 브로드캐스트해서 모델을 업데이트해요. 델타 패킷에는 **싱크 액션**이 여러 개 들어 있고, 각 액션에도 **싱크 ID**가 붙어요. 이 방식 덕분에 클라이언트가 업데이트를 놓치지 않고, 어긋남이 생겼을 때 빠진 패킷을 찾을 수 있어요.

서버가 실행하는 동안 **부수 효과**(예: 히스토리 생성)를 일으킬 수 있어서, **델타 패킷**은 클라이언트가 보낸 원래 트랜잭션과 다를 수 있어요.

## 모델과 메타데이터 정의 [#defining-models-and-metadata]

Linear가 시작하면 먼저 모델의 메타데이터를 만들어요. 여기에는 모델의 프로퍼티, 메서드(액션), 계산된 값이 들어 있어요. LSE는 이 메타데이터를 관리하려고 `ModelRegistry`라는 상세한 사전을 유지해요.

![model registry](https://github.com/wzhudev/reverse-linear-sync-engine/raw/45dc91e78e03300b69cc088057a28c42cee35a41/imgs/model-registry.png)

LSE는 **데코레이터**로 모델과 프로퍼티를 정의하고, 그 메타데이터를 `ModelRegistry`에 기록해요.

모델의 메타데이터는 이래요.

1. **`loadStrategy`**: 모델을 클라이언트에 불러오는 방식을 정해요. 전략은 다섯 가지예요.
   - **`instant`**: 애플리케이션 부트스트래핑 중에 불러오는 모델이에요(기본 전략).
   - **`lazy`**: 부트스트래핑 중에는 불러오지 않고, 필요할 때 한꺼번에 가져오는 모델이에요(예: `ExternalUser`).
   - **`partial`**: 필요할 때 불러오는 모델이에요. 서버에서 인스턴스 일부만 가져와요(예: `DocumentContent`).
   - **`explicitlyRequested`**: 명시적으로 요청할 때만 불러오는 모델이에요(예: `DocumentContentHistory`).
   - **`local`**: 로컬 데이터베이스에만 저장하는 모델이에요. 이 전략을 쓰는 모델은 아직 찾지 못했어요.
2. **`partialLoadMode`**: 모델을 어떻게 하이드레이션할지 정해요. 값은 `full`, `regular`, `lowPriority` 세 가지예요.
3. **`usedForPartialIndexes`**: 부분 인덱스 기능과 관련 있어요.

프로퍼티의 메타데이터는 이래요.

1. `type`: 프로퍼티의 타입을 정해요.
2. `lazy`: 모델을 하이드레이션할 때만 프로퍼티를 불러올지 정해요.
3. `serializer`: 데이터를 전송하거나 저장할 때 프로퍼티를 어떻게 직렬화할지 정해요.
4. `indexed`: 프로퍼티를 데이터베이스에서 인덱싱할지 정해요. 참조에 써요.
5. `nullable`: 프로퍼티가 `null`일 수 있는지 정해요. 참조에 써요.
6. 그 밖의 항목들

`type`은 다음 값을 가진 열거형이에요.

1. **`property`**: 모델이 "소유"한 프로퍼티예요. 예를 들어 `title`은 `Issue`의 `property`예요.
2. **`ephemeralProperty`**: `property`와 비슷하지만 데이터베이스에 저장하지 않아요. 거의 쓰지 않는 타입이에요. 예를 들어 `lastUserInteraction`은 `User`의 ephemeral property예요.
3. **`reference`**: 모델이 다른 모델을 참조할 때 쓰는 프로퍼티예요. 값은 보통 참조 대상 모델의 ID예요. 참조는 지연 로딩할 수 있어서, 이 프로퍼티에 접근하기 전까지는 참조 대상 모델을 불러오지 않아요. 예를 들어 `subscription`은 `Team`의 `reference`예요.
4. **`referenceModel`**: `reference` 프로퍼티를 등록하면 `referenceModel` 프로퍼티도 함께 만들어요. 이 프로퍼티는 getter와 setter를 정의해서, 대응하는 `reference`로 참조 대상 모델에 접근하게 해 줘요.
5. **`referenceCollection`**: `reference`와 비슷하지만 모델 배열을 참조해요. 예를 들어 `templates`는 `Team`의 `referenceCollection`이에요.
6. **`backReference`**: `backReference`는 `reference`의 반대 방향이에요. 예를 들어 `favorite`은 `Issue`의 `backReference`예요. 가장 큰 차이는 `backReference`를 참조 대상 모델이 "소유"한다고 보는 점이에요. 참조 대상 모델(B)을 삭제하면 `backReference`(A)도 함께 삭제돼요.
7. **`referenceArray`**: 다대다 관계에 써요. 예를 들어 `Project`의 `members`는 `Users`를 참조하는 `referenceArray`라서, 사용자가 여러 프로젝트의 멤버가 될 수 있어요.

LSE는 여러 데코레이터로 다양한 타입의 프로퍼티를 등록해요. 이 장에서는 그중 세 가지를 먼저 살펴봐요.

### 스키마 해시 [#schema-hash]

`ModelRegistry`에는 **`__schemaHash`**라는 특별한 프로퍼티가 있어요. 모든 모델의 메타데이터와 프로퍼티의 메타데이터를 해시한 값이에요. 로컬 데이터베이스를 마이그레이션해야 하는지 판단하는 데 이 해시가 꼭 필요해요.

## 부트스트래핑 [#bootstrapping]

Linear의 전체 부트스트래핑은 이런 모습이에요.

![bootstrapping overview](https://github.com/wzhudev/reverse-linear-sync-engine/raw/45dc91e78e03300b69cc088057a28c42cee35a41/imgs/bootstrapping-overview.png)

1. `StoreManager`(`cce`)가 모델마다 `PartialStore`(`jm`)나 `FullStore`(`TE`)를 만들어요. 이 스토어들은 메모리 데이터와 IndexedDB를 동기화해요. 싱크 액션을 저장하는 `SyncActionStore`(`oce`)도 만들어요.
2. `Database`(`eg`)가 IndexedDB에 연결해서 데이터베이스와 테이블을 준비해요. 데이터베이스가 없으면 만들고, 마이그레이션이 필요하면 실행해요.
3. `Database`가 어떤 종류의 부트스트래핑을 할지 정해요.
4. 알맞은 부트스트래핑을 실행해요. 전체 부트스트래핑에서는 서버에서 모델을 가져와요.
5. 가져온 모델 데이터를 IndexedDB에 저장해요.
6. 바로 하이드레이션해야 하는 데이터를 메모리에 불러오고, 관찰 가능성을 켜요.
7. 델타 패킷을 받을 수 있게 서버와 연결을 맺어요.

Linear는 로그인한 워크스페이스마다 데이터베이스를 하나씩 만들어요. 이 데이터베이스의 메타데이터에는 다음 필드가 있어요.

1. `lastSyncId`. 아래 절에서 설명해요.
2. **`firstSyncId`**: 클라이언트가 **전체 부트스트래핑**을 할 때의 `lastSyncId` 값이에요. 뒤에서 보겠지만, 이 값으로 증분 동기화를 어디서 시작할지 정해요.
3. **`subscribedSyncGroups`**. 아래 절에서 설명해요.
4. 그 밖의 필드들

전체 부트스트래핑을 하면 응답에 이 메타데이터가 들어 있고, LSE가 이를 데이터베이스에 저장해요.

### `lastSyncId` [#lastsyncid]

**`lastSyncId`**는 LSE의 핵심 개념이에요. 이 개념은 트랜잭션, 델타 패킷과 이어져 있어요. 이어지는 장에서 더 자세히 다룰 거예요. 지금 이 부분을 다 이해하지 못해도 괜찮아요. 계속 읽다가, 뒤 장을 읽은 다음 이 절로 돌아오면 모든 게 하나로 맞춰질 거예요.

`lastSyncId`는 트랜잭션에 순서를 매기는 번호표 역할을 해요. 먼저 Linear의 협업 방식부터 짚어 볼게요.

Linear는 [로컬 퍼스트 소프트웨어](https://www.inkandswitch.com/local-first/)의 기준으로 자주 꼽혀요. CRDT를 쓰는 대부분의 주류 로컬 퍼스트 애플리케이션과 달리, Linear의 협업 모델은 OT에 더 가까워요. 중앙 서버가 모든 트랜잭션의 순서를 정하기 때문이에요.

- LSE에서는 클라이언트가 보낸 모든 트랜잭션이 [전순서](https://en.wikipedia.org/wiki/Total_order)를 따라요.
- 반면 CRDT는 보통 [부분 순서](https://en.wikipedia.org/wiki/Partially_ordered_set)만 있으면 돼요.

이 전순서를 나타내는 것이 `sync id`예요. `sync id`는 계속 커지는 정수이고, 이름에서 알 수 있듯 `lastSyncId`는 가장 최근의 `sync id`예요.

서버가 트랜잭션을 성공적으로 실행하면 전역 **`lastSyncId`**가 1 늘어요. 이 ID는 사실상 **데이터베이스의 버전 번호** 역할을 해서, 모든 변경을 순서대로 추적할 수 있게 해 줘요.

![](https://github.com/wzhudev/reverse-linear-sync-engine/raw/45dc91e78e03300b69cc088057a28c42cee35a41/imgs/lastsyncid.png)

LSE는 트랜잭션 순서를 정하는 중앙 서버가 필요하기 때문에, 어느 정도는 CRDT(Conflict-Free Replicated Data Types, 충돌 없이 복제되는 데이터 타입)보다 OT(Operational Transformation, 작업을 변환해 충돌을 맞추는 방식)에 가까워요.

### SyncGroup [#syncgroup]

이 개념은 LSE에서 중요해요. 모든 워크스페이스가 같은 `lastSyncId` 카운터를 쓰지만, 권한이 없는 워크스페이스나 팀의 이슈에는 접근할 수 없고 델타 패킷도 받을 수 없어요. 이 제한은 접근 제어 메커니즘이 적용하고, 핵심 요소는 `subscribedSyncGroups`예요. `subscribedSyncGroups` 배열에는 내 사용자 ID, 내가 속한 팀, 미리 정해진 역할을 나타내는 UUID가 들어 있어요.

### 지연 하이드레이션 [#lazy-hydration]

Linear는 전체 부트스트래핑 중에 서버에서 모든 것을 한 번에 불러오지 않고, 매번 모든 것을 메모리에 올리지도 않아요. 대신 **지연 하이드레이션**을 지원해서, 필요할 때 필요한 데이터만 메모리에 불러와요. 이렇게 하면 성능이 좋아지고 메모리 사용량이 줄어요.  

`hydrate` 메서드가 있는 클래스는 하이드레이션할 수 있어요. `Model`, `LazyReferenceCollection`, `LazyReference`, `RequestCollection`, `LazyBackReference`가 그 예예요.  

LSE는 지연 모델을 불러올 때 **부분 인덱스**와 **싱크 그룹** 등 여러 방식을 키로 써요.  

## 동기화 [#syncing]

LSE 클라이언트는 모델에 작업을 하려고 트랜잭션을 서버로 보내요. 아래에서는 `UpdatingTransactions`를 예로 들어 LSE의 트랜잭션이 어떻게 동작하는지 간단히 살펴봐요.

![](https://github.com/wzhudev/reverse-linear-sync-engine/raw/45dc91e78e03300b69cc088057a28c42cee35a41/imgs/transaction-overview.png)

1. 프로퍼티에 새 값을 대입하면, 시스템이 바뀐 프로퍼티의 이름과 이전 값 같은 핵심 정보를 기록해요. **메모리에 있는 모델**은 이 변경을 반영해 **즉시** 업데이트돼요.
2. `issue.save()`를 호출하면 **`UpdateTransaction`**을 만들어요. 이 트랜잭션에 모델의 변경 내용이 담겨요.
3. 만들어진 `UpdateTransaction`을 요청 큐에 넣어요. 동시에 **캐시**를 위해 IndexedDB의 `__transactions` 테이블에도 저장해요.
4. `TransactionQueue`가 타이머를 예약해서(바로 실행하기도 해요) 큐에 쌓인 트랜잭션을 **배치**로 서버에 보내요.
5. 백엔드가 배치를 성공적으로 처리하면, 그 배치를 IndexedDB의 `__transactions` 테이블에서 지워요. 그러면 Local Storage Engine(LSE)이 캐시된 배치를 정리해요.
6. 트랜잭션은 `lastSyncId`가 담긴 델타 패킷이 도착해야 완료되고, 그때까지 기다려요.

트랜잭션의 핵심 기능은 이래요.

1. **캐싱** – 클라이언트의 연결이 끊기거나 종료돼도, 다시 연결할 때 트랜잭션을 다시 보낼 수 있어요.
2. **실행 취소와 다시 실행** – 클라이언트에서 트랜잭션을 실행 취소, 다시 실행, 되돌릴 수 있어서 서버가 거부해도 매끄럽게 처리해요.
3. **충돌 해결** – **last-writer-wins** 전략으로 충돌을 해결해요.

LSE는 델타 패킷을 받으려고 서버와 WebSocket 연결을 만들고, 델타 패킷을 받으면 다음 작업을 해요.

1. 사용자가 싱크 그룹에 추가되거나 제거됐는지 확인해요.
2. 특정 액션이 의존하는 대상을 불러와요.
3. 새 싱크 그룹과 그에 딸린 데이터를 로컬 데이터베이스에 써요.
4. 모든 싱크 액션을 순회하면서 해석해 로컬 데이터베이스를 업데이트해요.
5. 모든 싱크 액션을 다시 순회하면서 메모리 데이터를 업데이트해요.
6. 클라이언트의 `lastSyncId`를 업데이트하고, 싱크 그룹이 바뀌면 `firstSyncId`도 업데이트해요.
7. `lastSyncId`를 기다리던 완료된 트랜잭션을 마무리해요.
