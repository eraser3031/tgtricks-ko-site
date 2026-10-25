# 마치며

웹: /docs/linear/06-conclusion/ · Markdown: /docs/linear/06-conclusion.md

> 전체 내용을 정리하고, 액션과 계산 값에 관한 부록, 면책 조항, 감사의 말을 담았어요.

> 이 문서는 Wenzhao Hu의 [Reverse Engineering Linear's Sync Engine](https://github.com/wzhudev/reverse-linear-sync-engine/blob/45dc91e78e03300b69cc088057a28c42cee35a41/README.md)을 한국어로 옮기고 말투를 다듬은 번역이에요. 원문과 번역 모두 [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/deed.ko) 라이선스를 따라요.

Linear 동기화 엔진을 파헤치는 여정을 여기까지 함께해 주셔서 정말 기뻐요. 이 글에서는 폭넓은 주제를 다뤘어요.

1. **모델 정의**: LSE는 데코레이터로 모델, 프로퍼티, 참조를 정의해요. 모델 메타데이터는 `ModelRegistry` 클래스에 저장되고, LSE 전반에서 널리 쓰여요.
2. **관찰 가능성**: LSE는 MobX로 모델을 옵저버블로 만들어요. 그래서 모델과 프로퍼티의 변경을 추적하고, 그에 맞는 트랜잭션을 만들 수 있어요. 관찰 가능성은 데코레이터로 더해요.
3. **부트스트래핑**: LSE는 세 가지 부트스트래핑을 지원해요. 그중 전체 부트스트래핑을 자세히 살펴봤어요.
4. **지연 로딩**: LSE는 지연 로딩하는 데이터를 필요할 때 하이드레이션해요. 이 과정을 관리하려고 부분 인덱스와 싱크 그룹을 어떻게 쓰는지 알아봤어요.
5. **동기화**: LSE는 트랜잭션으로 클라이언트와 서버를 동기화해요. 트랜잭션을 어떻게 만들고, 큐에 넣고, 서버로 보내는지 설명했어요. 캐시, 리베이스, 충돌 해결 같은 처리 방식도 함께 다뤘어요.
6. **실행 취소와 다시 실행**: LSE는 트랜잭션을 기반으로 실행 취소와 다시 실행을 지원해요.

이제 Linear 동기화 엔진이 어떻게 동작하는지 한결 또렷해졌길 바라요. 최대한 많은 내용을 담으려 했지만 아직 살펴볼 주제가 남아 있어요. 더 깊이 들어가고 싶다면 이런 주제를 추천해요.

1. **다른 트랜잭션 타입**: 이 글에서는 `UpdateTransaction`을 살펴봤지만, LSE에는 `CreateTransaction`, `DeleteTransaction`, `ArchiveTransaction` 같은 타입도 있어요. 이들은 어떻게 동작할까요? `onDelete`, `onArchive` 같은 메타데이터 필드는 트랜잭션에 어떤 영향을 줄까요?
2. **다른 부트스트래핑 타입**: 이 글에서는 전체 부트스트래핑에 집중했지만, LSE는 부분 부트스트래핑과 로컬 부트스트래핑도 지원해요. 전체 부트스트래핑과 어떻게 다르고, 언제 쓸까요?

여러분의 생각, 질문, 발견을 Issues에 남겨 주시길 적극 권해요. 의견을 기다릴게요.

## 부록 A: 액션과 계산된 값 [#appendix-a-actions-and-computed-values]

액션(`rt`)과 Computed(`O`)

`Issue`의 `moveToTeam`과 `parents`를 예로 들게요. 각각 `Action` 데코레이터와 `Computed` 데코레이터가 붙어 있어요.

```jsx
Pe([rt], re.prototype, "moveToTeam", null);
Pe([O], re.prototype, "parents", null);
```

원본 코드는 아마 이런 모습일 거예요.

```typescript
@ClientModel("Issue")
class Issue {
  @Action
  moveToTeam() {
    // implementation
  }

  @Computed
  get parents() {
    // implementation
  }
}
```

**액션(Action)**과 **computed**는 MobX의 핵심 기본 요소예요. 부트스트래핑 중에 MobX의 `makeObservable` API를 직접 불러서, `moveToTeam`과 `parents`를 옵저버블로 만들어요.

## 면책 조항 [#disclaimer]

이 연구는 오로지 학습 목적으로 진행했어요. 독자는 연구 결과로 Linear와 경쟁하는 소프트웨어를 만들면 안 되고, 이 글의 정보를 Linear의 시스템을 방해하거나 침해하는 데 써도 안 돼요. Linear 팀이 요청하면 이 저장소를 기꺼이 삭제할게요.

## 감사의 말 [#credits]

Tuomas Artman이 발표와 팟캐스트에서 LSE의 동작 방식을 아낌없이 공유해 주었어요. 감사드려요.

[@zxch3n](https://github.com/zxch3n), [@vincentdchan](https://github.com/vincentdchan), [@promer94](https://github.com/promer94)에게 특별히 감사해요. 소중한 리뷰를 해 주었어요.

---

© 2025 Wenzhao Hu. 이 저작물은 크리에이티브 커먼즈 저작자표시 4.0 국제 라이선스(Creative Commons Attribution 4.0 International License)에 따라 이용할 수 있어요. 라이선스 사본은 https://creativecommons.org/licenses/by/4.0/ 에서 볼 수 있고, 우편으로 요청하려면 Creative Commons, PO Box 1866, Mountain View, CA 94042, USA로 편지를 보내면 돼요.
