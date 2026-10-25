# 2장: 부트스트래핑과 지연 로딩

웹: /docs/linear/02-bootstrapping/ · Markdown: /docs/linear/02-bootstrapping.md

> IndexedDB에 로컬 데이터베이스를 만들고, 부트스트래핑 방식을 고르고, 모델을 지연 하이드레이션하는 과정을 따라가요.

> 이 문서는 Wenzhao Hu의 [Reverse Engineering Linear's Sync Engine](https://github.com/wzhudev/reverse-linear-sync-engine/blob/45dc91e78e03300b69cc088057a28c42cee35a41/README.md)을 한국어로 옮기고 말투를 다듬은 번역이에요. 원문과 번역 모두 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.ko) 라이선스를 따라요.

모델을 정의했으니 이제 **모델을 클라이언트에 불러올** 차례예요. 이 장에서는 LSE가 모델을 **부트스트래핑하고 지연 로딩하는 방법**을 알아봐요.

먼저 전체 흐름을 큰 그림으로 훑어서 기본 이해를 잡고, 그다음에 세부 내용으로 들어가요. 이 과정에는 여러 모듈이 얽혀 있어서, 각 모듈도 짧게 소개할게요.

![부트스트래핑 개요](https://github.com/wzhudev/reverse-linear-sync-engine/raw/45dc91e78e03300b69cc088057a28c42cee35a41/imgs/bootstrapping-overview.png)

1. `StoreManager`(`cce`)가 모델마다 `PartialStore`(`jm`) 또는 `FullStore`(`TE`)를 만들어요. 이 스토어들은 메모리의 데이터를 IndexedDB와 동기화해요. 동기화 액션을 저장하는 `SyncActionStore`(`oce`)도 함께 만들어요.
2. `Database`(`eg`)가 IndexedDB에 연결하고 데이터베이스와 테이블을 준비해요. 데이터베이스가 없으면 새로 만들고, 마이그레이션이 필요하면 마이그레이션도 해요.
3. `Database`가 어떤 종류의 부트스트래핑을 할지 정해요.
4. 알맞은 부트스트래핑을 실행해요. 전체 부트스트래핑이면 서버에서 모델을 가져와요.
5. 가져온 모델 데이터를 IndexedDB에 저장해요.
6. 바로 하이드레이션(저장된 데이터로 모델 객체를 메모리에 만드는 일)해야 하는 데이터를 메모리에 불러오고, 관찰 가능성(observability)을 켜요.
7. 델타 패킷을 받기 위해 서버와 연결을 만들어요.

LSE의 부트스트래핑은 **전체 부트스트래핑**, **부분 부트스트래핑**, **로컬 부트스트래핑**, 이렇게 세 가지예요. 이 글에서는 **전체 부트스트래핑**을 자세히 설명해요.

## `ObjectStore` 만들기 [#create-objectstores]

> [!NOTE]
> 코드 참조
>
> - `cce`: `StoreManager`
> - `p3`: `PartialStore`
> - `TE`: `FullStore`
>
> 부트스트래핑은 `km.startBootstrap`(`SyncedStore.startBootstrap`)에서 시작해요. `StoreManager`는 getter인 `eg.storeManager`(`Database.storeManager`)를 통해 처음 필요할 때 만들어져요.

부트스트래핑의 첫 단계는 `StoreManager`를 만드는 일이에요. 이 모듈은 `ModelRegistry`에 등록된 모델마다 `ObjectStore` 인스턴스를 만들고 관리해요. 각 `ObjectStore`는 IndexedDB에서 그 모델에 해당하는 테이블을 맡아요.

![object-stores.png](https://github.com/wzhudev/reverse-linear-sync-engine/raw/45dc91e78e03300b69cc088057a28c42cee35a41/imgs/object-stores.png)

_모델이 80종류라서 `ObjectStore`도 80개예요._

앞서 말했듯이 모델에는 `loadStrategy` 메타데이터가 있고, LSE는 이 필드를 보고 알맞은 스토어 타입을 만들어요. 로드 전략이 `partial`인 모델은 `PartialObjectStore`(`p3`, `Jm`)가 관리하고, 나머지 모델은 모두 `FullObjectStore`(`TE`)를 써요.

`ObjectStore`는 만들어질 때 연결된 모델의 해시를 계산해요. 이 해시가 데이터베이스에서 테이블 이름이 돼요. 예를 들어 `Issue` 모델의 `storeName`은 `119b2a...`이고, 같은 해시 이름의 테이블이 이에 해당해요.

![](https://github.com/wzhudev/reverse-linear-sync-engine/raw/45dc91e78e03300b69cc088057a28c42cee35a41/imgs/model-store-class.png)

![](https://github.com/wzhudev/reverse-linear-sync-engine/raw/45dc91e78e03300b69cc088057a28c42cee35a41/imgs/model-store-db.png)

`loadStrategy`가 `partial`인 모델은 `<hash>_partial`이라는 데이터베이스가 하나 더 만들어져요. 이 데이터베이스에는 해당 모델을 지연 로딩할 때 쓰는 인덱스가 저장돼요. 이 인덱스를 쓰는 지연 로딩 메커니즘은 이 장의 마지막에서 자세히 살펴봐요.

![](https://github.com/wzhudev/reverse-linear-sync-engine/raw/45dc91e78e03300b69cc088057a28c42cee35a41/imgs/model-partial-store-db.png)

## IndexedDB에 데이터베이스와 테이블 만들기 [#create-databases--tables-in-indexeddb]

> [!NOTE] 
> 코드 참조
>
> - `eg.open`: `Database.open`
> - `jn` 또는 `Xn`: `DatabaseManager` - `databaseInfo`, `registerDatabase`, `database`
> - `cce.checkReadinessOfStores`: `StoreManager.checkReadinessOfStores`
> - `TE.checkIsReady`: `SyncStore.checkIsReady`

`ObjectStore`를 만들고 나면 다음은 데이터베이스를 준비하는 단계예요. IndexedDB에 데이터베이스와 테이블이 없으면 새로 만들어요.

LSE는 IndexedDB에 두 종류의 데이터베이스를 둬요. `linear_databases`와 `linear_(hash)` 같은 이름의 데이터베이스예요.

**`linear_databases`**: 다른 데이터베이스의 정보를 저장하는 데이터베이스예요. LSE는 워크스페이스에 로그인한 사용자마다 데이터베이스를 따로 만들어요. 사용자가 여러 워크스페이스에 속해 있으면, 로그인한 워크스페이스마다 데이터베이스를 하나씩 만들어요.

![linear databases](https://github.com/wzhudev/reverse-linear-sync-engine/raw/45dc91e78e03300b69cc088057a28c42cee35a41/imgs/linear-databases.png)

데이터베이스 정보에는 다음 항목이 들어 있어요.

1. **`name`**: 데이터베이스 이름이에요. `userId`, `version`, `userVersion`으로 계산해요. 그래서 사용자 신원이 다르면 데이터베이스도 여러 개가 생겨요.
2. **`schemaHash`**: 데이터베이스 마이그레이션에 써요. `ModelRegistry`의 `_schemaHash` 프로퍼티에 해당해요.
3. **`schemaVersion`**: 데이터베이스 마이그레이션이 필요한지 판단하는 증가 카운터예요. 새 `schemaHash`가 IndexedDB에 저장된 값과 다르면 카운터가 올라가요. 올라간 버전은 [`IndexedDB.open`](http://indexdb.open/)의 두 번째 인자로 넘겨서 마이그레이션이 필요한지 확인해요.

이 밖에도 항목이 더 있어요. 이 정보를 어떻게 계산하는지는 `jn.databaseInfo`에서 확인할 수 있어요.

**`linear_(hash)`**: 워크스페이스 하나의 데이터가 들어 있는 데이터베이스예요. 예를 들어 `linear_b4782b3125a816b51a44e59f2e939efa`에는 제 개인 워크스페이스의 데이터가 저장돼 있어요.

이 데이터베이스 안에는 앞 절에서 이야기한 대로 모델마다 테이블이 있어요. 여기에 특별한 테이블이 두 개 더 있어요.

첫 번째는 **`_meta`** 테이블이에요. 모델별 영속화 정보와 데이터베이스 자체의 메타데이터를 담아요.

![](https://github.com/wzhudev/reverse-linear-sync-engine/raw/45dc91e78e03300b69cc088057a28c42cee35a41/imgs/meta-persistence.png)

_모델의 영속화 상태_

모델마다 `_meta` 테이블에 레코드가 하나씩 있어요. `persisted` 필드가 `true`이면, 워크스페이스에 있는 그 모델의 인스턴스를 모두 클라이언트에 불러왔다는 뜻이에요.

![](https://github.com/wzhudev/reverse-linear-sync-engine/raw/45dc91e78e03300b69cc088057a28c42cee35a41/imgs/meta-meta.png)

_데이터베이스의 메타데이터_

데이터베이스 메타데이터에는 다음 필드가 있어요.

1. `lastSyncId`.

---

**`lastSyncId`**는 LSE의 핵심 개념이라서 여기서 먼저 소개할게요. 트랜잭션(클라이언트가 보내고 서버가 실행하는 변경), 델타 패킷(서버가 보내는 증분 변경)과 이어지는 개념인데, 이 부분은 뒤 장에서 더 자세히 다뤄요. 지금 다 이해하지 못해도 괜찮아요. 계속 읽다가 뒤 장을 마친 뒤에 이 절로 돌아오면 모든 게 맞아떨어질 거예요.

Linear는 [로컬 퍼스트 소프트웨어](https://www.inkandswitch.com/local-first/)의 기준으로 자주 꼽혀요. CRDT(서버 없이도 충돌 없이 병합되는 자료 구조)를 쓰는 대부분의 주류 로컬 퍼스트 앱과 달리, Linear의 협업 모델은 OT(operational transformation)에 더 가까워요. 중앙 서버가 모든 트랜잭션의 순서를 정하기 때문이에요.

LSE에서는 클라이언트가 보내는 모든 트랜잭션이 [전순서](https://en.wikipedia.org/wiki/Total_order)(total order, 모든 항목의 앞뒤가 정해진 순서)를 따라요. CRDT는 보통 [부분 순서](https://en.wikipedia.org/wiki/Partially_ordered_set)(partial order)만 있으면 돼요. 이 전순서를 나타내는 값이 `sync id`예요. `sync id`는 계속 늘어나는 정수고, 이름 그대로 `lastSyncId`는 가장 최근의 `sync id`예요.

서버가 트랜잭션을 성공적으로 실행할 때마다 전역 **`lastSyncId`**가 1씩 올라가요. 이 ID는 사실상 **데이터베이스의 버전 번호** 역할을 해서, 모든 변경을 순서대로 추적하게 해줘요.

![](https://github.com/wzhudev/reverse-linear-sync-engine/raw/45dc91e78e03300b69cc088057a28c42cee35a41/imgs/lastsyncid.png)

__데이터베이스가 바뀔 때마다 `lastSyncId`가 1씩 올라가요. `lastSyncId`는 트랜잭션, 델타 패킷과도 연결돼요.__

서버는 트랜잭션을 보낸 클라이언트에 응답할 때 갱신된 `lastSyncId`를 함께 보내요. 또 서버가 모든 클라이언트에 델타 패킷을 브로드캐스트할 때도, 이 패킷마다 해당 `lastSyncId`가 붙어요. 덕분에 클라이언트는 최신 데이터베이스 버전을 기준으로 로컬 상태를 서버와 맞출 수 있어요.

`sync id`는 OT 알고리즘의 **파일 리비전 번호**와 비슷한 개념이에요. (자세한 내용은 [OT를 다룬 제 글](https://wzhu.dev/posts/ot)에서 볼 수 있어요.) 다만 파일 리비전 번호는 보통 파일 하나에만 적용되지만, **`lastSyncId`는 어느 워크스페이스에서 일어난 변경이든 상관없이 데이터베이스 전체에 걸쳐 있어요**.

`lastSyncId`가 이렇게 데이터베이스 전체에 걸쳐 있다는 건 실제로도 확인할 수 있어요. 내 워크스페이스에서 트랜잭션이 한 번만 일어나도 `lastSyncId`가 크게 올라가는 일이 자주 있는데, 시스템 안의 모든 워크스페이스에서 일어난 변경을 추적하기 때문이에요.

클라이언트는 **`lastSyncId`**로 서버와 동기화된 상태인지 판단해요. 로컬 `lastSyncId`와 서버가 준 `lastSyncId`를 비교하면, 빠뜨린 트랜잭션이 있는지 알 수 있어요.

- 클라이언트의 `lastSyncId`가 서버가 준 값보다 **작으면**, 클라이언트가 동기화되지 않은 상태이고 일부 델타 패킷을 받지 못했다는 뜻이에요.
- 서버는 클라이언트가 최신 상태를 유지하도록 응답에 `lastSyncId`를 자주 넣어 보내요.

클라이언트의 `lastSyncId`는 처음에 **전체 부트스트래핑** 과정에서 정해져요. 이때 데이터베이스의 최신 상태를 가져와요. 이후 서버에서 **델타 패킷**을 받을 때마다 `lastSyncId`가 새로 동기화된 상태에 맞게 갱신돼요.

이제 데이터베이스 메타데이터의 나머지 필드로 돌아가요.

---

2. **`firstSyncId`**: 클라이언트가 **전체 부트스트래핑**을 할 때의 `lastSyncId` 값이에요. 뒤에서 보겠지만, 이 값은 증분 동기화를 어디서부터 시작할지 정하는 데 써요.
3. **`backendDatabaseVersion`**: 백엔드 데이터베이스의 버전이에요. 이름 그대로 클라이언트와 서버 데이터베이스가 호환되는지 추적하는 데 써요.
4. **`updatedAt`**: 데이터베이스나 그 메타데이터를 마지막으로 갱신한 시각의 타임스탬프예요. 이름 그대로예요.
5. **`subscribedSyncGroups`**.

---

`subscribedSyncGroups`는 LSE에서 아주 중요한 필드예요. 모든 워크스페이스가 같은 `lastSyncId` 카운터를 공유하지만, 권한이 없는 워크스페이스나 팀의 이슈는 볼 수 없고 그곳의 델타 패킷도 받을 수 없어요. 접근 제어 메커니즘이 이 제한을 강제하는데, 핵심 역할을 하는 것이 `subscribedSyncGroups`예요. 이 배열에는 내 사용자 ID, 내가 속한 팀, 미리 정의된 역할을 나타내는 UUID가 들어 있어요.

> [!NOTE]  
> 초기 버전의 LSE에서는 `SyncGroup`이 어떻게 동작하는지 이해하기가 특히 어려웠어요. 그런데 Linear 팀이 최근 업데이트에서 `userSyncGroup`과 `teamSyncGroup`을 도입하면서 그 목적이 분명해졌어요. 이 변경으로 `SyncGroup`이 특정 "User" 또는 "Team"에 연결된 모델의 모음이라는 사실이 드러났어요.

> [!NOTE] 
> Linear의 데이터베이스 메타데이터 변경  
> 2024년 말에 Linear가 데이터베이스 메타데이터 필드를 바꿨어요. 이 스크린샷은 바뀐 메타데이터를 보여주지만, 소스 코드 발췌는 변경 전에 가져온 거예요. 예를 들어 `subscribedSyncGroups`는 `userSyncGroups`로 바뀌었어요. 이 변경이 LSE가 동작하는 핵심 개념에는 큰 영향을 주지 않아서, 이 글에서는 그 차이를 다루지 않을게요.

---

여기까지가 `_meta` 테이블 설명이에요. 이제 두 번째 특별한 테이블인 **`_transaction`**을 살펴봐요. 이 테이블에는 아직 보내지 않은 트랜잭션이나 서버와 동기화하려고 큐에 쌓아둔 트랜잭션이 저장돼요. 트랜잭션은 다음 장에서 더 자세히 다뤄요.

![](https://github.com/wzhudev/reverse-linear-sync-engine/raw/45dc91e78e03300b69cc088057a28c42cee35a41/imgs/cached-transactions.png)

_캐시된 트랜잭션_

부트스트래핑 과정으로 돌아가서, IndexedDB에 이 두 종류의 데이터베이스가 어떻게 만들어지는지 알아봐요. 소스 코드와 주석은 `ng.initializeDatabase`(`SyncClient.initializeDatabase`)에서 볼 수 있어요.

**1단계: 워크스페이스 메타데이터 가져오기**

`Xn.databaseInfo` 메서드로, 부트스트래핑할 워크스페이스의 메타데이터를 가져오면서 시작해요. 이 단계에서 `linear_databases` 데이터베이스가 아직 없으면 초기화해요. 메타데이터를 성공적으로 가져오면 `Xn.registerDatabase` 메서드로 `linear_databases` 데이터베이스에 저장해요.

**2단계: 워크스페이스 전용 데이터베이스 만들기**

다음으로 LSE는 `linear_b4782b3125a816b51a44e59f2e939efa` 같은 워크스페이스 전용 데이터베이스를 준비해요. 먼저 데이터베이스에 연결해서 새로 만들거나 마이그레이션해야 하는지 확인해요. 만들거나 마이그레이션해야 하면, `StoreManager`가 `createStores` 메서드(`this.storeManager.createStores(i, l)`)를 호출해서 모델에 필요한 테이블을 초기화해요.

이 단계에서 LSE는 데이터베이스 메타데이터도 읽으려고 해요. 하지만 전체 부트스트래핑에서는 아직 저장된 메타데이터가 없어서, 모든 필드가 `0`이나 다른 기본값으로 초기화돼요.

**3단계: 스토어 준비 상태 확인**

마지막 단계에서는 스토어마다 준비 상태를 확인해요. 처음 로드할 때는 테이블이 모두 비어 있어서 준비된 스토어가 하나도 없어요.

여기까지 오면 LSE가 데이터베이스 준비를 끝내고, 서버에서 데이터를 불러올 준비를 마친 상태예요. 이 과정이 어떻게 돌아가는지 더 깊이 살펴봐요.

## 부트스트래핑 종류 정하기 [#determine-the-bootstrapping-type]

> [!NOTE]
> 코드 참조
>
> - `ng.bootstrap`: `SyncClient.bootstrap`
> - `eg.requiredBootstrap`: `Database.requiredBootstrap`

다음 단계에서는 부트스트래핑 종류를 정하고 실행해요.

`Database.requiredBootstrap` 메서드가 알맞은 부트스트래핑 종류를 정하고, 실행에 필요한 파라미터를 줘요. 이 메서드는 다음 필드를 가진 객체를 반환해요.

1. **`type`**: 실행할 부트스트래핑의 종류예요.

---

부트스트래핑에는 세 종류가 있어요.

1. **`full`**: LSE가 필요한 모델을 모두 서버에서 가져와요.
2. **`local`**: 로컬 데이터베이스에서 데이터를 불러오고, 증분 델타로 서버와 동기화해요.
3. **`partial`**: 로드 전략에 따라 일부 모델만 서버에서 가져와요.

**LSE는 다음 경우에 전체 부트스트래핑을 해요**(설명을 단순하게 하려고 데모 프로젝트 로직은 뺐어요).

1. **준비된 스토어가 없을 때**: 앞에서 본 것처럼 새로 만든 테이블은 아직 비어 있어서 쓸 수 없어요.
2. **`lastSyncId`가 정의되지 않았을 때**: 데이터베이스에 마지막 동기화 지점의 기록이 없다는 뜻이에요.
3. **모델이 오래됐을 때**: 클라이언트가 온라인인데 일부 모델이 오래됐으면, 전체 부트스트래핑으로 모든 데이터를 새로 가져와요.

이 글에서는 **전체 부트스트래핑** 과정에 집중해요.

---

`requiredBootstrap`이 반환하는 값에는 다음 필드도 있어요.

2. **`modelsToLoad`**: 로드 전략이 **instant**나 **lazy**인 모델의 이름이에요.
3. **`lastSyncId`**: 클라이언트가 지금 동기화돼 있는 데이터베이스 스냅숏(특정 시점의 데이터베이스 상태)을 나타내요. 전체 부트스트래핑에서는 아직 서버에서 받은 데이터가 없어서 이 값이 0이에요. 이때 서버가 최신 스냅숏을 돌려줘요.

## 데이터베이스 부트스트래핑 [#bootstrapping-the-database]

> [!NOTE] 
> 코드 참조
>
> 1. `ng.bootstrap`: `SyncClient.bootstrap`
> 2. `eg.bootstrap`: `Database.bootstrap`
> 3. `Xm.fullBootstrap`: `BootstrapHelper.fullBootstrap`
> 4. `sd.restModelsJsonStreamGen`: `GraphQLClient.restModelsJsonStreamGen`

LSE가 전체 부트스트래핑을 시작하면 `GraphQLClient.restModelsJsonStreamGen` 메서드로 요청을 보내요. 이 함수는 서버에서 모델을 가져오는 일을 맡고, 이 글의 나머지 부분에서도 여러 번 나와요.

요청은 이렇게 생겼어요.

```
https://client-api.linear.app/sync/bootstrap?type=full&onlyModels=WorkflowState,IssueDraft,Initiative,ProjectMilestone,ProjectStatus,TextDraft,ProjectUpdate,IssueLabel,ExternalUser,CustomView,ViewPreferences,Roadmap,RoadmapToProject,Facet,Project,Document,Organization,Template,Team,Cycle,Favorite,CalendarEvent,User,Company,IssueImport,IssueRelation,TeamKey,UserSettings,PushSubscription,Activity,ApiKey,EmailIntakeAddress,Emoji,EntityExternalLink,GitAutomationTargetBranch,GitAutomationState,Integration,IntegrationsSettings,IntegrationTemplate,NotificationSubscription,OauthClientApproval,Notification,OauthClient,OrganizationDomain,OrganizationInvite,ProjectLink,ProjectUpdateInteraction,InitiativeToProject,Subscription,TeamMembership,TimeSchedule,TriageResponsibility,Webhook,WorkflowCronJobDefinition,WorkflowDefinition,ProjectRelation,DiaryEntry,Reminder
```

파라미터는 두 개예요.

1. **`type`**: 여기서는 `"full"`이에요.
2. **`onlyModels`**: 불러올 모델 이름을 쉼표로 이은 목록이에요. `requiredBootstrap`이 반환한 `modelsToLoad`에 해당해요.

> [!NOTE] 
> Linear의 부트스트래핑 요청 변경
> 2024년 말에 적용한 최적화로, Linear 팀이 이 요청 하나를 여러 요청으로 나눴어요. 큰 워크스페이스에서 캐시 성능과 로딩 속도를 높이려는 거예요. 이 변경은 LSE가 동작하는 방식에 영향을 주지 않아서 여기서는 자세히 다루지 않아요. 더 알고 싶다면 브라우저의 디버그 도구를 열고 소스 코드에서 `splitToCacheableRequests`를 검색해 보세요.

응답 예시는 이렇게 생겼어요.

```jsx
{"id":"8ce3d5fe-07c2-481c-bb68-cd22dd94e7de","createdAt":"2024-07-03T11:37:04.865Z","updatedAt":"2024-07-03T11:37:04.865Z","userId":"4e8622c7-0a24-412d-bf38-156e073ab384","issueId":"01a3c1cf-7dd5-4a13-b3ab-a9d064a3e31c","events":[{"type":"issue_deleted","issueId":"01a3c1cf-7dd5-4a13-b3ab-a9d064a3e31c","issueTitle":"Load data from remote sync engine."}],"__class":"Activity"}
{"id":"ec9ec347-4f90-465c-b8bc-e41dae4e11f2","createdAt":"2024-07-03T11:37:06.944Z","updatedAt":"2024-07-03T11:37:06.944Z","userId":"4e8622c7-0a24-412d-bf38-156e073ab384","issueId":"39946254-511c-4226-914f-d1669c9e5914","events":[{"type":"issue_deleted","issueId":"39946254-511c-4226-914f-d1669c9e5914","issueTitle":"Reverse engineering Linear's Sync Engine"}],"__class":"Activity"}
// ... many lines omitted here
_metadata_={"method":"mongo","lastSyncId":2326713666,"subscribedSyncGroups":["89388c30-9823-4b14-8140-4e0650fbb9eb","4e8622c7-0a24-412d-bf38-156e073ab384","AD619ACC-AAAA-4D84-AD23-61DDCA8319A0","CDA201A7-AAAA-45C5-888B-3CE8B747D26B"],"databaseVersion":948,"returnedModelsCount":{"Activity":6,"Cycle":2,"DocumentContent":5,"Favorite":1,"GitAutomationState":3,"Integration":1,"Issue":3,"IssueLabel":4,"NotificationSubscription":2,"Organization":1,"Project":2,"ProjectStatus":5,"Team":1,"TeamKey":1,"TeamMembership":1,"User":1,"UserSettings":1,"WorkflowState":7,"Initiative":1,"SyncAction":0}}
```

응답은 JSON 객체의 스트림이에요. 마지막 줄을 뺀 각 줄이 모델 인스턴스 하나의 정보예요. 예를 들어 다음은 `Issue` 모델을 나타내는 객체예요.

```json
{
  "id": "556c8983-ca05-41a8-baa6-60b6e5d771c8",
  "createdAt": "2024-01-22T01:02:41.099Z",
  "updatedAt": "2024-05-16T08:23:31.724Z",
  "number": 1,
  "title": "Welcome to Linear ðŸ‘‹", // Text encoding issue. Here's actually an emoji.
  "priority": 1,
  "boardOrder": 0,
  "sortOrder": -84.71, // LSE uses fractional indexing for sorting
  "startedAt": "2024-05-16T08:16:57.239Z",
  "labelIds": ["30889eaf-fac5-4d4d-8085-a4c3bd80e588"],
  "teamId": "89388c30-9823-4b14-8140-4e0650fbb9eb",
  "projectId": "3e7ada3c-f833-4b9c-b325-6db37285fa11",
  "projectMilestoneId": "397b95c4-3ee2-47b0-bad1-d6b1c7003616",
  "subscriberIds": ["4e8622c7-0a24-412d-bf38-156e073ab384"],
  "previousIdentifiers": [],
  "assigneeId": "4e8622c7-0a24-412d-bf38-156e073ab384",
  "stateId": "030a7891-2ba5-4f5b-9597-b750950cd866",
  "reactionData": [],
  "__class": "Issue"
}
```

응답의 마지막 줄에는 이번 부트스트래핑 요청에 대한 메타데이터가 들어 있어요. 이 메타데이터의 일부 필드는 데이터베이스 메타데이터의 해당 필드를 갱신하는 데 써요.

```json
{
  "method": "mongo",
  "lastSyncId": 2326713666,
  "subscribedSyncGroups": [
    "89388c30-9823-4b14-8140-4e0650fbb9eb",
    "4e8622c7-0a24-412d-bf38-156e073ab384",
    "AD619ACC-AAAA-4D84-AD23-61DDCA8319A0",
    "CDA201A7-AAAA-45C5-888B-3CE8B747D26B"
  ],
  "databaseVersion": 948,
  "returnedModelsCount": {
    "Activity": 6,
    "Cycle": 2,
    "DocumentContent": 5,
    "Favorite": 1,
    "GitAutomationState": 3,
    "Integration": 1,
    "Issue": 3,
    "IssueLabel": 4,
    "NotificationSubscription": 2,
    "Organization": 1,
    "Project": 2,
    "ProjectStatus": 5,
    "Team": 1,
    "TeamKey": 1,
    "TeamMembership": 1,
    "User": 1,
    "UserSettings": 1,
    "WorkflowState": 7,
    "Initiative": 1,
    "SyncAction": 0
  }
}
```

주요 필드는 다음과 같아요.

- **`method`**: 결과를 어디서 가져왔는지 알려줘요. 여기서는 `"mongo"`인데, MongoDB 캐시에서 데이터를 가져왔다는 뜻이에요. Tuomas는 Linear 동기화 엔진의 확장을 다룬 발표에서 이 전략을 이야기했어요.
- **`lastSyncId`**: 이번 부트스트래핑 요청 뒤에 클라이언트가 맞춰지는 스냅숏이에요. 예를 들어 스냅숏 ID는 `2326713666`이에요.
- **`subscribedSyncGroups`**: 관련된 증분 변경을 받으려면 클라이언트가 구독해야 하는 싱크 그룹을 알려줘요.
- **`returnedModelsCount`**: 응답에 담긴 모델 수가 이 값과 맞는지 확인해서 요청이 올바른지 검증해요.

> [!NOTE]
> Linear의 부트스트래핑 요청 변경
> 
> 앞서 말한 최적화에서 Linear는 `subscribedSyncGroups`를 응답에서 빼고, `/sync/user_sync_groups`로 보내는 사전 요청으로 옮겼어요. 이제 `/sync/bootstrap` 요청에서는 싱크 그룹이 요청 파라미터에 들어가요. 그래서 부트스트래핑 요청을 나눌 수 있어요.

마지막으로 가져온 모델을 각각의 오브젝트 스토어에 쓰고, 변경 사항에 맞게 데이터베이스 메타데이터를 갱신해요.

## 하이드레이션과 오브젝트 풀 [#hydration-and-object-pool]

> [!NOTE] 
> 코드 참조
>
> 1. `ng.bootstrap`: `SyncClient.bootstrap`.
> 2. `eg.getAllInitialHydratedModelData`: `Database.getAllInitialHydratedModelData`.
> 3. `ng.addModelToLiveCollections`: `SyncClient.addModelToLiveCollections`.
> 4. `as.updateFromData`: `ClientModel.updateFromData`.
> 5. `as.updateReferencedModels`: `ClientModel.updateReferencedModels`.

**모델 하이드레이션**은 원본 모델을 메모리에 모델 객체로 만들어서 **오브젝트 풀**에 넣는 과정이에요. 그러면 애플리케이션의 다른 부분이 모델에 접근할 수 있어요. 원본 모델을 `ObjectStore`에 쓰고 나면 이 단계가 이어져요.

LSE는 `Database.getAllInitialHydratedModelData`를 호출해서 모델 하이드레이션을 시작해요. 이 단계에서 `Database`는 `loadStrategy`가 `instant`인 모델을 불러와요. 이런 모델마다 LSE가 `ModelRegistry`에서 생성자를 가져와 모델 객체를 만들고, 이 객체를 `addModelToLiveCollections` 메서드로 오브젝트 풀에 추가해요.

오브젝트 풀은 `SyncClient`의 `modelLookup`이라는 맵으로 구현돼 있어요. 이 맵은 모델 ID를 해당 모델 객체와 연결해서, Linear의 다른 부분이 ID로 모델을 빠르게 찾을 수 있게 해줘요.

모델 객체를 만들 때 LSE는 디하이드레이션된(객체가 되기 전의 원본) 모델 데이터를 생성자에 바로 넘기지 않아요. 객체를 먼저 초기화한 다음 `updateFromData` 메서드를 호출해서 데이터를 채우는 방식으로 하이드레이션해요.

또 `attachToReferencedProperties`를 호출해서 참조를 해석하고 채워요.

## 지연 하이드레이션 [#lazy-hydration]

> [!note] 
> 코드 참조
>
> - `as.hydrate`: `Model.hydrate`.
> - `Issue`의 생성자(`re`)
> - `Et`: `LazyReferenceCollection`
>   - `hydrate`
>   - `getCoveringPartialIndexValues`
> - `Nt`: `LazyReferenceCollection`
> - `Ku`: `PartialIndexHelper`
>   - `resolveCoveringPartialIndexValues`
>   - `partialIndexInfoForModel`
>   - `processPartialIndexInfoForModel`

LSE는 부트스트래핑 종류와 상관없이 모든 데이터를 메모리에 불러오지 않아요. 대신 앱을 쓰는 동안 필요한 데이터를 네트워크 요청이나 로컬 데이터베이스 쿼리로 가져와요. 예를 들어 이슈의 상세 화면을 열면, LSE가 그 이슈에 달린 댓글을 비동기로 불러와요. 이것을 **지연 하이드레이션**이라고 해요.

`hydrate` 메서드가 있는 클래스는 하이드레이션할 수 있어요. 예를 들면 다음과 같아요.

- `Model`
- `LazyReferenceCollection`
- `LazyReference`
- `RequestCollection`
- `LazyBackReference`
- 그 밖의 여러 가지

먼저 `Model`의 `hydrate` 메서드를 살펴봐요. 이 메서드는 하이드레이션이 필요한 프로퍼티를 모두 확인하고, 각 프로퍼티의 `hydrate` 메서드를 호출해요. 하이드레이션이 필요한 프로퍼티는 네 가지예요.

1. `LazyReferenceCollection`
2. `LazyReference`
3. `Reference`와 `ReferenceCollection`: 모델과 함께 하이드레이션되도록 설정된 것

`Reference`와 `ReferenceCollection`의 하이드레이션은 깊이 들어가지 않을게요. 다른 `Model` 인스턴스의 `hydrate` 메서드를 재귀로 호출할 뿐이에요. 대신 지연 하이드레이션을 맡는 `LazyReferenceCollection`과 `LazyReference`에 집중해요.

먼저 `LazyReferenceCollection`을 이야기해 볼게요.

앞에서 프로퍼티 정의를 다룰 때, `referenceCollection`이 일곱 가지 프로퍼티 타입 중 하나라는 것을 봤어요. 이제 이것을 더 깊이 살펴봐요. `LazyReferenceCollection` 프로퍼티에는 `OneToMany`(`Nt`) 데코레이터를 써요. 예를 들어 `comments`는 `Issue` 모델의 `LazyReferenceCollection` 프로퍼티예요. `OneToMany` 데코레이터는 프로퍼티의 메타데이터를 `ModelRegistry`에 등록해요.

```ts
Pe([Nt()], re.prototype, "comments", void 0);
```

또 모델의 생성자에서 이 프로퍼티용 `LazyReferenceCollection` 인스턴스를 초기화해요. 예를 들어 `Issue`의 생성자에서는 이렇게 해요.

```ts
this.comments = new Et(nt,this,"issueId"),
```

소스 코드는 대략 이런 모습이에요.

```js
@ClientModel()
class Issue extends BaseModel {
  @OneToMany()
  public comments = new LazyReferenceCollection(Comment, this, "issueId");
}
```

> [!NOTE] 
> 여기서 쓰는 데코레이터의 변경
>
> 제가 이 글을 쓰기 시작한 뒤에, Linear 팀이 개발자가 `LazyReferenceCollection`의 생성자를 직접 호출하지 않아도 되는 새 방식을 도입했어요. 요약하면 `OneToMany`와 비슷한 데코레이터를 더 추가했고, 이 데코레이터들이 여러 옵션으로 `LazyReferenceCollection`을 자동으로 만들어요. 이 변경은 지연 하이드레이션이 동작하는 방식에 영향을 주지 않아서, 단순하게 설명하려고 이 글에서는 뺐어요.

`LazyReferenceCollection`의 `hydrate` 메서드는 먼저 `this.getCoveringPartialIndexValues`를 호출해서 부분 인덱스 값을 가져와요. 그러면 부분 인덱스는 무엇일까요?

---

> [!note] 
> 코드 참조
>
> - `re.constructor`: `Issue.constructor`
> - `LazyReferenceCollectionImpl` (`Et`)
>   - `hydrate`
>   - `getCoveringPartialIndexValues`
> - `LazyReferenceCollection` (`Nt`)
> - `PartialIndexHelper` (`Ku`)
>   - `resolveCoveringPartialIndexValues`
>   - `partialIndexInfoForModel`
>   - `processPartialIndexInfoForModel`

**부분 인덱스**는 LSE에서 중요한 역할을 해요. **어떤 모델을 지연 로딩해야 하는지 어떻게 정할까?** 라는 핵심 질문, 다시 말해 지연 로딩하는 모델을 조회할 때 **쿼리에 어떤 파라미터를 써야 할까요?** 라는 질문에 답을 주기 때문이에요.

모델 ID를 알고 있다면 답은 간단해요. 하지만 LSE가 `User`에게 할당된 `Issue`를 불러와야 할 때는 `Issue` ID를 바로 알지 못할 수 있어요.

Linear의 데이터베이스 스키마를 설계한다고 상상해 보세요. `User`에게 할당된 `Issue`를 조회하려면 `assigneeId` 필드를 넣고 그 필드에 인덱스를 만들 거예요. 이 개념은 LSE의 프런트엔드 코드에도 비슷하게 적용돼요. `Issue` 모델을 정의할 때 `User` 모델에 대한 참조를 만들면, LSE가 그 필드의 인덱스를 자동으로 만들어요.

```typescript
Pe(
  [
    pe(() => K, "assignedIssues", {
      nullable: !0,
      indexed: !0,
    }),
  ],
  re.prototype,
  "assignee",
  void 0
);
```

원본 소스 코드는 이런 모습일 거예요.

```typescript
@ClientModel("Issue")
class Issue extends Model {
  @Reference(() => User, "assignee", {
    nullable: true,
    indexed: true,
  })
  public assignee: User | null;
}
```

`User` 모델도 `Issue`를 참조해요.

```typescript
st([Nt()], K.prototype, "assignedIssues", void 0);

// In User's constructor:
this.assignedIssues = new Et(re,this,"assigneeId",void 0,{
  canSkipNetworkHydration: ()=>this.canSkipNetworkHydration(re)
}),
```

원본 소스 코드는 이런 모습일 거예요.

```typescript
@ClientModel("User")
class User extends Model {
  @OneToMany()
  public assignedIssues = new LazyReferenceCollection(
    Issue,
    this,
    "assigneeId",
    undefined,
    {
      canSkipNetworkHydration: () => this.canSkipNetworkHydration(Issue),
    }
  );
}
```

LSE는 담당자의 ID로 `Issue`를 불러올 수 있어요. 다시 말해 `Issue`를 가져오는 쿼리 파라미터가 `assigneeId-<id-of-the-assignee>`가 될 수 있어요. 같은 방식으로 `Issue`가 속한 `Team`이나 연결된 `Project`를 기준으로도 `Issue`를 불러올 수 있어요.

모델을 어떤 방식으로 참조할 수 있는지 알아내려고 LSE는 `PartialIndexHelper`(`ku`) 클래스를 써요. 이 클래스는 한 모델을 다른 모델이 어떻게 참조할 수 있는지 설명하는 배열을 반환해요.

![직접 참조](https://github.com/wzhudev/reverse-linear-sync-engine/raw/45dc91e78e03300b69cc088057a28c42cee35a41/imgs/direct-referencies.png)

LSE는 이 방식을 한 걸음 더 밀고 나가요. 앞 절에서 이야기했듯이 `Issue`가 `Comment`를 참조하므로, `Comment`는 `Team`이 간접적으로 참조하는 셈이에요. 그래서 `Comment`에 대한 중첩 참조도 가능해요. 예를 들어 `issue.cycleId-<id-of-the-cycle>` 파라미터로 쿼리를 보내면, 이론상 그 사이클에 연결된 모든 이슈의 댓글을 전부 가져올 수 있어요.

LSE는 `Ku.processPartialIndexInfoForModel` 메서드에서 이런 중첩 참조를 계산하고, 최대 세 단계 깊이까지 지원해요. 아래 다이어그램은 여러 모델이 `Comment`를 직접 또는 간접으로 참조하는 모습을 보여줘요.

![참조](https://github.com/wzhudev/reverse-linear-sync-engine/raw/45dc91e78e03300b69cc088057a28c42cee35a41/imgs/references.png)

`getCoveringPartialIndexValues`로 돌아가 볼게요. 댓글을 조회할 때 쓰는 부분 인덱스는 스크린샷의 `i`처럼 생겼어요.

![부분 인덱스 값](https://github.com/wzhudev/reverse-linear-sync-engine/raw/45dc91e78e03300b69cc088057a28c42cee35a41/imgs/partial-index-values.png)

위 참조 다이어그램과 부분 인덱스 값 스크린샷의 관계가 분명히 보일 거예요. 간단히 말해 LSE는 각 쿼리 파라미터 끝에 참조하는 모델의 ID를 붙여요.

뒤에서 살펴보겠지만, 부분 인덱스는 서버에서 모델을 조회할 때 쓰고, 대상 모델을 이미 서버에서 가져왔는지 확인하는 데도 써요.

> [!note]
> 코드 참조
>
> - `ng.hydrateModelsByIndexedKey`: `SyncClient.hydrateModelsByIndexedKey`
> - `eg.getModelDataByIndexedKey`: `Database.getModelDataByIndexedKey`
> - `Jm`: `PartialStore`
>   - `getAllForIndexedKey`
>   - `hasModelsForPartialIndexValues`
>   - `getAllFromIndex`

부분 인덱스를 가져오고 나면, `LazyReferenceCollection`의 `hydrate` 메서드가 `SyncedStore.hydrateModels`를 호출하고, 이 메서드가 다시 `SyncClient.hydrateModelsByIndexedKey`를 불러요.

`Issue`의 `Comment` 객체를 지연 로딩한다고 가정해 봐요. 메서드 파라미터는 다음과 같아요.

1. `e`: `Comment` 모델의 클래스예요.
2. `t`: 쿼리 파라미터예요. 필드는 다음과 같아요.
   - `key`: `Issue`를 기준으로 `Comment` 참조를 불러온다는 뜻이에요.
   - `coveringPartialIndexValues`: 이 `Comment` 객체를 다른 모델이 간접적으로 참조할 수도 있다는 뜻이에요.
   - `value`: `Comment`를 참조하는 `Issue`의 ID예요.

![covering partial index values](https://github.com/wzhudev/reverse-linear-sync-engine/raw/45dc91e78e03300b69cc088057a28c42cee35a41/imgs/covering-partial-index-values.png)

이 구현에서 `LazyReferenceCollection`(LSE)은 먼저 `Database.getModelDataByIndexedKey`를 호출해서 로컬 데이터베이스에서 모델을 불러올 수 있는지 확인해요. 불러올 수 없으면 다음 조건을 보고 **네트워크 하이드레이션**이 필요한지 정해요.

1. **`coveringPartialIndexValues`가 없을 때:** `coveringPartialIndexValues` 파라미터가 없으면, LSE는 요청한 모델을 전에 서버에서 가져왔는지 알 수 없어요.
2. **스토어에 부분 인덱스가 없을 때:** `coveringPartialIndexValues`가 부분 스토어에 없으면 네트워크 하이드레이션이 필요해요.
3. **`canSkipNetworkHydration`이 `true`를 반환할 때:** 이 옵션이 `true`이면 LSE는 네트워크 하이드레이션을 건너뛰고 로컬에서 데이터를 불러와요.

`ObjectStore`를 다룰 때, `partial` 로드 전략인 모델에는 부분 인덱스를 추적하는 부분 인덱스 스토어가 있다고 했어요. 조건 2에서 확인하는 부분 스토어가 바로 이 스토어예요. 예를 들어 `Comment` 모델의 부분 인덱스 스토어에 레코드가 두 개 있다면, LSE가 전에 그 인덱스로 `Comment` 객체를 불러온 적이 있다는 뜻이고, 해당 댓글을 어느 시점에 서버에서 가져왔다는 걸 확인해 줘요. 이 부분 인덱스가 언제 갱신되는지는 뒤에서 이야기해요.

![부분 인덱스 스토어](https://github.com/wzhudev/reverse-linear-sync-engine/raw/45dc91e78e03300b69cc088057a28c42cee35a41/imgs/partial-index-stores.png)

조건을 확인한 뒤 LSE는 이렇게 처리해요.

- 네트워크 하이드레이션이 필요 없으면 `getAllFromIndex`를 호출해서 IndexedDB를 조회해요.
- 네트워크 하이드레이션이 필요하면 `BatchModelLoader.addRequest`로 요청을 예약해요.

> [!NOTE]
> 코드 참조
>
> - `BatchedRequest` (`PE`)
>   - `addRequest`
> - `BatchModelLoader` (`wm`)
>   - `handleBatch`
>   - `loadSyncBatch`
>   - `loadFullModels`
>   - `handleLoadedModels`
> - `Database` (`eg`)
>   - `setPartialIndexValueForModel`

`BatchModelLoader`는 이름 그대로 여러 네트워크 하이드레이션 요청을 GraphQL 요청 하나로 묶어요. LSE가 요청의 중복을 어떻게 없애는지는 이 글에서 자세히 다루지 않아요(코드에 이해를 돕는 주석을 달아 뒀으니 참고하세요). 여기서는 LSE가 배치 처리를 어떻게 하는지에 집중해요.

`BatchModelLoader.handleBatch` 메서드에서 Linear는 요청을 세 범주로 나눠요.

1. 부분 인덱스 키와 연결된 요청
2. `SyncGroup`과 연결된 요청
3. `indexedKey`와도 `SyncGroup`과도 연결되지 않은 요청

LSE는 범주마다 다른 메서드로 처리해요. 순서대로 각각 `loadSyncBatch`, `loadPartialModels`, `loadFullModels`예요.

`loadSyncBatch`는 `GraphQLClient.resetModelsJsonStream`을 호출해서 `https://client-api.linear.app/sync/batch`로 요청을 보내요. 요청 본문은 이렇게 생겼어요.

```json
{
  "firstSyncId": 3528373991,
  "requests": [
    {
      "indexedKey": "issueId",
      "keyValue": "bda1a998-91b0-4ceb-8f89-91b7f6608685",
      "modelName": "Comment"
    },
    {
      "indexedKey": "issueId",
      "keyValue": "bda1a998-91b0-4ceb-8f89-91b7f6608685",
      "modelName": "IssueHistory"
    }
  ]
}
```

응답은 이렇게 생겼어요.

```json
{"id":"9a4ea82f-bd0e-4a3d-a8f3-430ea570bbbb","createdAt":"2025-01-27T05:11:59.451Z","updatedAt":"2025-01-27T05:11:59.337Z","issueId":"bda1a998-91b0-4ceb-8f89-91b7f6608685","userId":"4e8622c7-0a24-412d-bf38-156e073ab384","bodyData":"{\"type\":\"doc\",\"content\":[{\"type\":\"paragraph\",\"content\":[{\"type\":\"text\",\"text\":\"Some comment.\"}]}]}","reactionData":[],"subscriberIds":["4e8622c7-0a24-412d-bf38-156e073ab384"],"__class":"Comment"}
{"id":"6168d074-cfc0-45ef-9a14-2e5162cbf3dd","createdAt":"2025-01-19T16:02:24.806Z","updatedAt":"2025-01-19T16:02:24.806Z","issueId":"bda1a998-91b0-4ceb-8f89-91b7f6608685","fromCycleId":"43714e70-c8a5-44f8-b2ce-3b8613397955","toCycleId":"8c74d891-d9f3-4b47-b4cc-79cf27d9f93c","__class":"IssueHistory"}
{"id":"c7726ecb-672f-4604-a0f5-f2bf1e420ba7","createdAt":"2025-01-06T11:49:21.638Z","updatedAt":"2025-01-06T11:49:21.638Z","issueId":"bda1a998-91b0-4ceb-8f89-91b7f6608685","actorId":"4e8622c7-0a24-412d-bf38-156e073ab384","fromPriority":3,"toPriority":1,"__class":"IssueHistory"}
{"id":"b68d9b8a-6fb4-45ab-944e-7b45a8c30673","createdAt":"2025-01-26T02:17:17.771Z","updatedAt":"2025-01-26T02:17:17.771Z","issueId":"bda1a998-91b0-4ceb-8f89-91b7f6608685","actorId":"4e8622c7-0a24-412d-bf38-156e073ab384","fromPriority":1,"toPriority":2,"__class":"IssueHistory"}
{"id":"4c8a41ec-b0b5-448e-a47d-34dfb11112d1","createdAt":"2024-12-30T07:34:42.354Z","updatedAt":"2024-12-30T08:08:27.668Z","issueId":"bda1a998-91b0-4ceb-8f89-91b7f6608685","actorId":"4e8622c7-0a24-412d-bf38-156e073ab384","fromPriority":0,"toPriority":3,"fromStateId":"dfbab132-44b7-47b9-a411-906426533033","toStateId":"7bd765d0-7fa4-40ed-9b62-ae963436682c","toCycleId":"398ba8c7-c523-43c8-b6c7-d748d0e171a4","addedLabelIds":["bd1ce6b0-b0d0-49c9-b4a3-f905674fe9ac"],"__class":"IssueHistory"}
{"id":"cd0833d4-d92e-40ef-a7d2-223d6c7b4592","createdAt":"2025-01-05T16:01:14.558Z","updatedAt":"2025-01-05T16:01:14.558Z","issueId":"bda1a998-91b0-4ceb-8f89-91b7f6608685","fromCycleId":"398ba8c7-c523-43c8-b6c7-d748d0e171a4","toCycleId":"43714e70-c8a5-44f8-b2ce-3b8613397955","__class":"IssueHistory"}
_metadata_={"returnedModelsCount":{"Comment":1,"IssueHistory":5}}
```

이 응답이 낯익지 않나요? 맞아요. 전체 부트스트래핑에서 쓴 것과 같은 형식이에요. 이후 handleLoadedModels에서 응답을 파싱하고, 모델을 데이터베이스에 쓰고, 메모리에 객체를 만들어요. 중요한 점은 요청의 부분 인덱스가 **데이터베이스에 저장된다**는 거예요. 그래서 LSE가 다음에 그 모델을 하이드레이션할 때는 네트워크 하이드레이션이 필요 없다는 것을 알 수 있어요.

이런 의문이 들 수 있어요. _왜 `/sync/patch` 요청 파라미터에는 `firstSyncId`가 있고 `lastSyncId`는 없을까요? `lastSyncId`는 클라이언트가 최신 데이터와 맞는지 판단하는 데 쓰잖아요. 이 모델들은 갱신되지 않을까요?_ 그 답은 LSE가 증분 변경(델타 패킷)을 처리하는 방식에 있고, 이는 4장에서 설명해요.

기본 아이디어는 이래요. 델타 패킷이 도착하면 LSE는 영향을 받는 모델 가운데 아직 로드되지 않은 모델을 찾아서 바로 불러와요. 이 과정 뒤에 지연 하이드레이션이 끝나더라도 LSE는 메모리에 있는 모델을 덮어쓰지 않아요. 구현 세부 사항은 `createModelsFromData` 메서드에서 볼 수 있어요.

---

> [!NOTE]
> 코드 참조
>
> - `BatchModelLoader` (`wm`)
>   - `handleBatch`
> - `Database` (`eg`)
>   - `loadPartialModels`

부분 인덱스 키 대신 `syncGroups`를 쿼리 파라미터로 쓰는 경우도 있어요. 예를 들어 `Team`에는 연결된 `Issue`가 많아서 `lazyReferenceCollection` 타입의 `issue` 프로퍼티가 있어요. 이때는 `customNetworkHydration`으로 `Team`의 `Issue`를 불러오는 쿼리 파라미터를 정의해요.

```typescript
this.issues = new Et(re, this, "teamId", void 0, {
  customNetworkHydration: () => [
    {
      modelClass: re, // Issue model
      syncGroup: this.id,
    },
    {
      modelClass: mr, // Attachment model
      syncGroup: this.id,
    },
  ],
});
```

LSE가 `Team`의 `Issue`를 불러올 때, `loadPartialModels`는 `BootstrapHelper.partialBootstrap`을 호출하고, 이 메서드가 다음과 같은 요청을 보내요.

```
https://client-api.linear.app/sync/bootstrap?type=partial&noSyncPackets=true&useCFCaching=true&noCache=true&firstSyncId=3577987809&syncGroups=aa788b7b-9b76-4caa-a439-36ca3b3d6820&onlyModels=Issue,Attachment&modelsHash=4f1dabd6151ad381a502c352b677d5c4
```

요청 파라미터에서 볼 수 있듯이 `modelClass`가 `onlyModels` 프로퍼티에 대응하고, 요청에 `syncGroups`도 들어 있어요.

---

## 2장 정리 [#takeaway-of-chapter-2]

2장에서 배운 내용을 정리해요.

- LSE는 두 종류의 데이터베이스를 만들어요.
  - 다른 데이터베이스의 정보를 저장하는 `linear_databases` 데이터베이스
  - 특정 워크스페이스의 모델, 메타데이터, 트랜잭션을 저장하는 `linear_database_<id>` 데이터베이스
- 부트스트래핑에는 전체, 부분, 로컬 세 종류가 있어요. 이 장에서는 전체 부트스트래핑을 자세히 다뤘어요.
- 싱크 ID는 데이터베이스의 전역 버전 번호예요. 클라이언트가 최신 데이터와 맞는지 판단하는 데 도움이 돼요.
- LSE는 서버나 로컬 데이터베이스에서 모델을 메모리로 지연 하이드레이션할 수 있어요.

다음 장부터는 LSE가 클라이언트와 서버 사이에서 변경을 어떻게 동기화하는지 살펴봐요. 먼저 로컬 변경을 서버로 보내는 방법부터 시작해요.
