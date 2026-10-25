# 1장: 모델과 메타데이터 정의

웹: /docs/linear/01-models-metadata/ · Markdown: /docs/linear/01-models-metadata.md

> 모델, 프로퍼티, 참조를 데코레이터와 메타데이터로 정의하는 방법과 MobX로 관찰 가능하게 만드는 과정을 살펴봐요.

> 이 문서는 Wenzhao Hu의 [Reverse Engineering Linear's Sync Engine](https://github.com/wzhudev/reverse-linear-sync-engine/blob/45dc91e78e03300b69cc088057a28c42cee35a41/README.md)을 한국어로 옮기고 말투를 다듬은 번역이에요. 원문과 번역 모두 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.ko) 라이선스를 따라요.

가장 먼저 LSE가 모델을 어떻게 정의하는지 알아봐야 해요.

## `ModelRegistry` [#modelregistry]

> [!NOTE]
> 코드 참조
>
> - `rr`: `ModelRegistry`

LSE는 모델의 메타데이터를 `ModelRegistry`라는 상세한 딕셔너리에 모아서 관리해요. Linear는 시작할 때 가장 먼저 이 메타데이터를 만들어요. 메타데이터에는 모델의 프로퍼티, 메서드(액션), 계산된 값(computed value)이 들어 있어요.

![model registry](https://github.com/wzhudev/reverse-linear-sync-engine/raw/45dc91e78e03300b69cc088057a28c42cee35a41/imgs/model-registry.png)

> [!NOTE]
> 난독화된 이름
>
> 스크린샷에 보이는 이름(예: Xs)은 GitHub 소스 코드의 이름(rr)과 다를 수 있어요. 스크린샷마다 이름이 서로 다를 수도 있어요. Linear는 거의 30분마다 배포하기 때문에 지극히 자연스러운 일이에요.

`ModelRegistry`는 static 멤버를 가진 클래스예요. 여러 종류의 메타데이터를 저장하고, 이 정보를 등록하고 가져오는 메서드를 제공해요. 예를 들면 이런 것들이 있어요.

- **`modelLookup`**: 모델 이름을 생성자에 연결해요.
- **`modelPropertyLookup`**: 모델 프로퍼티의 메타데이터를 저장해요.
- **`modelReferencedPropertyLookup`**: 모델 참조의 메타데이터를 저장해요.
- 그 밖의 여러 가지

이 장에서는 이 메타데이터 중 일부가 어떻게 등록되는지 살펴봐요. 특히 모델과 모델의 프로퍼티에 집중해요.

## 모델 [#model]

> [!NOTE]
> 코드 참조
>
> - `We`: `ClientModel` 데코레이터
> - `as`: `Model` 기본 모델 클래스
> - `re` `Vs`: `Issue` 모델 클래스
> - `rr.registerModel`: `ModelRegistry.registerModel`

![](https://github.com/wzhudev/reverse-linear-sync-engine/raw/45dc91e78e03300b69cc088057a28c42cee35a41/imgs/models.png)

LSE는 JavaScript의 `class` 키워드로 모델을 정의해요. 모든 모델 클래스는 기본 `Model` 클래스를 상속해요. 이 기본 클래스가 주는 핵심 프로퍼티와 메서드는 다음과 같아요.

- **`id`**: 모델마다 하나씩 붙는 고유한 UUID예요. 오브젝트 풀에서 모델을 찾는 키로 쓰여요.
- **`_mobx`**: 모델을 옵저버블(값이 바뀌면 구독자에게 알려 주는 값)로 만들 때 필요한 빈 오브젝트예요. 자세한 내용은 "관찰 가능성" 절에서 다뤄요.
- **`makeObservable`**: 관찰 가능성을 켜는 메서드예요. 모델은 생성될 때 기본적으로 옵저버블이 아니라서, 알맞은 시점에 이 메서드를 직접 불러야 해요.
- **`store`**: `SyncedStore`에 대한 참조예요. 뒤쪽 장에서 자세히 살펴봐요.
- **`propertyChanged`, `markPropertyChanged`, `changeSnapshot`**: 프로퍼티 변경을 추적하고 `UpdateTransaction`을 만드는 메서드예요.
- **그 밖의 것들**: 나머지 중요한 프로퍼티와 메서드는 뒤쪽 장에서 다뤄요.

> [!NOTE]
> 이 글을 쓰는 동안 Linear 팀이 프로퍼티를 저장하는 방식을 바꿨어요. `_mobx` 오브젝트를 없애고, 이제 모든 모델이 `__data` 프로퍼티에 프로퍼티 값을 저장해요. 이 변경은 일부 데코레이터의 구현과 하이드레이션 과정에 영향을 줘요. 하지만 LSE를 이해하는 데는 영향이 없어서, 이 글의 관련 부분은 고치지 않았어요.

모델의 메타데이터에는 다음이 들어 있어요.

1. **`loadStrategy`**: 모델을 클라이언트에 어떻게 불러올지 정해요. 전략은 다섯 가지예요.
   - **`instant`**: 애플리케이션 부트스트래핑 중에 불러오는 모델이에요(기본 전략).
   - **`lazy`**: 부트스트래핑 중에는 불러오지 않고, 필요할 때 한꺼번에 가져오는 모델이에요(예: `ExternalUser`).
   - **`partial`**: 필요한 만큼만 불러오는 모델이에요. 서버에서 인스턴스 일부만 가져온다는 뜻이에요(예: `DocumentContent`).
   - **`explicitlyRequested`**: 명시적으로 요청할 때만 불러오는 모델이에요(예: `DocumentContentHistory`).
   - **`local`**: 로컬 데이터베이스에만 저장하는 모델이에요. 이 전략을 쓰는 모델은 아직 찾지 못했어요.
2. **`partialLoadMode`**: 모델을 어떻게 하이드레이션(저장된 데이터로 모델 인스턴스를 채우는 일)할지 정해요. 값은 `full`, `regular`, `lowPriority` 세 가지예요.
3. **`usedForPartialIndexes`**: 부분 인덱스 기능과 관련 있어요.
4. 그 밖의 여러 가지

![](https://github.com/wzhudev/reverse-linear-sync-engine/raw/45dc91e78e03300b69cc088057a28c42cee35a41/imgs/count-of-models.png)

_이 글을 쓰기 시작했을 때 Linear에는 모델이 76개 있었어요. 글을 마칠 즈음에는 80개예요._

> [!NOTE]
> `local`은 어디에 쓸까요?
>
> 지금은 `local` 로드 전략을 쓰는 모델이 없는데, 이 전략을 어디에 쓰는지 궁금할 수 있어요.
> Tuomas는 Local First Conf 발표에서 서버 코드를 고치지 않고 새 기능을 개발하는 방법을 설명했어요. 제 추측으로는 이렇게 하는 것 같아요. 새 모델의 로드 전략을 처음에는 `local`로 두면 로컬 IndexedDB에만 저장돼요. 모델이 확정되면 로드 전략을 다른 전략 중 하나로 바꿔서 동기화를 켜는 거예요.

LSE는 **TypeScript 데코레이터**로 `ModelRegistry`에 메타데이터를 등록해요. 모델의 메타데이터를 등록하는 데코레이터는 `ClientModel`(`We`라고도 해요)이에요. `Issue` 모델을 예로 볼게요.

```tsx
re = Pe([We("Issue")], re);
```

원본 소스 코드는 아마 이런 모습일 거예요.

```typescript
@ClientModel("Issue")
class Issue extends Model {}
```

`ClientModel`은 이렇게 동작해요.

1. 모델의 이름과 생성자 함수를 `ModelRegistry`의 `modelLookup`에 등록해요.
2. 모델의 이름, 스키마 버전, 프로퍼티 이름을 합쳐서 **해시 값**을 만들어요. 이 값은 `ModelRegistry`에 등록되고, 데이터베이스 스키마를 확인할 때 쓰여요. 모델의 `loadStrategy`가 `partial`이면 이 정보도 해시에 포함해요.

`ClientModel`이 어떻게 동작하는지는 소스 코드에서 더 자세히 볼 수 있어요.

## 프로퍼티 [#properties]

> [!NOTE]
> 코드 참조
>
> - `vn`: `PropertyTypeEnum` 열거형
> - `w`: `Property` 데코레이터
> - `pe`: `Reference` 데코레이터
> - `A4`: `registerReference` 헬퍼 함수
> - `rr.registerModel`: `ModelRegistry.registerModel`
> - `rr.registerProperty`: `ModelRegistry.registerProperty`

모델에는 JavaScript 클래스 프로퍼티로 구현한 프로퍼티가 있어요. 프로퍼티마다 프로퍼티 메타데이터가 연결돼 있어요. 주요 필드는 다음과 같아요.

1. `type`: 프로퍼티의 타입을 정해요.
2. `lazy`: 모델이 하이드레이션될 때만 이 프로퍼티를 불러올지 정해요.
3. `serializer`: 데이터를 전송하거나 저장할 때 프로퍼티를 직렬화하는 방법을 정해요.
4. `indexed`: 프로퍼티를 데이터베이스에서 인덱싱할지 정해요. 참조에 써요.
5. `nullable`: 프로퍼티가 `null`일 수 있는지 정해요. 참조에 써요.
6. 그 밖의 여러 가지

`type`은 다음 값을 가진 열거형이에요.

1. **`property`**: 모델이 "소유하는" 프로퍼티예요. 예를 들어 `title`은 `Issue`의 `property`예요.
2. **`ephemeralProperty`**: `property`와 비슷하지만 데이터베이스에 저장하지 않아요. 거의 쓰지 않는 타입이에요. 예를 들어 `lastUserInteraction`은 `User`의 ephemeral 프로퍼티예요.
3. **`reference`**: 모델이 다른 모델을 참조할 때 쓰는 프로퍼티예요. 값은 보통 참조하는 모델의 ID예요. 참조는 지연 로딩할 수 있어요. 이 프로퍼티에 접근하기 전까지 참조하는 모델을 불러오지 않는다는 뜻이에요. 예를 들어 `subscription`은 `Team`의 `reference`예요.
4. **`referenceModel`**: `reference` 프로퍼티를 등록하면 `referenceModel` 프로퍼티도 함께 만들어져요. 이 프로퍼티는 getter와 setter를 정의해서, 짝이 되는 `reference`로 참조하는 모델에 접근해요.
5. **`referenceCollection`**: `reference`와 비슷하지만 모델 배열을 가리켜요. 예를 들어 `templates`는 `Team`의 `referenceCollection`이에요.
6. **`backReference`**: `reference`의 반대 방향이에요. 예를 들어 `favorite`은 `Issue`의 `backReference`예요. 가장 큰 차이는 `backReference`를 참조되는 모델이 "소유한다"고 보는 점이에요. 참조되는 모델(B)을 삭제하면 `backReference`(A)도 함께 삭제돼요.
7. **`referenceArray`**: 다대다 관계에 써요. 예를 들어 `Project`의 `members`는 `Users`를 참조하는 `referenceArray`예요. 한 사용자가 여러 프로젝트의 멤버가 될 수 있어요.

LSE는 프로퍼티 종류마다 다른 데코레이터를 써서 등록해요. 이 장에서는 그중 세 가지를 먼저 살펴봐요.

### `Property` (`w`) [#property-w]

`Issue` 모델을 예로 들게요. `priority`와 `title`은 `Issue`의 `property` 타입 프로퍼티로 선언돼 있어요.

```tsx
Pe([w()], re.prototype, "title", void 0);
Pe(
  [
    w({
      serializer: P_,
    }),
  ],
  re.prototype,
  "priority",
  void 0
);
```

원본 소스 코드는 아마 이런 모습일 거예요.

```tsx
@ClientModel("Issue")
class Issue extends Model {
  @Property()
  public title: string;

  @Property({ serializer: PrioritySerializer })
  public priority: Priority;
}
```

`Property`는 이렇게 동작해요.

1. `M1`을 불러서 프로퍼티를 옵저버블로 만들어요. `M1`은 [관찰 가능성](#observability-m1) 절에서 다뤄요.
2. 프로퍼티를 `ModelRegistry`에 등록해요.

자세한 내용은 소스 코드를 참고하세요.

### `Reference` (`pe`)와 `OneToMany` (`Nt`) [#reference-pe-and-onetomany-nt]

예를 들어 `assignee`는 `Issue`의 `reference`예요. 이슈 하나에 담당자는 한 명만 지정할 수 있기 때문이에요. 반대로 `assignedIssues`는 `User`의 `LazyReferenceCollection`이에요. 한 사용자가 담당하는 이슈는 여러 개일 수 있기 때문이에요.

```tsx
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

st([Nt()], K.prototype, "assignedIssues", void 0);
```

원본 소스 코드는 아마 이런 모습일 거예요.

```tsx
@ClientModel("Issue")
class Issue extends Model {
 @Reference(() => User, "assignedIssues", {
   nullable: true,
   indexed: true,
 })
 assignee: User | null;
}

@ClientModel("User")
class User extends Model {
  @OneToMany()
  assignedIssues: LazyReferenceCollection;

  constructor() {
    this.assignedIssues = new LazyReferenceCollection(Issue, this, "assigneeId", undefined, {
      canSkipNetworkHydration: () => this.canSkipNetworkHydration(Issue)
    }),
  }
}
```

`Reference` 데코레이터(정확히는 `registerReference` 함수)를 실행하면 프로퍼티가 실제로 두 개 등록돼요. `assignee`와 `assigneeId`예요.

두 프로퍼티는 타입이 달라요. `assignee`는 `referenceModel` 타입이고, `assigneeId`는 `reference` 타입이에요. `assignee` 프로퍼티는 데이터베이스에 저장하지 않고, `assigneeId`만 저장해요.

LSE는 getter와 setter로 `assigneeId`와 `assignee`를 연결해요. `assignee`에 값을 넣으면 `assigneeId`가 새 값의 `ID`로 바뀌어요. 반대로 `assignee`를 읽으면 그 `ID`로 데이터 스토어에서 해당 레코드를 가져와요.

그리고 `assigneeId`도 `M1`로 옵저버블로 만들어요.

![model property lookup](https://github.com/wzhudev/reverse-linear-sync-engine/raw/45dc91e78e03300b69cc088057a28c42cee35a41/imgs/model-property-lookup.png)

_`ModelRegistry`에는 `referenceModel`과 `reference`가 짝을 이룬 프로퍼티가 아주 많아요._

## 스키마 해시 [#schema-hash]

`ModelRegistry`에는 **`__schemaHash`**라는 특별한 프로퍼티가 있어요. 모든 모델의 메타데이터와 프로퍼티의 메타데이터를 해시한 값이에요. 이 해시는 로컬 데이터베이스를 마이그레이션해야 하는지 판단할 때 중요해요. 자세한 내용은 뒤쪽 장에서 다뤄요. 해시를 계산하는 방법은 소스 코드에 주석으로 이미 적어 두었기 때문에 여기서는 다시 설명하지 않을게요.

> [!NOTE]
> TypeScript 데코레이터
>
> TypeScript는 데코레이터를 트랜스파일할 때 프로퍼티 데코레이터를 모델 데코레이터보다 먼저 처리해요. 그래서 프로퍼티 데코레이터가 먼저 실행돼요. `ModelRegistry.registerModel`이 호출될 때는 그 모델의 프로퍼티가 모두 등록된 뒤라서, 프로퍼티의 메타데이터도 `__schemaHash`에 포함돼요.

## 관찰 가능성 (`M1`) [#observability-m1]

> [!NOTE]
> 코드 참조
>
> - `M1`: `observabilityHelper`

`M1` 함수는 모델과 프로퍼티를 옵저버블로 만드는 핵심 역할을 해요.

`Object.defineProperty`로 옵저버블이 되어야 하는 프로퍼티에 getter와 setter를 정의해요. 프로퍼티에 값을 넣으면 setter가 `__mobx`에 MobX box를 만들어야 하는지 확인하고, 그 box에 값을 넣어요.

getter에도 같은 논리가 적용돼요. box가 있으면 그 box에서 값을 가져와요.

React 컴포넌트를 `observer`로 감싸면 MobX가 어떤 컴포넌트가 어떤 옵저버블을 구독하는지 추적해요. 옵저버블 값이 바뀌면 그 컴포넌트를 자동으로 새로 그려요.

값을 넣을 때는 `propertyChanged` 메서드도 불러요. 이 메서드가 프로퍼티 변경을 이전 값과 새 값과 함께 등록해요. 이 변경 정보는 나중에 `UpdateTransaction`을 만들 때 쓰여요. `UpdateTransaction`은 3장에서 다뤄요.

더 자세한 내용은 소스 코드를 확인하세요.

## 1장 정리 [#takeaway-of-chapter-1]

이 장에서 다룬 핵심을 정리해 볼게요.

- **LSE의 모델과 프로퍼티**: 동작 방식을 정의하는 메타데이터가 이들을 좌우해요.
- **모델 정의**: LSE는 JavaScript 클래스로 모델을 정의하고, 데코레이터로 모델·프로퍼티·참조를 `ModelRegistry`에 등록해요.
- **로드 전략**: 모델은 `instant`, `lazy`, `partial`, `explicitlyRequested`, `local` 같은 여러 전략으로 불러올 수 있어요.
- **프로퍼티 타입**: LSE는 프로퍼티를 `property`, `reference`, `referenceModel`, `referenceCollection`, `backReference`, `referenceArray` 같은 여러 타입으로 나눠요.
- **반응형 데이터 처리**: LSE는 `Object.defineProperty`로 getter와 setter를 구현해서, 참조를 효율적으로 처리하고 관찰 가능성을 만들어요.

다음 장들에서는 이 메타데이터를 실제로 어떻게 쓰는지 살펴봐요. 계속 읽어 보세요.
