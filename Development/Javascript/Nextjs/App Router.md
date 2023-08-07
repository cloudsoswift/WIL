기존의 File System 기반 Router에서 진화한 형태.
Next.js 버전 13에서, Shared Layout, Nested Routing, loading states, error handling 등을 지원하는 React Server Component에 기반하는 App Router가 등장함.
## 동작 방식
App Router는 `app`이라는 새로운 디렉토리에서 작동함.
`pages` 폴더를 사용하는 기존 방식의 `Pages Router`와 공존 가능.
##### 폴더와 File의 역할
- 폴더: Route를 정의하는데 사용. Route는 Nested Folder의 단일 경로로, File System 계층 구도 따라 내려가며 `page.js` 파일을 포함한 Leaf folder까지 내려감.
- 파일: Route Segment에 표시되는 UI를 만드는데 사용 됨.
##### Route Segment
- Route의 각 폴더는 `Route Segment`를 나타냄. 각 Route Segment는 URL Path의 Segment에 매핑됨.
- ex) ...com/dashboard/settings => `app\dashboard\settings\`

