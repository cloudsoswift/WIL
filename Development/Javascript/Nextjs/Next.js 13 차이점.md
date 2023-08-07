# 서론
---
최근 진행중인 토이프로젝트에서 Next.js 버전 13을 사용하게 되었는데, 기존에 React 강의를 통해 배웠던 Next.js 12와 다른점(ex. App Router 사용, getServerSideProps 등)이 꽤 산재해 있는 것 같아 이를 정리하고자 합니다.  
참고로 어떤 점이 바뀌었는지는 Next.js의 블로그([Next.js 13 | Next.js (nextjs.org)](https://nextjs.org/blog/next-13)) 글을 참고했습니다.
# 본론
### App Directory (beta)
- Layouts RFC의 Feedback
- production 환경에서 에서 사용 하는것은 권장되지 않음.
	- `pages/` 디렉토리가 한동안은 지원이 계속 될 예정이고, `next/image`나 `next/link` 같은 발전된 컴포넌트 역시 사용할 수 있으므로 `pages/` 디렉토리를 사용하는 것이 권장됨.
- `app directory`는 다음과 같은 기능을 지원합니다.
#### Layouts
#### Server Components
#### Streaming
#### Support for Data Fetching
React의 최근 [Promise RFC에 대한 지원](https://github.com/acdlite/rfcs/blob/first-class-promises/text/0000-first-class-support-for-promises.md)은 Data를 Fetching하고 Promise를 다루는 새로운 방법을 제시했습니다.
이는 바로 React와 Next.js에서 Extend된 `fetch` API 입니다.  
`fetch` API는 fetch request를 자동으로 중복제거를 하고, 컴포넌트 수준에서 Data를 fetch, cache, revalidate 하는 하나의 유연한 방법을 제공합니다.
즉, `fetch` API를 사용하여 SSG(Static Site Generation, 정적 사이트 생성), SSR(Server-side Rendering, 서버 측 렌더링), ISR(Incremental Static Regeneration, 점진적 정적 생성)의 모든 이점을 사용할 수 있습니다.
##### 정적 사이트 생성(Static Site Generation)
###### 기존의 사용방법
###### `fetch` API 사용
```javascript
// This request should be cached until manually invalidated.
// Similar to `getStaticProps`.
// `force-cache` is the default and can be omitted.
fetch(URL, { cache: 'force-cache' });
```

##### 서버 측 렌더링(Server-side Rendering)
###### 기존의 사용방법
###### `fetch` API 사용
```javascript
// This request should be refetched on every request.
// Similar to `getServerSideProps`.
fetch(URL, { cache: 'no-store' });
```
##### 점진적 정적 생성(Incremental Static Regeneration)
전체 사이트를 rebuild 할 필요 없이, page 단위로 정적 생성을 사용할 수 있게 해주는 기능 입니다.  
빌드 시점에 pre-rendered된 page에 대해 첫 요청이 이뤄지면, 처음에는 cache된 page를 보여줍니다.  
첫 요청이 있은 후 ~ 10초 간의 모든 요청들 역시 cached page를 즉시 보여줍니다.  
10초가 지난 후에도 다음 요청은 여전히 cached page를 보여줍니다.  
이때, Next.js는 background에서 page의 재생성을 Trigger 한다는 점이 다릅니다. page가 재생성되고 나서는 Next.js가 cache를 무효화 하고, 업데이트 된 page를 보여줍니다.  
만약 background에서 재생성이 실패한다면, cached page는 변경되지 않습니다.  
generated 되지 않은 경로에 대해 요청이 이뤄지면, Next.js는 첫 요청에 대해 page를 server-render 합니다. 이후 요청들은 cache에서 정적 파일을 제공 받게 됩니다.
###### 기존의 사용방법
기존에 ISR을 사용하려면, `getStaticProps`에 `revalidate` prop을 넣어줘야 했습니다.
```javascript
export async function getStaticProps() { 
	const res = await fetch('https://.../posts');
	const posts = await res.json();
	return { 
		props: { 
			posts,
		},
		// Next.js는 다음과 같은 상황에서 page를 재생성하려고 시도합니다: 
		// - request가 들어올 때
		// - 최대 10초 마다 한 번씩
		revalidate: 10, // 초마다 한 번씩
	}
}
```
###### `fetch` API 사용
```javascript
// 이 요청은 10초의 lifetime으로 cache 되어야 합니다.
// 이는 `getStaticProps`를 `revalidate` prop을 설정하여 사용하는 것과 유사합니다.
fetch(URL, { next: { revalidate: 10 } });
```