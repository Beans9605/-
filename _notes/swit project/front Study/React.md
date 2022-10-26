```toc
```

[[Front-End Study Index]]
프론트엔드 목차 페이지

# React 

## DOM ? Virtual DOM ?

### DOM

DOM은 MDN(Mozilla)에서는 DOM을 "HTML, XML document와 상호작용하고 표현하는 API이다, DOM은 브라우저에서 로드되며, 노드 트리로 표현하는 document 모델이다."라고 코멘트 하고있다.

<img src="/assets/Pasted_image_20221025073952.png" />
![](Pasted_image_20221025073952.png)

DOM의 가장 큰 특징은 노드 트리형태 즉 내가 구성한 화면은 각각 하위 노드가 있고 하위 노드는 상위 노드가 로드 되어야지 로드되는 트리의 형태를 가지므로 상위 컴포넌트가 가진 스타일 및 옵션을 선 반영하게 된다.
그 이후에 자기 자신 노드가 가진 스타일 및 옵션 적용하게 된다.

DOM Rendering의 가장 큰 특징은 모든 노드가 로드되어야지 페이지를 보여준다. 
<img src="/assets/Pasted%20image%2020221025074050.png" />
![](Pasted%20image%2020221025074050.png)

보시다 싶이 내가 제일 하위 노드를 수정한다고 하면, 그 노드 로드하기 위해 수많은 상위 노드들을 렌더링하여 형태를 만든 후 그 다음에 하위 노드를 랜더링하는 비효율적인 랜더링 방식을 취한다. top-down
(내 상위 노드들이 랜더링 돼야지 하위 노드가 랜더링할 수 있기 때문 - tree 구조)

### Virtual DOM

virtual DOM은 수정사항이 있더라도 한번만 렌더링하는 기술, 가상 메모리에 수정사항을 저장해서 재랜더링 할때 한번에 랜더링 할 수있도록 도와주는 기술을 의미.

보통의 DOM은 수정사항이 있으면 그 위의 노드들을 모두 랜더링해야하는 불편함이 있었다면 virtual dom은 그 수정사항을 모두 한번에 처리해주는 기술이므로 많은 작업을 해야 수정사항을 확인할 수 있는 dom보다 효율적으로 랜더링하게된다.
<img src="/assets/Pasted%20image%2020221025074527.png" />
![](Pasted%20image%2020221025074527.png)

가상 돔을 생성하여 그 안 가상 메모리를 사용하여 내부적으로 수정사항을 모두 처리하여 실제로 브라우저에서 런타임 했을 때 그 화면이 한번만 랜더링 하도록 하는 기술

React는 이 virtual dom 형식을 채택하여, 하나의 컴포넌트의 변경사항에 따라서 한번의 렌더링을 지원하고 있다. 이런 특징 덕에 한 페이지에서 interactive 한 페이지를 보여줄 수 있음.
<img src="/assets/Pasted%20image%2020221025074900.png" />
![](Pasted%20image%2020221025074900.png)


## SPA (Single Page Application)

SPA, 싱글페이지 애플리케이션은 하나의 페이지를 불러올때 html, js, css 요소들을 한번에 불러오는 방식을 말함. 페이지 또는 후속 페이지의 상호 작용은 서버로부터 새로운 페이지를 불러오지 않음. 즉 페이지가 다시 로드되지 않음.

React를 사용하여 SPA를 만들 수 있으나, react를 쓰면 무조건적으로 해야하는 필수 사항은 아님. 

# React 구조화

실무에서 React의 Component 폴더 및 구조화 단계에 대해서 설명하자면 아래와 같은 방법으로 사용함.

```tree
/src
├─components
│  └─common
├─config
├─containers
├─hooks
├─pages
├─static
├─store
└─utils
```

## Component

