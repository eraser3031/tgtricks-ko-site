# 3장: 트랜잭션

웹: /docs/linear/03-transactions/ · Markdown: /docs/linear/03-transactions.md

> 모델 변경을 트랜잭션으로 묶어 큐에 넣고, 서버로 보내고, 실패하면 되돌리는 방법을 살펴봐요.

> 이 문서는 Wenzhao Hu의 [Reverse Engineering Linear's Sync Engine](https://github.com/wzhudev/reverse-linear-sync-engine/blob/45dc91e78e03300b69cc088057a28c42cee35a41/README.md)을 한국어로 옮기고 말투를 다듬은 번역이에요. 원문과 번역 모두 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.ko) 라이선스를 따라요.

앞 장에서는 LSE가 서버에서 기존 모델을 불러오는 방법을 살펴봤어요. 이번에는 LSE가 클라이언트와 서버 사이에서 변경 사항을 동기화하는 방법을 살펴봐요. 특히 이 장에서는 클라이언트에서 한 변경을 서버에 동기화하는 과정을 다뤄요.

기본적인 질문부터 시작해 볼게요. **이슈(Issue)의 담당자를 바꾸면 어떤 일이 일어날까요?** 네트워크 처리, 오프라인 캐싱, 관찰 가능성 같은 복잡한 일을 LSE는 어떻게 코드 두 줄로 해낼까요?

```jsx
issue.assignee = user;
issue.save();
```

이 장에서는 `UpdateTransaction`을 예로 들게요. 앞에서처럼 세부 내용에 들어가기 전에 전체 과정을 먼저 훑어봐요.

![](https://github.com/wzhudev/reverse-linear-sync-engine/raw/45dc91e78e03300b69cc088057a28c42cee35a41/imgs/transaction-overview.png)

1. 프로퍼티에 새 값을 대입하면 시스템이 핵심 정보를 기록해요. 바뀐 프로퍼티의 이름과 이전 값이에요. **메모리에 있는 모델**은 이 변경을 반영해 **즉시** 업데이트돼요.
2. `issue.save()`를 호출하면 **`UpdateTransaction`** 이 만들어져요. 이 트랜잭션에 모델의 변경 내용이 담겨요.
3. 만들어진 `UpdateTransaction`은 요청 큐에 들어가요. 동시에 **캐싱**을 위해 IndexedDB의 `__transactions` 테이블에도 저장돼요.
4. `TransactionQueue`는 타이머를 예약해서(때로는 바로 실행해서) 큐에 쌓인 트랜잭션을 **배치** 단위로 서버에 보내요.
5. 백엔드가 배치를 성공적으로 처리하면 IndexedDB의 `__transactions` 테이블에서 그 배치가 지워져요. 그러면 Local Storage Engine(LSE)이 캐시된 배치를 비워요.
6. 트랜잭션은 `lastSyncId`가 담긴 델타 패킷이 도착할 때까지 기다린 뒤에 완료돼요.

`UpdateTransaction` 말고도 트랜잭션 종류가 네 가지 더 있어요. `TransactionQueue`는 이 트랜잭션을 만드는 메서드도 각각 제공해요.

| 축소된 이름 | 원래 이름              | 설명                                                                |
| -------------- | -------------------------- | -------------------------------------------------------------------------- |
| `M3` `Zo`      | `BaseTransaction`           | 모든 트랜잭션 타입의 기반 클래스예요.                                  |
| `Hu`           | `CreationTransaction`       | 새 모델 오브젝트를 추가하는 트랜잭션이에요.                              |
| `zu`           | `UpdatingTransaction`       | 기존 모델 오브젝트의 프로퍼티를 업데이트하는 트랜잭션이에요.        |
| `g3`           | `DeletionTransaction`       | 모델 오브젝트를 삭제하는 트랜잭션이에요(예: 댓글 삭제).     |
| `m3`           | `ArchivalTransaction`       | 모델 오브젝트를 보관 처리하는 트랜잭션이에요(예: 이슈 보관).    |
| `y3`           | `UnarchiveTransaction`      | 모델 오브젝트의 보관을 해제하는 트랜잭션이에요.                     |
| `Tc`           | `LocalTransaction`          | 아무 동작도 하지 않고 모델 오브젝트를 감싸기만 하는 단순한 트랜잭션 래퍼예요. |

## 무엇이 바뀌었는지 알아내기 [#figuring-out-what-has-been-changed]

> [!note]  
> **코드 참조**
>
> - `M1`: LSE 모델에 관찰 가능성을 더하는 데코레이터예요.
> - `as.propertyChanged`: `ClientModel.propertyChanged`
> - `as.markPropertyChanged`: `ClientModel.markPropertyChanged`
> - `as.referencedPropertyChanged`: `ClientModel.referencedPropertyChanged`
> - `as.updateReferencedModel`: `ClientModel.updateReferencedModel`

[관찰 가능성](/docs/linear/01-models-metadata/#observability-m1) 절에서 설명했듯이, LSE는 `M1` 함수로 모델 프로퍼티를 옵저버블로 만들어요. `M1`은 관찰 가능성을 더하는 데서 그치지 않고 트랜잭션을 만드는 데도 중요한 역할을 해요. 동작 방식은 이래요.

모델의 프로퍼티에 새 값을 대입하면 다음 순서로 변경이 기록돼요.

1. 세터가 대입을 가로채서 `propertyChanged`를 실행해요.
2. `propertyChanged`가 **프로퍼티 이름**, **이전 값**, **새 값**을 넘겨 `markPropertyChanged`를 호출해요.
3. `markPropertyChanged`가 이전 값을 직렬화해서 `modifiedProperties`에 저장해요. 이 직렬화된 데이터는 나중에 트랜잭션을 만들 때 써요.

![수정된 프로퍼티](https://github.com/wzhudev/reverse-linear-sync-engine/raw/45dc91e78e03300b69cc088057a28c42cee35a41/imgs/modified-properties.png)

**`save()`를 호출하기 전에 메모리의 모델은 이미 업데이트된 상태**라는 점을 기억해 두세요. 트랜잭션이 메모리의 모델을 업데이트하는 게 **아니에요**. 프로퍼티를 바꾸는 순간 바로 업데이트돼요.

다만 트랜잭션은 **실행 취소**와 **다시 실행**, 그리고 메모리 모델 업데이트에서 중요한 역할을 해요. 이 내용은 **5장**에서 자세히 살펴봐요.

## `UpdateTransaction` 만들기 [#generating-an-updatetransaction]

> [!note]
> 코드 참조
>
> - `as.save`: `ClientModel.save`. 이 메서드가 `SyncedStore.save`를 호출해요.
> - `sg.save`: `SyncedStore.save`. 이 메서드가 `SyncClient.update`를 호출해요.
> - `ng.update`: `SyncClient.update`
> - `uce.update`: `TransactionQueue.update`

`SyncClient.update`는 `TransactionQueue.update` 메서드를 호출해서 `UpdateTransaction` 인스턴스를 만들어요. `UpdateTransaction`을 만드는 동안 모델의 `changeSnapshot` 함수가 실행돼요. 그 결과 변경 내용을 나타내는 오브젝트가 만들어지고, 이 오브젝트가 `UpdateTransaction`의 `changeSnapshot` 프로퍼티에 연결돼요.

![변경 스냅샷](https://github.com/wzhudev/reverse-linear-sync-engine/raw/45dc91e78e03300b69cc088057a28c42cee35a41/imgs/change-snapshot.png)

`UpdateTransaction`에는 다음 프로퍼티가 있어요.

- **`type`**: 트랜잭션의 종류예요.
- **`model`**: 이 트랜잭션과 관련된 메모리 속 모델 오브젝트예요.
- **`batchIndex`**: 트랜잭션마다 `batchIndex`가 있어요. `TransactionQueue`에도 `batchIndex` 프로퍼티가 있는데, 트랜잭션을 만들 때 이 값을 넣어줘요. `batchIndex`가 같은 트랜잭션은 한 묶음이 되어 서버에 한 번에 전송돼요.

`TransactionQueue.update`의 마지막에는 `TransactionQueue.enqueueTransaction` 메서드를 호출해서 트랜잭션을 `createdTransactions` 큐에 넣어요.

## 트랜잭션 큐잉 [#queueing-transactions]

> [!note]
> 코드 참조
>
> - `uce`: `TransactionQueue`
>   - `enqueueTransaction`
>   - `commitCreatedTransactions`
>   - `dequeueNextTransactions`
>   - `dequeueTransaction`
> - `ww`: `MicrotaskScheduler`
> - `as.prepare`: `Model.prepare`
> - `as.updateMutation`: `Model.updateMutation`
> - `zu.graphQLMutation`: `UpdateTransaction.graphQLMutation`

`TransactionQueue`는 트랜잭션 인스턴스를 만들 뿐 아니라, 트랜잭션을 관리하고 서버에 보내는 일도 맡아요. 이를 위해 배열 네 개를 써요.

![트랜잭션 큐](https://github.com/wzhudev/reverse-linear-sync-engine/raw/45dc91e78e03300b69cc088057a28c42cee35a41/imgs/transaction-queues.png)

1. **`createdTransactions`**: 트랜잭션을 만들면 가장 먼저 이 배열에 들어가요.

**`commitCreatedTransactions`** 스케줄러가 `createdTransactions`의 트랜잭션을 모두 `queuedTransactions` 끝으로 옮기고, `TransactionQueue`의 `batchIndex`를 1 올려요. `commitCreatedTransactions` 스케줄러는 마이크로태스크(현재 실행 중인 작업이 끝난 직후에 도는 작은 작업)로 동작해요. 그래서 **같은 이벤트 루프 안에서 만든 트랜잭션은 `batchIndex`가 같아요**.

트랜잭션을 `queuedTransactions`로 옮길 때는 `__transactions` 테이블에도 저장해요. 트랜잭션을 서버에 보내기 전에 클라이언트가 닫혀도, `__transactions` 테이블에서 다시 불러와 재전송할 수 있어요.

2. **`queuedTransactions`**: 실행을 기다리는 트랜잭션이에요.

**`dequeueTransaction`** 스케줄러가 `queuedTransactions`에서 트랜잭션을 준비하고, 배치 단위로 `executingTransactions`에 옮겨요.

한 배치에 어떤 트랜잭션을 `executingTransactions`로 옮길지는 세 가지 요인이 정해요. 처음 두 요인은 바로 아래에, 세 번째 요인은 GraphQL 쿼리 예시 뒤에 있어요.

- **트랜잭션 수 제한:** `queuedTransactions`에 트랜잭션이 너무 많으면 스케줄러가 `executingTransactions`로 아무것도 옮기지 않아요. 요청이 너무 많이 몰려 서버에 부담을 주는 걸 막기 위해서예요.
  
- **배치 인덱스와 독립성:** 한 배치로 묶으려면 트랜잭션의 `batchIndex`가 같아야 하고, 서로 **독립적**이어야 해요.

실행하기 전에 LSE는 각 트랜잭션의 `prepare` 메서드를 호출해요. 이 메서드는 트랜잭션에 맞는 GraphQL 뮤테이션 쿼리를 만들어요. 트랜잭션 오브젝트마다 이 쿼리를 만들어 `graphQLMutationPrepared` 프로퍼티에 연결하는 `graphQLMutation` 함수가 있어요. 예를 들어 이슈의 담당자를 바꾸면 다음과 같은 쿼리가 만들어질 수 있어요.

```json
{
  "mutationText": "issueUpdate(id: \"a3dad63b-8302-4f1f-a874-a80e6d9ed418\", input: $issueUpdateInput) { lastSyncId }",
  "variables": {
    "issueUpdateInput": {
      "assigneeId": "4e8622c7-0a24-412d-bf38-156e073ab384"
    }
  },
  "variableTypes": {
    "issueUpdateInput": "IssueUpdateInput"
  }
}
```

- **GraphQL 뮤테이션 크기 제한(세 번째 요인):** `graphQLMutationPrepared` 프로퍼티는 크기로 평가해요. `executingTransactions` 앞쪽에 쌓인 트랜잭션의 크기를 합친 값이 일정 기준을 넘으면, 스케줄러는 `executingTransactions`로 트랜잭션을 더 옮기지 않아요. 지나치게 큰 GraphQL 쿼리를 서버에 보내지 않기 위해서예요.

마지막으로 스케줄러가 `executeTransactionBatch`를 호출해서, 다음 배치의 트랜잭션을 `queuedTransactions`에서 `executingTransactions`로 옮겨요.

3. **`executingTransactions`**: 서버로 보냈지만 아직 수락도 거절도 되지 않은 트랜잭션이에요. 다음 절에서 이 트랜잭션을 어떻게 실행하는지 설명할게요.

4. **`persistedTransactionsEnqueue`**: 데이터베이스를 부트스트랩할 때 `__transactions` 테이블에 저장돼 있던 트랜잭션을 이 배열로 불러와요. 원격 업데이트를 처리한 뒤에는 이 트랜잭션을 `queuedTransactions`로 옮기고 실행을 기다려요. 이 내용은 이 장의 마지막 절에서 다뤄요.

**`completedButUnsyncedTransactions`** 라는 특별한 배열도 하나 더 있어요. 이 배열이 어떻게 동작하는지는 4장에서 **트랜잭션 리베이스**를 설명할 때 알려드릴게요.

## 트랜잭션 실행 [#executing-transactions]

> [!NOTE] 
> 코드 참조
>
> - `uce.executeTransactionBatch`: `TransactionQueue.executeTransactionBatch`
> - `dce.execute`: `TransactionExecutor.execute`
> - `OE`: `WaitSyncQueue`

LSE는 `TransactionExecutor`를 만들어서 트랜잭션 배치를 실행해요. `TransactionExecutor.execute` 메서드는 배치에 있는 각 트랜잭션의 `graphQLMutationPrepared` 프로퍼티를 하나의 GraphQL 뮤테이션 쿼리로 합쳐서 서버에 보내요. 예를 들면 이래요.

```json
{
  "query": "mutation IssueUpdate($issueUpdateInput: IssueUpdateInput!) { issueUpdate(id: \"a3dad63b-8302-4f1f-a874-a80e6d9ed418\", input: $issueUpdateInput) { lastSyncId } }",
  "variables": {
    "issueUpdateInput": {
      "assigneeId": "4e8622c7-0a24-412d-bf38-156e073ab384"
    }
  },
  "operationName": "IssueUpdate"
}
```

응답에는 뮤테이션 쿼리마다 `lastSyncId`가 들어 있어요.

```json
{
  "data": {
    "issueUpdate": {
      "lastSyncId": 3273967562
    }
  }
}
```

이 예시는 뮤테이션 쿼리가 하나뿐이라 너무 단순해 보일 수 있어요. 더 복잡한 예로 새 `Project`를 만드는 경우를 살펴볼게요. 이 경우 LSE가 한 배치 안의 여러 트랜잭션을 어떻게 다루는지 분명히 볼 수 있어요. 요청은 이렇게 생겼어요.

```json
{
  "query": "mutation ProjectCreate_DocumentContentCreate($projectCreateInput: ProjectCreateInput!, $documentContentCreateInput: DocumentContentCreateInput!) { o1:projectCreate(input: $projectCreateInput) { lastSyncId }, o2:documentContentCreate(input: $documentContentCreateInput) { lastSyncId } }",
  "variables": {
    "projectCreateInput": {
      "id": "940e248c-2226-4b0a-a14e-8a410ccfaaa7",
      "name": "New Project",
      "description": "",
      "color": "#bec2c8",
      "statusId": "fd3884d6-b740-41d9-919c-796119d6c5ed",
      "memberIds": [],
      "sortOrder": 1005.71,
      "prioritySortOrder": 0,
      "priority": 0,
      "teamIds": ["369af3b8-7d07-426f-aaad-773eccd97202"]
    },
    "documentContentCreateInput": {
      "id": "bb75174e-1e26-46ae-94b6-67977be435c3",
      "projectId": "940e248c-2226-4b0a-a14e-8a410ccfaaa7"
    }
  },
  "operationName": "ProjectCreate_DocumentContentCreate"
}
```

응답은 이래요.

```json
{
  "data": {
    "o1": {
      "lastSyncId": 0
    },
    "o2": {
      "lastSyncId": 3588467486
    }
  }
}
```

응답을 받으면 LSE는 다음 일을 해요.

1. `executingTransactions` 큐에서 트랜잭션을 제거해요.
2. `__transactions` 테이블에서 캐시를 지워요.
3. 각 트랜잭션의 `transactionCompleted` 메서드를 호출해 트랜잭션을 완료해요.

트랜잭션의 `syncInNeededForCompletion` 프로퍼티에는 응답에 든 `lastSyncId` 중 가장 큰 값이 들어가요. 그리고 `lastSyncId`가 일치하는 싱크 액션이 클라이언트에 도착해야 트랜잭션이 완료된 것으로 봐요. 트랜잭션이 바로 완료되지 못하면 `completedButUnsyncedTransactions` 큐로 옮겨요. 다음 장에서 설명하듯이, `completedButUnsyncedTransactions` 큐는 **트랜잭션 리베이스**를 하는 데 꼭 필요해요.

여기서 중요한 점이 있어요. 지금까지 LSE는 IndexedDB의 모델 테이블(예: `Issue` 테이블)을 **전혀** 수정하지 않았어요. Linear에서 로컬 데이터베이스는 서버 데이터베이스(SSOT, 데이터의 기준이 되는 단일 출처)의 부분 집합이라서, 서버가 승인하지 않은 변경은 담을 수 없기 때문이에요. 서버가 트랜잭션을 거절했을 때 모델 테이블을 미리 수정해 뒀다면, 로컬 데이터베이스의 변경을 되돌리기 어렵고 실수하기도 쉬워요.

서버가 뮤테이션 쿼리를 거절해도 `transactionCompleted`는 호출돼요. 이때는 오류와 함께 호출돼요. 그러면 트랜잭션이 `rollback` 메서드를 실행해서 클라이언트에서 한 변경을 되돌리고, `executingTransaction` 큐에서 빠져요.

## 저장된 트랜잭션 [#persisted-transactions]

> [!NOTE] 
> 코드 참조
>
> - `eg.putTransaction`: `Database.putTransaction`
> - `ng.bootstrap`: `SyncClient.bootstrap`
> - `uce.loadPersistedTransactions`: `TransactionQueue.loadPersistedTransactions`
> - `uce.confirmPersistedTransactions`: `TransactionQueue.confirmPersistedTransactions`

앞에서 트랜잭션을 `queuedTransactions`로 옮길 때 캐싱을 위해 `__transactions` 테이블에도 저장한다고 했어요. 이때 트랜잭션의 `serialize` 메서드를 호출해요. `serialize`는 트랜잭션 종류마다 구현이 달라요.

저장된 트랜잭션은 이렇게 복원돼요.

1. Linear를 다시 시작하면 `TransactionQueue.loadPersistedTransactions`가 캐시된 트랜잭션을 불러오면서 역직렬화해요.
2. 트랜잭션 종류마다 정적 메서드 `fromSerializedData`가 구현돼 있어요. 이 메서드는 트랜잭션을 **다시 실행**해서 메모리의 모델을 수정해요. 덕분에 재시작 뒤에도 클라이언트의 상태를 되살릴 수 있어요.
3. 마지막으로 부트스트랩 과정에서 `TransactionQueue.confirmPersistedTransactions`를 호출해, 불러온 트랜잭션을 `createdTransactions`로 옮겨요.

이 과정에서 뜻밖의 문제가 생길 가능성이 조금 있어요. 예를 들어 클라이언트가 트랜잭션을 서버에 보냈는데 응답을 받기 전에 창이 닫히면, 이 트랜잭션은 `__transactions` 테이블에 남아요. 클라이언트를 다시 시작하면 이 트랜잭션을 다시 불러와서 서버에 또 보내요.

멱등성(같은 요청을 여러 번 보내도 결과가 같은 성질)이 없는 트랜잭션도 있어서, 사용자가 "존재하지 않는 모델은 삭제할 수 없어요" 같은 오류를 만날 수 있어요. 드물게 일어나고 사용자 경험에 큰 영향을 주지도 않지만, 알아 둘 만한 점이에요. (OT 시스템에서는 더 심각한 문제가 될 수 있어요. 그래서 OT 시스템은 보통 트랜잭션이나 오퍼레이션에 증가하는 카운터를 넣어 중복을 걸러내요.)

## 3장 정리 [#takeaway-of-chapter-3]

이 장을 정리해 볼게요.

**첫째, 클라이언트에서 하는 작업은 로컬 데이터베이스의 테이블을 직접 수정하지 않아요!** 메모리의 모델만 바꾸고, 그 변경을 트랜잭션으로 서버에 보내요. 다음 장에서 보겠지만, 서버에서 그 변경의 델타 패킷을 받은 뒤에야 로컬 모델이 업데이트돼요.

**둘째, LSE는 트랜잭션 큐로 트랜잭션을 관리해요.** 큐는 트랜잭션을 배치 단위로 서버에 보내도록 예약해요. 이렇게 배치로 묶으면 서버에 보내는 요청 수가 줄어서 효율이 좋아져요.

**마지막으로, LSE는 트랜잭션을 견고하게 다뤄요.** 트랜잭션을 로컬 데이터베이스에 저장해 두었다가 클라이언트를 다시 시작한 뒤에 복원할 수 있어요. 덕분에 클라이언트가 중단되거나 네트워크 연결이 끊겨도 변경 사항을 잃지 않아요.
