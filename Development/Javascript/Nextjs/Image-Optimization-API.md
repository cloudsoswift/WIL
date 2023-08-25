# 서론
[The HTTP Archive의 웹 연감(Web Almanc)](https://almanac.httparchive.org/en/2022/page-weight#content-type-and-file-formats)에 따르면 이미지는 [일반적인 웹 사이트의 페이지를 무겁게 만드는데 큰 비중을 차지](https://almanac.httparchive.org/en/2022/page-weight#image-bytes)하며, [<code><b>LCP</b></code> 성능에 상당한 영향](https://almanac.httparchive.org/en/2022/performance#lcp-image-optimization)을 미칠 수 있습니다.
- **`LCP(최대 콘텐츠풀 페인트, Largest Contentful Paint)`** : 페이지가 <mark style="background: #FFF3A3A6;"><i>처음 로드를 시작한 시점</i></mark>을 기준으로, 페이지의 메인 콘텐츠가 로드 되었을 만한 시점(= *viewport내 <b><code>가장 큰 이미지 또는 텍스트 블록</code></b>의 렌더링 시간*)을 표시하는 메트릭. LCP가 빠르면 사용자가 <mark style="background: #FFF3A3A6;">페이지를 사용할 수 있다고 판단</mark>하는데 도움이 됨.
	- [Largest Contentful Paint(최대 콘텐츠풀 페인트, LCP) (web.dev)](https://web.dev/i18n/ko/lcp/)
그래서 Next.js에서는 HTML의 `<img>` 태그를 확장한 `<Image>` 태그를 제공합니다.  
`Image` 태그는 다음과 같은 기능들을 지원합니다.
- 크기 최적화(Size Optimization): WebP, AVIF와 같은 최신 Image Format을 사용하여 자동으로 기기에 맞는 크기의 이미지를 제공합니다.
- 시각적 안정성(Visual Stability): 이미지가 로딩 중일 때, 페이지 콘텐츠가 예기치 않게 이동하는 것을 자동으로 방지합니다.
- 더 빠른 페이지 로딩(Faster Page Loads): 이미지가 브라우저의 lazy loading을 이용해 viewport에 들어올때만 로드되며, 선택적으로 블러 처리된 placeholder를 사용할 수 있습니다.
- 자산 유연성(Asset Flexibility): 이미지를 원하는 해상도에 맞게 리사이징(On-demand Image Resizing)해서 사용자에게 전송합니다. (원격 서버에 저장된 이미지도 가능합니다.)

# 사용방법
---
```js
import Image from 'next/image'
```
`next/image`로 부터 `Image` 컴포넌트를 불러와 사용합니다.
## 로컬에 저장된 이미지
local에 이미지가 저장되어 있는 경우, 해당 이미지를 import한 뒤, `Image`의 `src`로 넘겨줍니다.  
```js
import Image from 'next/image'
import pic from './img.png';
...
<Image src={pic} />
```
`width`, `height`는 같은 값은 Next.js에서 [import한 파일의 값들을 토대로 자동으로 정의](https://nextjs.org/docs/app/building-your-application/optimizing/images#image-sizing)합니다. 그리고 이 값들은 이미지를 로딩하는 동안 페이지에서 레이아웃이 예기치 않게 바뀌는 것([Cumulative Layout Shift](https://nextjs.org/learn/seo/web-performance/cls))을 방지하는데에 사용됩니다.
## 원격 서버에 저장된 이미지
remote server에 이미지가 저장되어 있는 경우, src로 URL 문자열을 넘겨줘야 합니다.  
또한, Next.js가 Build 과정에서 원격 파일을 접근할 수 없으므로, `width`나 `height`와 같은 값은 수동으로 입력되어야 합니다. 
이때, 이미지의 `width` 및 `height`값은 *이미지의 종횡비(aspect ratio)를 유추*하고, *Layout Shift를 방지*하기 위해 사용되는 것이지, <mark style="background: #FFF3A3A6;">Render된 이미지의 높이와 너비를 결정하지 않습니다.</mark>
```js
import Image from 'next/image'
...
<Image src="https://..." width={500} height={500} ... />
```
#### (옵션)URL 패턴 정의
`next.config.js`에서 Image Optimization에 <mark style="background: #FFF3A3A6;">지원되는 URL 패턴 목록을 지정</mark>하므로써, 악의적인 사용을 방지할 수 있습니다.
```js
module.exports = {
	images: {
		remotePatterns: [ 
			{ 
				protocol: 'https',
				hostname: 's3.amazonaws.com', 
				port: '', 
				pathname: '/my-bucket/**', 
			}, 
		], 
	},
}
```
위와 같이 지정하면, AWS S3의 특정 bucket의 이미지만 허용합니다.