컴포넌트들은 http call 혹은 socket 통신 등 서버로 부터 데이터를 받아서 사용하지 않고, 보여지는 형태 (css 및 컴포넌트 내에서 동작하는 기본 기능) 만 구현되어있는 상태들을 이야기함, 그 중에서도 각각의 서비스에서 필요한 컴포넌트에 따라서 폴더로 재구분
1.  common    
    -   모든 서비스들이 공통으로 사용하는 컴포넌트
    -   서로 다른 컴포넌트 구분 없이 모두가 사용할 수 있는 컴포넌트의 집합

## config

공통으로 사용할 static 변수 및 action type들을 시리얼화 하고, 각각 action 및 return type에 사용할 수 있도록 한 파일들을 모아둔 폴더

### Containers

컴포넌트들을 활용하여 실제 http call 및 socket 통신으로 필요한 데이터를 서버로 부터 받아서 화면에 출력해주는 기능 구현된 파일들의 집합, Components 폴더와 마찬가지로 각각의 서비스에 필요한 컨테이너에 따라서 폴더로 재구분

### Hooks

React Hooks를 활용한 Custom hooks, 데이터 변화 감지에 따라서 실행시키는 jsx 파일들의 집합으로, debounce, useTitle 등등 각각 기능에 필요한 custom hooks 집합
<img src="https://velog.velcdn.com/images%2Fdenmark-choco%2Fpost%2F45c244b4-0e73-4662-b4ac-d10bebab15eb%2Fcc006f00-a420-11e9-99a6-d0bdf5f0c7bb.png" />
-   hook은 react functional에서 사용하는 class형에서 흔히 알려진 리액트 생명주기를 모방 및 개선점으로 등장한 기능
    
-   기존 class 형태의 react 생명주기는 컴포넌트가 mount 되기 전, mount 되는 도중, mount 된 후 순서대로 순차 렌더링 되는 형식으로 구성됐으나, 순차 실행에 대한 한계점에 따른 functional react hooks 가 등장함
    
-   hook은 이전 react 생명주기와 동일하게 mount 이전, mount 도중, mount 이후로 구분되어있지만, 변수를 감지하여, 감지된 변수가 새롭게 변화된 순간을 캐치하고 다시 렌더링 작업을 해주는 용도로 사용된다. ( 기존에는 감지 개념이 아닌 순차 실행 개념 )
    

### Pages

컨테이너들의 집합으로, 레이아웃 구성만 해주는 역할을 하는 jsx 파일, 화면 구성 시에 grid & flex 및 자유로운 레이아웃 구성을 하여, 컨테이너 및 컴포넌트의 재활용을 위해 존재함

### Static

정적파일 관리 디렉토리로 이미지나 css ( 차후 각 컴포넌트, container 등등에 있는 css를 모두 모아서 정적파일로 사용 예정 ), font를 저장해놓는 폴더

### Store

React에서는 모든 변수가 각각의 컴포넌트 내에서 생성되고 사용된 후에 컴포넌트가 닫히면 사라지는 지역 state 형태로 모든 데이터를 관리하고 있음. 그러나, 이런 관리 방법은 하위 컴포넌트가 다시 상위 컴포넌트에게 데이터를 callback을 통해 전달해줘야할 때 문제가 생김. 하위 컴포넌트가 많아지면 각각의 하위 컴포넌트가 해당 데이터를 물고 있고 다시 전달해줘야하므로 데이터 무결성이 보장되지 않을 수 있기 때문에 관리에 애로사항이 생김.

해당 문제를 타 언어에서는 전역변수로 해결하고 있으나, React의 경우는 Store라는 최상위 컴포넌트의 State 개념으로 저장하고 관리하게 됨

<img src="https://cdn.filepicker.io/api/file/eHSa386Q2qz4PUCDNmPA" />

해당 그림처럼 Store에 변수가 저장돼있고 그 저장된 변수를 State로 이동시킴 (지역 State) 그리고 그 스테이트를 변경하기 위해서 Actions에 존재하는 Store 변경 Custom function을 call하게 됨. 그렇게 불러온 custom function은 그 함수 내에서 reducer라는 Store 업데이트를 도와주는 파이프라인을 통해 Store를 변경하게 됨, 이렇게 변경된 Store 내의 State는 이 State를 가진 모든 컴포넌트들에게 영향을 주게 됨

