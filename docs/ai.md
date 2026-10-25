# AI 연동

웹: /docs/ai/ · Markdown: /docs/ai.md

> llms.txt와 페이지별 Markdown으로 AI 에이전트가 기법 리포트, 시스템 설계, 렌더링과 미디어 하위 문서를 읽을 수 있어요.

이 사이트는 AI 에이전트가 읽을 수 있는 정적 파일을 진입점으로 준비해 뒀어요. 진입점은 모두 읽기만 할 수 있고, 로그인도 필요 없어요. 웹 페이지와 같은 콘텐츠 소스에서 빌드할 때 만들기 때문에, 웹 페이지와 내용이 똑같아요.

## 진입점 한눈에 보기 [#endpoints]

| 진입점 | 경로 | 내용 |
| --- | --- | --- |
| llms.txt | /llms.txt | 사이트 색인: 전체 페이지의 제목, 링크, 요약 |
| llms-full.txt | /llms-full.txt | 전체 페이지의 Markdown 전문 |
| 페이지별 Markdown | /docs/<경로>.md | 페이지 주소 끝의 슬래시를 빼고 .md를 붙여요. 예: `/docs/rea.md`, `/docs/system/08-postbox-storage.md` |

경로는 이 사이트의 도메인이 기준이에요.

웹 페이지의 인터랙티브 컴포넌트는 Markdown에서 표와 목록으로 펼쳐져요. 예를 들어 기법 색인 페이지의 Markdown은 기법 300개를 모두 표로 보여 주고, 각 항목은 해당 장의 앵커(`[#...]` 형태의 제목 표식)로 연결돼요.

## 사용 예 [#usage]

에이전트에게 사이트 전체 맥락을 주는 방법은 두 가지예요.

- **필요한 장만 읽히기**: `/llms.txt`를 먼저 읽히고, 그다음 필요한 장의 `.md` 주소를 골라 읽히세요.
- **전부 한 번에 넣기**: `/llms-full.txt`를 쓰세요. 이 파일은 크기가 커서 컨텍스트 창(모델이 한 번에 읽을 수 있는 분량)이 넉넉한 모델에 알맞아요.

기법 하나를 찾을 때는 이렇게 해요.

1. [기법 색인](/docs/explorer/)에서 번호와 장을 확인해요.
2. 해당 장의 Markdown에서 `[#t01-01]` 같은 앵커로 항목을 찾아요.

시스템 설계와 렌더링과 미디어 하위 문서에는 항목 번호가 없어요. 그래서 `/llms.txt`에서 장을 고른 뒤 장 전체를 읽어요. 예: `/docs/system/08-postbox-storage.md` 또는 `/docs/media/01-lottie-pipeline.md`.

## 페이지의 버튼 [#page-actions]

모든 페이지의 제목 아래에는 버튼이 두 개 있어요.

- **Markdown 복사**: 이 페이지의 Markdown을 클립보드에 복사해요.
- **AI에서 열기**: 이 페이지를 Markdown으로 보거나 ChatGPT, Claude, Cursor, Scira AI에서 열어요. 열 때 AI가 먼저 이 페이지 주소를 읽도록 요청하는 문장도 함께 보내요.

## 구현 설명 [#implementation]

| 부분 | 구현 |
| --- | --- |
| 사이트 | Fumadocs와 Next.js 정적 내보내기(`output: 'export'`). 서버 없이 정적 호스팅으로 배포해요 |
| llms.txt와 전문 | 빌드 전에 번역 원고에서 만들어 `public/llms.txt`, `public/llms-full.txt`로 내보내요 |
| 페이지별 Markdown | 같은 원고를 `public/docs/<경로>.md`로 내보내요. 정적 파일이라 HTML 페이지(`/docs/<경로>/`)와 경로가 겹치지 않아요 |
| 검색 | Fumadocs 정적 검색. 빌드할 때 색인을 `/api/search`로 내보내고 브라우저에서 질의해요 |
