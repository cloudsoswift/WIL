# 개요
An intuitive Redux side effect manager.
Easy to manage, easy to test, and executes efficiently.
액션을 모니터링하고 있다가, 특정 액션이 발생하면 이에 따라 특정 작업을 하는 방식으로 사용합니다.
- 특정 작업이란, 특장 자바스크립트를 실행하는 것 일수도 있고, 다른 액션을 디스패치 하는 것 일수도 있고, 현재 상태를 불러오는 것 일수도 있습니다.
## 만들게 된 원인
### Redux Middleware의 필요성
“스토어(Store)의 상태 변화를 다루는 리듀서(Reducer)는 순수 함수로 작성해야 한다”라는 원칙 때문에 Redux Middleware가 필요해짐.
- Reducer에서 Side Effect 가진 함수 실행시 '순수 함수를 사용해야 한다'는 원칙을 위배하게 됨.
- 이를 막기 위해 미들웨어가 Side Effect를 가진 작업을 처리하도록 하고자 함.
### Redux Middleware
미들웨어는 Redux의 기능 확장하기 위한 수단으로, Dispatch 함수를 감싸는 역할 수행 함.
여러 미들웨어들은 서로 Chaining되고, 마지막으로 Chaining 된 함수가 Store의 Dispatch 함수로 주입 됨.
이 Dispatch 함수에 Action 객체 담아 호출시, 미들웨어 체인 거쳐가면서 작업 수행함.
모든 미들웨어 지나면 Reducer로 Action 전달되어서 State 업데이트 됨.
Redux Middleware로는 **Redux Promise Middleware**, **Redux Thunk**, **Redux Logger** 등이 있음.
## Redux Saga
### Declarative Effects(선언적 이펙트)
- redux-saga에서 Saga들은 **제너레이터 함수**를 사용하여 구현 됨.
	- Saga Logic을 표현하기 위해, **Effect**라는 [yield](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/yield) 일반 JS 객체를 생성함.
		- Effect는 Middleware에서 해석할 수 있는 정보를 포함한 객체. 미들웨어에게 *특정 작업*(ex. 비동기 함수 호출, store에 action dispatch하기 등)을 수행하라고 하는 명령어로 볼 수도 있음.
- Effect를 생성하기 위해서, '`redux-saga/effects`'에서 제공하는 함수를 사용해야 함.
- Saga는 Effect들을 다양한 방식으로 yield 할 수 있음.
	- 가장 쉬운 방법은 Promise를 yield 하는 것.
##### 예시
```javascript
import { takeEvery } from 'redux-saga/effects'  
import Api from './path/to/api'  
  
function* watchFetchProducts() {  
	yield takeEvery('PRODUCTS_REQUESTED', fetchProducts)  
}  
  
function* fetchProducts() {  
	const products = yield Api.fetch('/products')  
	console.log(products)  
}
```
- 위 예시에선 Generator 내에서 직접 Api.fetch()를 호출함.
	- Generator 함수 안에선, yield 오른편의 표현식은 평가된 후 caller에게 결과가 yield(반환?) 됨.
###### *만약 위 Generator를 테스트 하려면?*
```javascript
const iterator = fetchProducts()
assert.deepEqual(iterator.next().value, ??) // what do we expect ?
```
- Generator가 생성한 첫 번째 값을 확인하고자 하는데, 테스트 중 실제 서비스를 실행하는 것은 실용적이지 않으므로 Mock 함수로 대체 해야함.
	- 즉, 실제 요청을 보내지 않고 올바른 인수(여기선 `'/products'`)로 `Api.fetch`를 호출했는지만 확인하면 됨.
- Mock은 테스트를 어렵고, 신뢰하기 힘들게 만듦. 반면, 값을 반환하는 함수는 `equal()` 쓰면 되므로 쉬움.
- 올바른 함수, 올바른 인수를 사용해 호출하는지 확인해야 하는데, Generator 내부에서 직접 비동기 함수를 호출하는 대신 **함수 호출에 대한 설명**만 **산출**해낼 수 있음. 즉, 아래와 같은 객체를 얻을 수 있음.
```javascript
// Effect -> call the function Api.fetch with `./products` as argument  
{  
	CALL: {  
		fn: Api.fetch,  
		args: ['./products']  
	}  
}
```
- 다시 말해, Generator는 명령이 포함된 일반 객체를 산출하고, `redux-saga`는 해당 명령을 실행하고 실행 결과를 Generator에게 반환하는 작업을 처리함.
	- 이렇게 하므로써 Generator를 테스트 할 때 '예상되는 명령'을 산출된 객체(yielded Object)와 `deepEqual`을 통해 비교만 하면 됨.
 - 이러한 이유로, `redux-saga`는 비동기 호출을 하는 다른 방식을 제공함.
```javascript
import { call } from 'redux-saga/effects'  
  
function* fetchProducts() {  
	const products = yield call(Api.fetch, '/products')  
	// ...  
}
```
- 바로 `call(fn, ...args)` 함수를 사용하는 것.
	- Redux에서 Action Creator 써서 Action 설명하는 일반 객체 생성하는 것 처럼, 함수 호출을 설명하는 객체 만듦.
	- redux-saga는 함수 호출을 실행하고, generator 함수를 resolve된 response와 함께 재개, 진행 하는 작업을 처리함.
- 이를 통해 Redux 환경 외부에서 Generator를 테스트 하는것이 쉬워짐.
```js
// expects a call instruction  
assert.deepEqual(  
	iterator.next().value,  
	call(Api.fetch, '/products'),  
	"fetchProducts should yield an Effect call(Api.fetch, './products')"  
)
```
- 무언가를 Mocking 할 필요 없이 동등성 테스트만으로도 충분하게 됨.
- 이러한 선언적 호출을 통해, Generator를 반복하고 산출된 값에 대해 동등함 테스트만으로 Saga 내부 로직을 테스트 가능함.
	- *아무리 복잡한 연산 로직이라도!*
- `call`은 객체 메서드 호출도 지원하여, 다음과 같이 사용 가능함.
```js
yield call([obj, obj.method], arg1, arg2, ...) // as if we did obj.method(arg1, arg2 ...)
```