이 복잡한 과정을 zustand라는 라이브러리로 한번에 해결
[
```javascript
import create from 'zustand'  
import { headerOptions } from '@/config/config'  
import axios from 'axios'  
import { getCookie, removeCookie } from '@/utils/cookies'  
 
//set과 get은 reducer  
const useAuth = create((set, get) => ({  
    // store  
    userId: "",  
    // action  
    userIdHandler: (value) => set(state => ({ userId: value })),  
      
    // action  
    auth: () => {  
        const request = axios.post('/api/com/CheckApiKey', `token=${getCookie("userToken")}`, headerOptions.content.urlencodedHeader)  
            .then(res => {  
                if (res.status === 200 && res.data.length > 0 && res.data[0].result?.flag) {  
                    return {  
                        result: true,  
                        userId: res.data[0].result.id  
                    }  
                }  
                else {  
                    return {  
                        result: false,  
                        userId: ""  
                    }  
                }  
            })  
        return {  
            payload: request,  
            type: "AUTH"  
        }  
    },  
    login: (userId, userPwd, adminRoll) => {  
        const request = adminRoll ?  
            axios.post('/api/login/LoginAdmin', `agencyId=${userId}&agencyPwd=${userPwd}`, headerOptions.content.urlencodedHeader) :  
            axios.post('/api/login/LoginUser', `userId=${userId}&userPwd=${userPwd}`, headerOptions.content.urlencodedHeader)  
        return {  
            payload: request,  
            type: "LOGIN"  
        }  
    },  
    logout: () => {  
        get().userIdHandler("")  
        removeCookie("userToken")  
        removeCookie("autoLogin")  
        removeCookie("admin")  
    }  
}))  
export default useAuth
```]()


```

1.  auth.js
    
2.  common.js
    
3.  configStore.js
    

### utils

컴포넌트 외의 필요 기능으로, 쿠키, 세션과 같이 브라우저 단에서 관리하는 관리 코드 및 axios, socket 통신 및 네트워크 통신 시에 필요한 여러가지 기능을 미리 설정 해주는 코드를 저장해두는 폴더 프로젝트 전반에 사용하는 필요 기능을 저장하는 폴더

Hooks는 리액트 16.8 에 새로 도입된 기능 함수형 컴포넌트에서도 상태 관리를 할 수 있는 state, 렌더링 직후 작업을 설정하는 effect 기능들을 통해 작업 다양성 추구

## State Hook
### useState

비동기적 상태 관리에서 동기 상태 관리와 충돌하는 경우

```javascript
function App() {

	const [count, setCount] = useState(0)

	const plusCount = () => {
		setCount(count + 1)
	}

	const asyncPlusCount = () => {
		setTimeout(() => {
			setCount((prev) => prev+1)
		}, [2000])
	}

	return <div>
		{ count }
	</div>
}
```

state는 말 그대로 상태이기 때문에, setCount를 할때 참조되는 값이 count의 현재 상태가 아닌 함수를 실행할때 그때의 값을 참조하여 사용되기 때문에, setCount를 할 때의 시점의 state value를 바로 가져와 쓸 수 있는 내부 함수를 통해서 변경해야함.

내부 함수의 구조에 대해서 자세히 설명하자면 아래와 같다.

```javascript

const [count, setCount] = useState(0)

