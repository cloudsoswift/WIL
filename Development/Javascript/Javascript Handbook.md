# Common
## Module
- 참고 Article
	- [CommonJS와 ESM에 모두 대응하는 라이브러리 개발하기: exports field](https://toss.tech/article/commonjs-esm-exports-field)
	- [Modules: ECMAScript modules](https://nodejs.org/api/esm.html#modules-ecmascript-modules)
- 모듈 형식 강제화 [출처](https://webpack.kr/guides/ecma-script-modules/#flagging-modules-as-esm)
	- CommonJS
		- `.cjs`로 **파일 확장자** 지정.
		- package.json (In Node.js)
			- `"type": "commonjs"`
	- ESM
		- `text/javascript` 또는 `application/javascript`로 MIME를 지정할 경우 ESM 유형으로 강제 적용 됨.
		- `.mjs`로 **파일 확장자** 지정.
		- package.json (In Node.js)
			- `"type": "module"`
- #CJS (CommonJS) - **동기**
	- 모질라의 Kevin Dangoor에 의해 2009년 시작된 프로젝트로, 당시 이름은 ServerJS
	- 모든 파일이 로컬 디스크에 있어 필요할 때 바로 불러올 수 있는 상황을 전제, 즉 *동기적으로 동작 가능한 Server-side JS 환경*을 전제로 함.
		- 모두 다운로드 되기 전에는 아무 것도 할 수 없는 상태.
	  - 브라우저에서는 CommonJS를 지원하지 않으므로, Transpiler와 함께 package되어야 함.
	- `require()` 및 `module.exports()` 사용.
- #ESM (ECMAScript Modules) - **비동기**
	- Client-Side JS에서 일반적으로 사용됨.
	- `import` 및 `export` 사용
	- import는 엄격하게 해석되어, 상대적 요청의 경우 파일 이름과 파일 확장자가 포함 되어야 함 ( ex. \**.js* 또는 \**.mjs* )
	- top-level body에서 `await` 키워드 사용 가능 -> 비동기
		- => 따라서 ESM -> CJS를 `import` 하는건 가능하지만, CJS -> ESM을 `require` 하는건 불가능.
- #AMD (Asynchronous Module Definition) - **비동기** [출처](https://github.com/amdjs/amdjs-api/blob/master/AMD.md)
	- Javascript 모듈과 해당 모듈의 Dependencies를 *비동기적*으로 로드할 수 있도록 모듈을 정의하는 메커니즘.
	- 모듈을 동기적으로 로딩할 경우 성능, 사용성, 디버깅 및 Cross-Domain Access 문제를 야기할 수 있는 브라우저 환경에 특히 적합함.
	- `define()`
		-  `define(id?, dependencies?, factory);`
		- AMD에서 정의하는 함수로, 자유 변수 또는 전역 변수로 사용 가능.
- #UMD (Universal Module Definition)
	- 모듈 작성 방식이 **CJS**와 **AMD** 두 방식으로 나누어져서 호환 문제가 발생하자, 이를 해결하기 위해 나온 디자인 패턴.
	 - AMD와 CommonJS, window에 추가하는 방식까지 모두 가능한 모듈을 작성하는 방식.
## Closure [출처](https://developer.mozilla.org/ko/docs/Web/JavaScript/Closures)
함수가 선언될 당시의 Environment를 기억했다가 나중에 호출될 때 원래의 환경에 따라 수행되는 것.
이름이 클로저인 이유는 함수 선언 시의 scope(lexical scope)를 포섭(closure)하여 이후 실행될 때 이용하기 때문

## 지양해야할 습관들  [출처](https://yozm.wishket.com/magazine/detail/1836/)
#### 1) let, const 대신 var를 사용하는 것
- 범위가 분명해짐(중괄호 사이).
- 전역 객체를 생성하지 않음.
- 동일 변수명을 다시 선언할 때 오류를 표시함.
#### 2) 주석을 사용해 코드를 설명하는 것
- 주석은 맥락을 설명하는 정도로만 사용
- 전문가처럼 주석을 작성할 수 있는 팁
	- 의견 중복하지 않기. 무슨 일을 하는지 쓰지 말고, 왜 하는지에 대해 작성하기.
	- 함수 / 변수 / 클래스 이름으로 설명하는 것이 장황한 주석보다 나음.
	- 최대한 많이 요약하기. 꼭 필요한 경우가 아니라면 문단 단위로 작성하지 말 것.
	- 주석 언어 사용에 대해 항상 일관성을 유지하기.
	- 시간이 지나도 주석은 수정되지 않아야 함.
#### 3) \=\=\= 대신 \=\=를 사용하는 것
- **일반 항등 연산자**(\=\=)는 피연산자가 유사한지 여부만 확인
- **완전 항등 연산자**(\=\=\=)는 피연산자의 타입과 값을 동시에 확인
#### 4) 옵셔널 체이닝을 사용하지 않는 것
- 해당 객체의 *모든 레퍼런스를 확인하지 않아도* 객체 **체인의 깊숙한 곳에 있는 속성값을 확인** 가능
- 존재하지 않는 **속성값에 접근**하려고 할 때 **오류를 방지**
#### 5) 매직 넘버와 매직 스트링을 사용하지 않는 것
- 매직 넘버(매직 스트링) -  코드에서 직접 사용되는 숫자 또는 문자열
	- 명확한 컨텍스트는 없지만 목적이 있는 값들.
	- => 상수에 할당하는게 가장 좋음.
#### 6) API 호출 오류에 대한 부적절한 처리
-  async/await 안에서 **try/catch를 사용**하여 에러를 처리해야 함.
#### 7) 객체를 단일 매개변수로 사용하는 것
- 객체에서 여러 속성값이 필요한 함수를 정의할 때, 단일 객체를 사용하는 것보다 여러 매개 변수를 사용하는 것이 좋음
- 이를 통해 얻을 수 있는 장점
	1.  함수에 필요한 매개변수를 정확히 명시할수록 코드를 읽는 것이 쉬워짐
	2. 함수를 테스트하기 쉽게 만들어 줌.
	3. 가비지 콜렉팅(사용한 객체를 메모리에서 지우는 행위) 혹은 불필요한 객체 매개변수 생성을 방지 -> 애플리케이션 성능 향상
	4. + 타입스크립트를 사용하면서 여러 개의 매개변수가 있는 경우 타입을 확인하고 오류를 방지하는 유형 검사 및 자동 추천의 이점
#### 8) 약어를 사용하지 않는 것
- `if (x !== "" && x !== null && x !== undefined) {...}`
  대신
  `if( !!x ) {...}`
  와 같이 코드 축약


  
# React 
- CRA with Typescript
	- `npx create-react-app my-app --template typescript`


# RxJS [#](https://yozm.wishket.com/magazine/detail/1753/)
## 공식 문서 [#](https://rxjs.dev/)
## RxJS 한번 배워보실래요? [#](https://yozm.wishket.com/magazine/detail/1753/)
### 비동기 작업 이벤트를 Array와 같은 Collection을 통해 처리 할 수 있도록 해주는 라이브러리
> Think of RxJS as Lodash for events.
> RxJS ~= Event용 Lodash
> *Observer 패턴* 과 *Iterator 패턴*을 결합하고, *함수형 프로그래밍*을 *컬렉션*과 결합해 이벤트 시퀀스를 관리하는 이상적인 방법에 대한 요구를 충족
### **Observable** 이라는 핵심 개념 사용
- **Observable**
	- `Array`와 `Promise`의 상위호환
	- Observable -> Promise 대체 가능.
		```javascript
		// Promise는 비동기 로직을 값으로 만들 수 있다.
		const fetchXXX = (props) => new Promise((resolve, reject) => {
		 fetch(url, props).then(res => resolve(res), err => reject(err))
		})
		
		// Observable은 Promise 대신 쓸 수 있다.
		const fetchXXX = (props) => new Observable(observer => {
		 fetch(url, props).then(res => observer.next(res), err => observer.error(err))
		})
		```
		- Promise -> Observable 대체 불가능.
			```javascript
			// 반대로 Promise는 Observable가 될 수 없다.
			const fromEvent = (target, type) => new Promise(resolve => {
			 // Promise는 여러개의 값을 받을 수 없다.
			 target.addEventListener(type, (event) => resolve(event))
			
			 // Promise는 종료 시 cleanup을 할 수가 없다.
			 return () => target.removeEventListener(type)
			})
			```
	- Array -> 비동기 다룰 수 없음. 대신 여러 개의 값 다룰 수 있음.
	- Promise -> 비동기를 다룰 수 있음. 대신 하나의 값만 다룰 수 있음.
	- Observable -> Array와 Promise의 역할 동시에 수행 가능.
		- 여러 값을 다루면서 비동기를 다룰 수있음
		- Array, Promise의 메서드들 가지고 있음.
### 예시를 든 설명
- Array와 Method를 활용해 선언적으로 프로그래밍 한 예시
	```javascript
	const points = [ [1,-1], [5,10], [10,-2], [-3,-5], [-10,9], ... ]
	const sum = points // 점들 중에서
	 .filter(([x, y]) => x > 0 && y > 0) // 이중 1사분면에 있는 값을 추려내
	 .slice(0, 5) // 5개만 골라서
	 .map(([x,y] => Math.sqrt(x*x + y*y)) // 원점과의 거리들의
	 .reduce((a, b) => a + b) // 총합
	```
- 위와 같은 로직인데, 배열이 아닌, '마우스를 통한 입력'에 대한 배열을 받는다면?
	```javascript
	points = [... click, ... click, ...click, ...click, ...]
	```
	- 위와 같은 배열에는 별도 라이브러리 없이는 *선언적으로 프로그래밍 하는게 불가능*.
- 그래서 이벤트를 다루는 새로운 객체인 `Observable`을 만들고 Array와 같은 메소드들을 추가하면 **이벤트를 선언적으로 Array를 다루듯이 만들 수 있는 라이브러리 Rx가 탄생**하게 됨.
	```javascript
	// click을 통해 Observable
	const clicks:Observable<MouseEvent> = fromEvent(window, "click")
	
	// fromEvent 코드는 어떻게 생겼을까?
	const fromEvent = (target, type) => new Observable(observer => {
	  target.addEventListener(type, (event) => observer.next(event))
	  return () => target.removeEventListener(type)
	})
	```
### RxJS 사용 예시
- 어떠한 사용자 시나리오에서 ~가 일어나면 ~를 해주세요 와 같은 요구사항이 있을때, 이러한 ~하면 ~의 구현을 *로직들의 조합*이 아닌 **값으로 처리**할 수 있도록 해줌.
	- 이를 통해 코드가 간결해지고 선언적으로 작성할 수 있게 됨.