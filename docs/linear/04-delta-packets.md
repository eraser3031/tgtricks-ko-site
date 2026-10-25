# 4장: 델타 패킷

웹: /docs/linear/04-delta-packets/ · Markdown: /docs/linear/04-delta-packets.md

> 서버와 연결을 맺고, 서버가 보낸 델타 패킷을 로컬 모델에 적용하는 과정을 따라가요.

> 이 문서는 Wenzhao Hu의 [Reverse Engineering Linear's Sync Engine](https://github.com/wzhudev/reverse-linear-sync-engine/blob/45dc91e78e03300b69cc088057a28c42cee35a41/README.md)을 한국어로 옮기고 말투를 다듬은 번역이에요. 원문과 번역 모두 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.ko) 라이선스를 따라요.

이 장에서는 LSE가 증분 업데이트(바뀐 부분만 담은 업데이트)를 처리하고 클라이언트를 서버와 계속 동기화하는 방법을 살펴봐요.

앞 장들에서처럼 전체 흐름부터 훑어볼게요.

1. 부트스트래핑이 끝날 무렵, 클라이언트는 저장된 트랜잭션을 불러온 뒤 서버와 WebSocket 연결을 맺어요. 이 연결로 증분 업데이트인 델타 패킷을 받아요.
2. 델타 패킷을 처리하려면 여러 작업이 필요해요. 메모리와 로컬 데이터베이스의 모델을 업데이트하고, 트랜잭션을 리베이스하는 작업 등이에요.

## 연결 맺기 [#establishing-connection]

> [!NOTE] 
> 코드 참조
>
> - `ng.startSyncing`: `SyncClient.startSyncing`
> - `ng.constructor`: `SyncClient.constructor`
> - `handshakeCallback` 콜백

부트스트래핑의 마지막 단계는 로컬 데이터베이스에서 저장된 트랜잭션을 불러온 뒤, 서버와 WebSocket 연결을 맺고 증분 업데이트를 받는 거예요.

연결이 맺어지면 `handshakeCallback`이 실행돼요. 클라이언트는 이 콜백의 파라미터로 받은 `lastSyncId`를 로컬의 lastSyncId와 비교해서 놓친 증분 변경이 있는지 확인해요. 차이가 있으면 놓친 델타 패킷을 서버에 요청하고 그대로 적용해요.

콜백의 파라미터는 다음과 같아요.

```typescript
{
  "userSyncGroups": {
    "all": [
      "89388c30-9823-4b14-8140-4e0650fbb9eb",
      "094f76cf-b0c1-4f6c-9908-1801a6654f05",
      "4e8622c7-0a24-412d-bf38-156e073ab384",
      "E7E6104E-AAAA-42BC-9B8B-B91FCDD9946B",
      "AD619ACC-AAAA-4D84-AD23-61DDCA8319A0",
      "CDA201A7-AAAA-45C5-888B-3CE8B747D26B",
      "B0B41C7E-AAAA-4C7D-A93D-CD9565DA4358"
    ],
    "optimized": [
      "4e8622c7-0a24-412d-bf38-156e073ab384",
      "E7E6104E-AAAA-42BC-9B8B-B91FCDD9946B",
      "AD619ACC-AAAA-4D84-AD23-61DDCA8319A0",
      "CDA201A7-AAAA-45C5-888B-3CE8B747D26B",
      "B0B41C7E-AAAA-4C7D-A93D-CD9565DA4358"
    ]
  },
  "lastSyncId": 3529152751,
  "lastSequentialSyncId": 3529152751,
  "databaseVersion": 1179
}
```

`SyncClient` 모듈은 WebSocket 연결의 `SyncMessage` 채널도 구독해요. 델타 패킷은 이 채널로 와요. 패킷을 받으면 `applyDelta` 메서드를 호출해서 업데이트를 처리하고 적용해요.

## 델타 적용 [#applying-deltas]

> [!NOTE]
> 코드 참조
>
> - `ng.applyDelta`: `SyncClient.applyDelta`
> - `ng.constructor`: `SyncClient.constructor`
> - `oce.addSyncPacket`: `SyncActionStore.addSyncPacket`
> - `zu.supportedPacket`: `DependentsLoader.supportedPacket`
> - `uce.modelUpserted`: `TransactionQueue.modelUpserted`

서버는 클라이언트에서 GraphQL 뮤테이션을 받으면 쿼리를 실행하고 델타 패킷 묶음을 만들어요. 그리고 이 패킷을 연결된 모든 클라이언트에 브로드캐스트해요. **뮤테이션을 보낸 클라이언트도 포함**해서요.

델타 패킷마다 서버에서 일어난 변경, 곧 **싱크 액션**이 담겨 있어요. 예를 들어 `Issue`의 담당자를 바꾸면 클라이언트는 다음과 같은 델타 패킷을 받아요.

```jsx
[
  {
    id: 2361610825,
    modelName: "Issue",
    modelId: "a8e26eed-7ad4-43c6-a505-cc6a42b98117",
    action: "U",
    data: {
      id: "a8e26eed-7ad4-43c6-a505-cc6a42b98117",
      title: "Connect to Slack",
      number: 3,
      teamId: "369af3b8-7d07-426f-aaad-773eccd97202",
      stateId: "28d78a58-9fc1-4bf1-b1a3-8887bdbebca4",
      labelIds: [],
      priority: 3,
      createdAt: "2024-05-29T03:08:15.383Z",
      sortOrder: -12246.37,
      updatedAt: "2024-07-13T06:25:40.612Z",
      assigneeId: "e86b9ddf-819e-4e77-8323-55dd488cb17c",
      boardOrder: 0,
      reactionData: [],
      subscriberIds: ["e86b9ddf-819e-4e77-8323-55dd488cb17c"],
      prioritySortOrder: -12246.37,
      previousIdentifiers: [],
    },
    __class: "SyncAction",
  },
  {
    id: 2361610826,
    modelName: "IssueHistory",
    modelId: "ac1c69bb-a37e-4148-9a35-94413dde172d",
    action: "I",
    data: {
      id: "ac1c69bb-a37e-4148-9a35-94413dde172d",
      actorId: "e86b9ddf-819e-4e77-8323-55dd488cb17c",
      issueId: "a8e26eed-7ad4-43c6-a505-cc6a42b98117",
      createdAt: "2024-07-13T06:25:40.581Z",
      updatedAt: "2024-07-13T06:25:40.581Z",
      toAssigneeId: "e86b9ddf-819e-4e77-8323-55dd488cb17c",
    },
    __class: "SyncAction",
  },
  {
    id: 2361610854,
    modelName: "Activity",
    modelId: "1321dc17-cceb-4708-8485-2406d7efdfc5",
    action: "I",
    data: {
      id: "1321dc17-cceb-4708-8485-2406d7efdfc5",
      events: [
        {
          type: "issue_updated",
          issueId: "a8e26eed-7ad4-43c6-a505-cc6a42b98117",
          issueTitle: "Connect to Slack",
          changedColumns: ["subscriberIds", "assigneeId"],
        },
      ],
      userId: "e86b9ddf-819e-4e77-8323-55dd488cb17c",
      issueId: "a8e26eed-7ad4-43c6-a505-cc6a42b98117",
      createdAt: "2024-07-13T06:25:41.924Z",
      updatedAt: "2024-07-13T06:25:41.924Z",
    },
    __class: "SyncAction",
  },
];
```

위 예시처럼 액션마다 정수형 `id` 필드가 있어요. 이 값은 싱크 액션에 연결된 싱크 ID예요. (세 번째 싱크 액션의 `id`는 두 번째 싱크 액션의 `id`보다 28 커요. `lastSyncId`가 워크스페이스 하나가 아니라 데이터베이스 전체에 걸쳐 있다는 뜻이에요.)

액션에는 `action` 타입도 있어요. 가능한 타입은 다음 여덟 가지예요.

1. `I` - 삽입(Insertion)
2. `U` - 업데이트(Update)
3. `A` - 보관(Archiving)
4. `D` - 삭제(Deletion)
5. `C` - 커버링(Covering)
6. `G` - 싱크 그룹 변경(Changing sync groups)
7. `S` - 싱크 그룹 변경(`G`와 어떻게 다른지는 아직 분명하지 않아요)
8. `V` - 보관 해제(Unarchiving)

`ng.applyDelta` 메서드가 이 싱크 액션을 처리해요. 이 메서드는 아래 일곱 단계를 차례로 해요(구현 세부 사항은 소스를 참고하세요).

1. 사용자가 싱크 그룹에 추가됐는지 제거됐는지 확인해요. 사용자가 싱크 그룹에 추가됐다면 LSE는 그 싱크 그룹의 모델을 가져오려고 네트워크 요청을 보내요(사실상 부분 부트스트래핑이에요). LSE는 응답이 올 때까지 기다린 뒤에 싱크 액션 처리를 이어가요.

2. 특정 액션의 의존 대상(싱크 액션이 다루는 모델과 관련된 다른 모델)을 불러와요.

---

> [!NOTE]  
> **코드 참조**
> - `ng.applyDelta`: `SyncClient.applyDelta`
> - `Zu.supportedPacket`: `DependentsLoader.supportedPacket`

이 단계에서는 싱크 액션에 관련된 일부 모델의 참조를 불러와요. 예를 들어 `Issue`의 하위 이슈가 바뀌면 LSE는 그 하위 이슈를 불러와야 해요. 왜 그럴까요? 답은 `DependentsLoader.supportedPacket`에 있어요. 이 메서드가 어떤 싱크 액션에서 의존 모델을 불러와야 하는지 가려내요.

싱크 액션이 불러오기 대상이 되려면 다음 조건을 모두 만족해야 해요.

- 타입이 "I", "A", "D"가 **아니어야** 해요.
- 액션이 다루는 모델이 `Issue` 또는 `Project`여야 하고, 부분 인덱스에 쓰이는 모델이어야 해요. `Issue`와 `Project`는 모두 여기에 해당해요.
- 액션 타입이 "V"이거나, 연결된 모델의 참조 중에 바뀐 것이 있고 LSE가 그 바뀐 참조의 부분 인덱스 키를 이미 저장하고 있어야 해요. (이 부분은 조금 까다로워요. 최대한 설명해 볼게요. 하지만 가장 확실하게 이해하는 방법은 소스 코드를 직접 보는 거예요.)

마지막 조건은 세 단계로 확인해요.

1. 먼저 LSE는 모델의 **임시 부분 인덱스 키(transient partial indexed keys)** 를 가져와요. 모델의 부분 인덱스와 의존 대상을 곱집합(Cartesian product)으로 만든 키예요. 예를 들어 `Issue`의 임시 부분 인덱스 키는 이렇게 생겼어요.

   ![](https://github.com/wzhudev/reverse-linear-sync-engine/raw/45dc91e78e03300b69cc088057a28c42cee35a41/imgs/transient-partial-index-keys.png)

   이 키는 부분 인덱스 9개와 의존 대상 17개의 곱집합이에요.

2. 다음으로 LSE는 모델의 참조 중에 바뀐 것이 있는지 확인해요. 바뀐 게 있으면 업데이트된 참조에 대한 새 부분 인덱스 값을 만들어요.

3. 마지막으로 LSE는 새 부분 인덱스 값이 이미 로컬 데이터베이스에 저장돼 있는지 확인해요. 저장돼 있다면 그 부분 인덱스 값이 이제 새 의존 모델을 가리키므로, 이 모델의 의존 대상을 업데이트해야 한다는 뜻이에요.

---

3. 새 싱크 그룹과 그 의존 대상의 데이터를 로컬 데이터베이스에 써요.

4. 모든 싱크 액션을 순회하며 해석해서 로컬 데이터베이스를 업데이트해요.

이 단계에서 LSE는 `TransactionQueue.modelUpserted`를 호출해서, 싱크 액션 때문에 더는 유효하지 않은 로컬 `CreationTransaction`을 지워요. `CreationTransaction`의 UUID가 모델의 ID와 같으면 그 트랜잭션을 취소해요. UUID는 클라이언트에서 만들기 때문에, 이 단계가 UUID 충돌을 막아줘요.

사용자가 싱크 그룹에서 나가면 그 그룹에 속한 모델도 함께 지워요. 앞 장에서 말했듯이 LSE는 서버가 변경을 확정하기 전에는 로컬 데이터베이스를 수정하지 않아요.

5. 모든 싱크 액션을 다시 순회하며 메모리의 데이터를 업데이트해요.

---

이 단계에서 LSE는 싱크 액션을 두 번 순회해요.

**첫 번째 순회**는 싱크 액션을 실행할 모델을 준비해요.

- 타입이 "I", "V", "U"인 액션은 LSE가 해당 모델 인스턴스를 만들어요.
- 타입이 "A"인 액션은 LSE가 모델의 프로퍼티를 업데이트해요.

그다음 LSE는 새로 만든 모델에 참조를 붙여요. 그 전에 LSE는 새로 만든 모델이 같은 델타 패킷 안의 다른 싱크 액션에서 삭제됐는지 확인해요. 불필요한 작업을 피하기 위해서예요. 모델을 만든 액션과 삭제하는 액션의 `syncId`를 비교해서 확인해요. 삭제 액션의 `syncId`가 더 크면 그 모델은 만들지 않아요.

**두 번째 순회**는 싱크 액션을 하나씩 처리해요. 타입이 "I", "V", "U", "C"인 액션에는 LSE가 `UpdateTransaction`을 **리베이스**해요. 리베이스는 서버에서 온 변경을 기준으로 아직 서버가 처리하지 않은 로컬 트랜잭션을 다시 맞추는 작업이에요. 자세한 내용은 아래에서 설명해요.

> [!NOTE]  
> **코드 참조**
> - `ng.applyDelta`: `SyncClient.applyDelta`
> - `uce.rebaseTransactions`: `SyncActionStore.addSyncPacket`
> - `zu.rebase`: `UpdateTransaction.rebase`

싱크 액션을 적용할 때 로컬 트랜잭션과 충돌이 생길 수 있어요. 예를 들어 동료가 담당자를 Alice로 바꾸는 동안 내가 동시에 Bob으로 바꾼다고 해 볼게요. 서버는 동료의 업데이트를 먼저 처리해요. 그러면 "last-writer-wins" 원칙(나중에 쓴 쪽이 이기는 원칙)에 따라 서버의 담당자는 결국 Bob이 돼요.

내 클라이언트에서는 이런 일이 일어나요.

1. 담당자를 바꾸는 `UpdateTransaction`을 만들어요.
2. 서버가 이 트랜잭션을 실행하기 전에, 담당자를 Alice로 바꾸는 델타 패킷이 도착해요.
3. 이때 LSE는 리베이스를 해야 해요. "last-writer-wins" 원칙에 따라 메모리의 모델을 다시 Bob으로 되돌려야 하거든요.

이 리베이스는 `rebaseTransactions` 메서드에서 일어나요. 큐에 있는 모든 `UpdateTransaction` 오브젝트가 `rebase` 메서드를 호출해요. 각 트랜잭션의 `original` 값은 델타 패킷의 값(여기서는 Alice)으로 업데이트되고, 메모리의 모델은 Bob으로 다시 설정돼요. 이 방식은 OT(Operational Transformation)와 비슷해요.

앞 장에서 설명한 `completedButUnsyncedTransactions` 큐를 기억하나요? 리베이스하는 동안 LSE는 이 큐에 있는 트랜잭션 중에 `syncIdNeededForCompletion`이 델타 패킷의 `lastSyncId`보다 작거나 같은 것이 있는지 확인해요. 있으면 그 트랜잭션을 `completedButUnsyncedTransactions` 큐에서 제거해요.

---

6. 클라이언트의 `lastSyncId`를 업데이트하고, 싱크 그룹이 바뀌었으면 `firstSyncId`도 업데이트해요.

7. `lastSyncId`를 기다리던 완료된 트랜잭션을 마무리해요.

델타 패킷을 받은 뒤 클라이언트는 그 패킷의 `lastSyncId`를 기다리는 트랜잭션이 있는지 확인해요. 있으면 다음과 같이 마무리해요.

```typescript
this.syncWaitQueue.progressQueue(this.lastSyncId); 
```

LSE는 위 단계를 모두 `updateLock.runExclusive` 콜백 안에서 실행해요. 그래서 델타 패킷 하나를 완전히 처리한 뒤에야 다음 패킷을 처리해요. 이렇게 해서 클라이언트와 서버의 상태가 일관되게 유지돼요.

## 4장 정리 [#takeaway-of-chapter-4]

**LSE는 델타 패킷으로 클라이언트를 서버와 동기화된 상태로 유지해요.** 클라이언트가 서버에 뮤테이션을 보내면 서버가 뮤테이션을 처리하고 델타 패킷을 만들어요. 이 패킷에는 서버에서 일어난 변경이 담겨 있고, 연결된 모든 클라이언트에 브로드캐스트돼요. 그러면 클라이언트가 이 변경을 메모리의 모델에 적용해요.

델타 패킷은 트랜잭션보다 더 많은 데이터를 담고 있어요. `UpdateTransaction`과 그에 대응하는 델타 패킷을 자세히 보면 알 수 있어요. 구체적으로 델타 패킷에는 담당자 변경에 대한 `IssueHistory`가 더 들어 있어요.

OT와 비교하면 서버 역할이 달라요.

- OT에서는 서버가 주로 오퍼레이션 변환, 권한 검증, 오퍼레이션 실행을 맡아 단일 진실 공급원을 유지해요.
- LSE의 백엔드는 이런 일에 더해 비즈니스 로직도 처리해요.

확인 응답을 보내는 범위도 달라요.

- OT는 변경한 클라이언트에만 확인 응답(acknowledgment)을 보내요.
- 반대로 **LSE는 수정된 모든 모델 프로퍼티를 연결된 모든 클라이언트에 보내요. 수정한 클라이언트가 뮤테이션을 보낸 클라이언트가 아니어도 마찬가지예요.** 이렇게 하면 WebSocket 연결 관리가 간단해져요.

마지막으로 LSE는 단순한 **Last-Writer-Wins** 전략으로 충돌을 해결해요. 이 전략은 `UpdateTransaction`의 충돌에만 적용돼요.