const asyncPlus = () => {
	// prestate라는 함수의 인자로 받는 것은 이 setState하기 직전 state 값을 가져오겠다는 인자로써 사용됨, 코드 동작 시점에 따라 맞춰서 이 직전 state를 가져와 사용함으로 비동기에도 유용하게 동작
	setCount((prestate) => prestate + 1)
}
```
비동기 뿐만 아니라 array, object 등 수정하기 까다로운 데이터를 새로 얉은 복사, 즉 메모리를 새로 할당하여 사용하지 않아도 쉽게 처리 가능하게끔 만듦

# 기타 개발 이슈

## Tree Shaking
#개발이슈 #javascript #tech #speed #improvement

### 개요
현 시점의 웹 애플리케이션들은 굉장히 크고, 대부분 자바스크립트로 만들어져있음, 18년 중순 HTTP Archive가 보여준 모바일 장치에서 자바스크립트의 평균 전송 크기는 350KB, 하지만 단순한 전송 크기일 뿐, 실제로 전송크디는 300KB로 압축돼서 오지만 파싱되고 컴파일 및 실행하면 900KB까지 증가 할 수 있다.
자바스크립트는 수행하는데 비용이 많이 듦, 따라서 자바스크립의 성능 개선을 위한 기술이 여러가지 있다.

코드 스플리팅(https://webpack.js.org/guides/code-splitting/), 자바스크립트 청크로 애플리케이션을 분할하고, 청크를 필요로 하는 애플리케이션의 경로에만 이 청크를 배분하여 성능을 개선하는 기술이 있다.

하지만 이 코드 스플리팅이 근본적인 원인을 해결해주진 않는다. 사용되지 않는 코드를 포함한 무거운 자바스크립트 애플리케이션의 일반적인 문제를 해결하기 위해서는 **트리 쉐이킹**이라는 기술을 사용해야한다.

### 본문
#### 트리 쉐이킹이란?
트리 쉐이킹은 사용하지 않는 코드를 제거하는 방식, (이 용어는 [[#Rollup]]에 의해 부상함)
트리 쉐이킹이라는 용어는 애플리케이션의 멘탈 모델(mental model)과 디펜던시 트리 구조에서 유래됐음. 트리 내 각 노드들은 앱을 위해 특징적인 기능들을 제공하는 디펜던시들을 나타냄,

``` javascript

import arrayUtils from "array-utils"

```

현 앱에서는 이러한 디펜던시들을 import (정적구문)으로 가져올 수 있음.
이렇게 모든 기능에 대한 디펜던시를 걸린 유틸 전체를 불러오게된다면, 나의 array-utils가 가진 모든 기능에 대한 코드를 불러오게 됨. (사용하지 않는 것을 포함한)

``` javascript

import { unique, implode, explode } from "array-utils"

```

거의 모든 다른 언어에서도 코드를 작성할때 권장하는 방식으로, 내가 사용할 라이브러리 코드만 따로 불러와서 사용하는 방식이다.
개발 빌드에서는 어떤 것도 설정하지 않았기 때문에 이렇게 작성하여도 import 된것과 상관 없이 전체 모듈을 불러옴
허나 프로덕션 빌들에서는 명시적으로 import 되지 않은 ES6 모듈로부터 export를 shake 하기 위해서 webpack을 설정하고 빌드 결과물 크기를 더 작게 할 수 있음.

## Rollup
### 개요
롤업은 웹팩과 마찬가지로 자바스크립트 번들러의 종류
- 자바스크립트는 실제로 불러와서 컴파일을 하려면 여러 개의 자바스크립트 파일을 하나로 만들어서 사용해야함, 이 작업을 사람이 일일히 할 수 없으므로 자바스크립트 파일들의 의존성과 연결성을 충족하는 하나의 파일로 만들기 위해서 웹팩이라는 자바스크립트 번들러가 등장함.
- react 역시 webpack으로 많은 js, jsx 파일들을 하나의 js 파일로 빌드하여 실제 동작하게끔 함.
- 수 많은 라이브러리를 연결하는 일은 굉장히 괴로운 것
롤업은 웹팩과 다르게 빌드 결과물을 ES6 모듈 형태로 만들 수 있음, ES6 모듈로 빌드가 가능하다는 것은 사용하는 쪽에서 라이브러리 전체를 불러오는 게 아니라 필요한 부분만 콕 집어서 가져올 수 있다는 특징 존재,


