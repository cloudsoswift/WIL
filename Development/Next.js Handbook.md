
# 마주한 에러들
#### Error: The default export is not a React Component in page: "/"
원래 아래와 같이 코드를 작성했었는데 위와 같은 에러 발생함.
```js
export const MeetupPage = () => {
  return (
    <div>
    </div>
  );
};
```
index.js로 된 페이지의 경우 default export를 설정해줘야 하는 듯 함.
##### 수정한 코드
```js
const MeetupPage = () => {
  return (
    <div>
    </div>
  );
};
export default MeetupPage
```