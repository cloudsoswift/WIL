# Third-party Package 사용하기
- Server Component가 나온지 얼마 되지 않아서, React 생태계의 서드파티 패키지들은 `"use client"` 지시문을 `useState`, `useEffect` 같은 Client 전용 기능을 사용하는 컴포넌트에 추가하기 시작했으나, 아직도 Client 전용 기능을 쓰지만, `"use client"` 지시문을 갖고 있지않은 컴포넌트들이 많습니다.
	- 이러한 컴포넌트들은 Client Components에선 예상대로 동작하지만, Server Components에서는 동작하지 않습니다.
###### Client Component
```typescript
'use client'
 
import { useState } from 'react'
import { Carousel } from 'acme-carousel'
 
export default function Gallery() {
  let [isOpen, setIsOpen] = useState(false)
 
  return (
    <div>
      <button onClick={() => setIsOpen(true)}>View pictures</button>
 
      {/* Works, since Carousel is used within a Client Component */}
      {isOpen && <Carousel />}
    </div>
  )
}
```

###### Server Component
```tsx
import { Carousel } from 'acme-carousel'
 
export default function Page() {
  return (
    <div>
      <p>View pictures</p>
 
      {/* Error: `useState` can not be used within Server Components */}
      <Carousel />
    </div>
  )
}
```

이러한 차이는 'Next.js가 해당 컴포넌트가 *Client 전용 기능을 사용하는지* **알 수가 없다**'는 점에서 기인합니다.
### 해결 방법
이러한 문제를 해결하려면, *Client 전용 기능에 의존*하는 <mark style="background: #FFF3A3A6;">서드파티 패키지들의 Component들을 <b>Wrapping</b>한 Client Component</mark>를 만들면 됩니다.
```tsx
// app/carousel.tsx
'use client'
 
import { Carousel } from 'acme-carousel'
export default Carousel

// app/page.tsx
import Carousel from './carousel'
 
export default function Page() {
  return (
    <div>
      <p>View pictures</p>
	  {/*  Carousel이 Client Component가 되어 잘 작동합니다. */}
      <Carousel />
    </div>
  )
}
```
