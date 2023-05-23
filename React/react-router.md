# 강의
## 개념
- SPA(Single Page Application) 구현을 위해 route(URL)에 따라 다른 *컴포넌트를 선택적으로(조건부) 렌더링*
- `react-router-dom` 에서 `<Route />` 컴포넌트 가져와서 렌더링
	- 해당 컴포넌트는 URL에 따라 로드 되어야 하는 React Component를 정의하는 컴포넌트
	- **props**
		- `path`
			- url이 기본 도메인 + a 라고 했을 때, a 에 해당하는 부분 기입.
			- React-Router는 URL을 평가해, URL 기반으로 올바른 컴포넌트 렌더링.
			- 아무 path도 활성화 된 것 없으면 화면에 아무것도 렌더링 X
		- `element`
			- JSX로 렌더링 될 컴포넌트를 할당.
			- `element={<컴포넌트이름 />}`
- 그리고 html에 `<App />` 컴포넌트를 render하는 부분(`App.js`)에 `<BrowserRouter>` 라는 컴포넌트 불러와서  `<App />` 컴포넌트 Wrapping 해줘야 한다.
## 페이지 이동
## 방법
#### `<a>` 태그 사용
- 이동 가능은 하지만, 매 번 새롭게 페이지(HTML 파일)를 요청한다.
	- 이러면 매 번 새로운 Application을 실행하는 셈이라 SPA의 장점도 퇴색되고, React가 관리하고 있던 state도 다 날아가게 된다.
#### `<Link>` (from `'react-router-dom'`) 컴포넌트 사용.
- props
	- `to`
		- 실제 이동할 주소 할당.
		- 현재 Route에 상대적인 주소 적어줘야 함.
- 화면을 유지하되, router에서 수동으로 URL 업데이트 해줌
#### + `<NavLink>` 라는 컴포넌트도 있음.
- props
	- `className`
		- 함수도 넘겨줄 수 있음.
		- 해당 함수에서는 해당 링크와 네비게이션 상태를 알아낼 수 있다.
			- Router에 의해 NavLink를 평가할 때나, 네비게이션 변경될 때 실행된다.
			- 인자로 받은 값.isActive(ex. `props.isActive`)를 통해 활성화 되어 있는지 확인해 처리한다.
## 동적 라우트
- Route를 통해 이동(컴포넌트 변경)시, 추가 값 넘기고 싶은 경우에는?
- => 동적 Path Segment 사용.
	- **:세그먼트 이름**
		- 예시)
		 ```jsx
		 <Route path='/product-detail/:productId'>
			 <ProductDetail />
		 </Route>
		```
		- `... path='/링크/:세그먼트이름'` 과 같이 사용
		- 그리고 해당 링크로 이동해 렌더링 된 컴포넌트 안에서 동적 Segment에 접근 가능.
#### 동적 Path Segment(= Path Variable) 사용하기
- `useParams` (from `'react-router-dom'`)훅 사용
	- 키-값 쌍을 가지는데, '키'는  페이지에 연결된 동적 세그먼트를 의미.
		- params.*동적세그먼트이름* 의 형태로 사용
		- ex) 위의 경우. 
			```jsx
			const params = useParam();
			console.log(params.productId) 
			```
## 중첩되는 링크
- 버전 5까지만 해도 exact 없으면 중첩되는거 다 렌더링 했는데, 6에서는 기본으로 exact 적용됨. 
	- 경로의 시작부분 일치하면 활성화 시키는건 이제 경로 뒤에 * 추가해주면 됨.
		- ex) `path='product/*'`
		- 경로 시작 부분 일치해도, 해당 경로대로 추축 해서 최선의 적합한 경로 찾아서 렌더링함.
		- 이제 ***순서도 상관 없어진 것***임.
			- 심지어 product/edit , product/:id 같이 중첩으로 인식할만한 것도, router에서 알아서 해줌
- Nested Route Handling
	- Route로 할당 된 페이지(컴포넌트) 내에서도 URL에 따라 또 다른 별도의 조건부 Content 만들 수 도 있음.
		- 이러면, 해당 페이지(컴포넌트) 내에서 Route 컴포넌트를 사용하면 됨.
			- + Route 컴포넌트는 항상 Routes로 감싸야 함.
		- 그리고 중첩된 Route의 부모 Route의 Path 뒤에 \*이 붙어야 함.
			- ex) /welcome과 해당 path에 지정된 컴포넌트 안에 path가 /welcome/to에 해당하는 컴포넌트 존재시 /welcome에서 /welcome/*으로 바뀌어야 함.
				- + 중첩된 Route의 경우, 부모 Route의 경로에 상대적이게 됨.
					- => 위의 예시의 경우 /welcome 안의 "/welcome/to" 는 "/to"로 해줘야 함.
		- => Route를 정의하는 것은 어느 곳에서든 정의할 수 있음.
			- 그리고 해당 Route는 Route를 포함한 페이지(컴포넌트)가 활성화 되지 않은 경우, 평가되지 않음.
			- ex) '/welcome' 에 할당된 page 안에 '/welcome/to' 에 할당한 페이지 존재시, '/welcome/to' 로 이동하면 welcome 에 해당하는 페이지 + 안에 /to 에 해당하는 페이지 렌더링 됨.
		- 또는 Route 컴포넌트 안에 또 다른 Route 넣어주는 방식도 됨.
			- ![[Pasted image 20230117170506.png]]
			- 이 때, 해당 하위 컴포넌트가 들어갈 위치는 `<Outlet />`(from 'react-router-') 을 해당 부모 컴포넌트 내에서 지정해주면 알아서 하위 컴포넌트가 들어가짐.
## 사용자 Redirection
- 특정한 경로로 이동했을 때, 다른 경로로 Redirection 시키고자 함.
	- 해당 경로로 Route 만들어 놓고, 해당 Route element로  `<Navigate />` 컴포넌트(from 'react-router-dom') 만들고 `to` prop으로 redireciton 시킬 링크 할당해주면 해당 URL로 변경됨.
		- ![[Pasted image 20230117164905.png]]
- 알 수 없는 경로 ( Error 페이지 )
	- Switch에 설정된 Route의 목록의 제일 마지막(후순위)에 `path='*'` 인 Route 설정.
### 학습 해야 할 내용
#### useLoader, Loader
#### useFetcher, Fetcher
# 코딩하다 겪은 것.
- useNavigate when component not rendered
	- `You should call navigate() in a React.useEffect(), not when your component is first rendered` 라는 경고 뜸
	- 이건 그냥 경고만 뜨는거고, useEffect()의 side-effect 함수 내에서 검사하는게 아니라도, 
#### React-router, React-router-dom 차이
- react-router는 코어까지 들어있는 master 브랜치에 있는 라이브러리 입니다.
그리고 react-router-dom은 그 중에서 DOM이 인식할 수 있는 컴포넌트들만 뺀 라이브러리 입니다. 예를 들어 \<Link\>(a태그로 렌더링되는), \<BrowserRouter\>와 같은 컴포넌트들이 있습니다.