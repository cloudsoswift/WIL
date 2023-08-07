# 현상
- 현재 토이 프로젝트에서 다음과 같은 구조로 BE pjt 폴더 안에 FE pjt 폴더를 두고 개발을 진행하고 있었다.
> project
	ㄴ backend
		ㄴ frontend
		ㄴ...
- yarn을 사용하는 project(`project/backend/frontend/`)의 최상위 폴더(`project/`)를 VSCode Workspace로 열어버리면 project 내부에서 import문들이 먹통이 된다.
![[../../../attached/Pasted image 20230807005030.png]]
- import를 하지 못하니 당연히 React와 JSX들도 제대로 인식이 안 된다.
```javascript
JSX element implicitly has type 'any' because no interface 'JSX.IntrinsicElements' exists

Cannot find module 'next/image' or its corresponding type declarations.
```

# 해결
---
- Typescript를 사용하는 Project는 VSCode 설정에서 TS Server가 사용할Typescript SDK 위치를 `.vscode/settings.json` 에서 설정해줘야 하는데, 그러지 않아서 발생한 에러였다.
- 최상위 폴더(`project/`)에는 설정한 게 아무것도 없어서`.vscode/` 폴더도 생성이 되어있지 않았다.
	- 그래서 `project/backend/frontend`에서 `yarn dlx @yarnpkg/sdks vscode` 를 통해 `.vscode/`폴더 및 `./vscode/settings.json` 파일을 얻을 수 있다.
	- `setting.json` 파일에서 `typescript.tsdk`라는 property를 통해 Typescript SDK가 위치한 폴더를 지정해 줄 수 있다.
	- 현재 나는 `project/backend/frontend` 폴더에서 저런 setting 파일들을 만들었기 때문에 해당 위치가 기준으로 경로가 설정되어있다.
	- 하지만 나는 VSCode workspace를 `project/`로 잡아놨기 때문에  `project`부터 `frontend`까지의 경로를 추가해줘야 한다.
		- 즉, 기존 파일에서 `.yarn/sdks/typescript/lib`라고 되어 있었다면, `project`폴더에서 이를 적용하려면 `project/backend/frontend/.yarn/sdks/typescript/lib` 라고 해줘야 한다는 것이다.
- 이렇게 설정을 하면 error도 나오지 않고 잘 작동한다.
# 참고한 글
- [visual studio code - How to configure VSCode to run Yarn 2 (with PnP) powered TypeScript - Stack Overflow](https://stackoverflow.com/questions/65328123/how-to-configure-vscode-to-run-yarn-2-with-pnp-powered-typescript)