# Input
## step
step 프로퍼티는 브라우저간 호환성 문제 때문에 한계가 있음.
- 크롬, 오페라 브라우저에서는 보통의 목적에 부합하게 작동(선택 단위를 제한)
- 파이어폭스, MS 엣지에서는 해당 목적대로 작동 X 
  (파이어폭스에서는 (type=time의 경우)seconds 입력하는 부분 추가 됨)
그래서 `datalist`로 대체해 사용.
- 근데 이 방식도 파이어폭스 에서는 작동 X.

# HTTP
## HTTP Request Method
#### `PATCH` [#](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/PATCH)
리소스의 일부를 수정 할 때 사용되는 메소드.
##### `PUT`과의 차이점 [#](https://developer.mozilla.org/ko/docs/Web/HTTP/Methods/PUT)
`PUT` 메서드는 리소스 전체의 완전한 교체만을 허용함, 즉 멱등적인 성질을 띔.
-> `PUT`은 지정된 Resource를 나타내는 데이터가 없다면 생성(`201`)하거나 대체(200 or 204)하게 됨.
반면 `PATCH` 메소드는 **멱등성**을 반드시 띄진 않지만, 띄는 경우도 있음.
`PUT`과 같이 모든 값을 덮어 씐다면 `PUT`과 같이 멱등성을 띄게 됨.
그래서 `POST`처럼 *다른 Resource에 부수효과*를 **미칠 수도 있음**.
##### 멱등성
동일한 Request를 **한 번 보내는 것** <-> **여러 번 연속해서 보내는 것** 이 **같은 효과**를 지니고 서버상의 Resource의 상태도 동일하게 남는 경우.
멱등성이 있어도 Response의 Status Code가 달라질 순 있음.
`GET /page1 HTTP/1.1` -> 여러 번 연속해서 호출해도 Client는 항상 같은 응답을 받음.
반면 `POST /add_row HTTP/1.1` 는 그렇지 않음. 여러 번 호출시 새로운 row들이 추가됨.

# Hyphen or Underscore [#](https://studiohawk.com.au/blog/dash-or-underscore-in-url-heres-how-its-affecting-your-seo/)
- 컴퓨터와 웹 크롤러는 Hyphen을 Spacer로 읽음.
	- 그러나 Underscore를 사용하면, 연결 기호 역할을 함.
	- 즉, Underscore를 쓰면 사람은 띄어쓰기란 것을 인식하지만, 컴퓨터는 그렇게 하지 못함.
- 또한, Underscore는 subfolders로만 사용 가능함.
	- 반면, Hyphen은 subdomains, subfolders, domain names로 사용 가능.
#### 참고 글
- [Google URL 구조 가이드라인 | Google 검색 센터  |  문서  |  Google for Developers](https://developers.google.com/search/docs/crawling-indexing/url-structure?hl=ko&visit_id=638269678343752941-4095266124&rd=1)
- 