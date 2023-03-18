# JWT & LocalStorage 를 이용한 Auto Login 구현
- 비록 디자인은 구리지만..! Authorization에 집중한 프로젝트
- 다양한 로그인 을 구현해본 웹 (Mocking API & Axios)<br/>
- Nav바를 통해 페이지 전환을 하는 웹 <br/>
- **Nav바 중 NeedLogin은 로그인이 되었을때만 접근가능한데 로직을 어떻게 설계해야할까?** 에 대한 답


<h3> 배포링크 보러가기 🔍 https://chanwoo-jeong.github.io/LoginProject/</h3>

## 📌 프로젝트 설명
### `진행 동기` 
 - 취준을 준비하던 중 wanted에서 로그인에 관련된 강의를 무료로 진행한다고 했다.
 - 커리큘럼을 보니 JWT , 세션 등 로그인에 대해서 깊이 배운다고 하여 신청하게 됐다.
 - 배*개발자 분이 강의해주시는데 그동안 혼자 독학했던 것보다 난이도가 꽤나 높아서 좋았다.
 - 총 4번의 강의를 진행했지만 배운 것들을 토대로 한번에 합쳐서 하나의 프로젝트로 만들어 보고싶었다.
 - 배운것들을 전체 종합하여 나만의 로그인 웹페이지를 제작해보기로 했다!

### `진행 기간` 
 - 2023.03.06 ~ 진행중 

### `사용기술`
- Typescript 기반 React , Styled-Component , Recoil , Router V6
- Jwt , LocalStorage

### `기능설명`
- 왼쪽 Nav 바를 통해 page를 이동할 수 있다. <br/>
- Nav 바 중 NeedLogin은 Login해야만 접근할 수 있다.<br/>
- ""님 환영합니다 는 localStorage login을 하면 바뀐다.<br/>
  <img style="margin-top:10px;" src="./src/assets/images/main.png" >
- login에는 일반로그인 , JWT로그인 , JWT & localStorage Login 3가지 옵션이 있다.<br/>
- localStorage login을 하면 로그인 정보가 저장되며 NeedLogin 페이지에 접근가능하다.<br/>
- Logout을 누르면 localStorage에 저장됐던 정보가 삭제되며 로그아웃 , 페이제접근 불가능하다.<br/>
  <img style="margin-top:10px;" src="./src/assets/images/Memory.png" >
  <img src="./src/assets/images/JWT.png">
  <img src="./src/assets/images/local.png">
  <img src="./src/assets/images/logined.png">
  <img src="./src/assets/images/Logout.png">

### `성장한점`
- Router v6 Outlet 을 이용해 페이지 전환시 전체 Layout 재랜더링이 아닌 내부 Page만 재랜더링할 수 있다. <br/>
- Jwt , refresh_Token , access_Token , LocalStorage에 대한 개념을 이해했다. <br/>
- page별 접근가능 권한을 Nav바 Auth를 통해 저장했다. <br />이에 따라 Nav바 확장성과 Auth를 통해 로그인이 필요한 페이지의 접근을 제어할 수 있게 되었다.

  ```javascript
    const NavPart = [
    {
        part: "PageA",
        pathname: `/PageA`,
        Auth: false
    },
    {
        part: "PageB",
        pathname: `/PageB`,
        Auth: false
    },
    {
        part: "PageC",
        pathname: `/PageC`,
        Auth: false // 로그인 필요없음 => false
    },
    { 
        part: "NeedLogin", 
        pathname: `/needLogin`, 
        Auth: true // 로그인 필요함 => true
    }
    ];
  ```
- Promise async , await 를 통해 로그인 정보를 비동기적으로 불러올 수 있다.
  ```javascript
    export const loginWithToken = async (
       args: LoginRequest
    ): Promise<LoginResultWithToken> => {
        
        const loginRes = await axios
        .post(`${BASE_URL}/auth/login`, {
            username: args.username,
            password: args.password,
        })
        .then((Response) => {
            return Response.data.access_token;
        })
        .catch((Error) => {
            console.log(Error);
        }) 
        if (loginRes) {
        return {
            result: "success",
            access_token: loginRes.access_token,
        };
        }
        return {
        result: "fail",
        access_token: null,
        };
    }
  ```
- 로그인을 하게되면 Recoil을 통해 로그인 여부를 전역으로 관리할 수 있게 되었다. <br/>또한 로그인이 해제되면 NeedLogin 페이지의 접근을 Recoil을 통해 즉각 통제한다.
  ```javascript
    import { atom } from 'recoil';

    export const isLogined = atom({
    key: 'isLogin',
    default: false,
    });
  ```
- localstorage 를 이용하여 발급한 token값을 저장 , 불러오기 , 삭제할 수 있다.
 ```javascript
    export const saveAccessTokenToLocalStorage = (accessToken: string) => {
    localStorage.setItem('accessToken', accessToken)
    }

    export const getAccessTokenFromLocalStorage = (): string => {
    return localStorage.getItem('accessToken') || ''
    }

    export const DeleteAccessTokenFormLocalStorage = () =>{
    return localStorage.removeItem('accessToken')
    }
  ```

  ## 📌트러블슈팅 & 깨달은점

  ### `FormData 란 ??`
  - 지금까지 인강에서 배울때는 input & useState & onChange 조합으로 회원가입을 구현했다. <br/>
  - 그러나 이렇게 되면 onChange를 할때마다 재랜더링이 일어나 비효율이 생기지 않을까? 라는 의문이 들었다.<br/>
  - 찾아보니 그래서 formData를 이용한다고 했다. 이것을 이용하면 재랜더링 없이 input value 값을 받고 보낼 수 있다.<br/>
  ```javascript
  const formData = new FormData(event.currentTarget);

    const loginPayload = {
      username: formData.get("username") as string, // blue
      password: formData.get("password") as string, // 1234!@#$  
    };

    const loginResult = await loginWithToken(loginPayload);
  ```

### `useCallback 이란 ??`
  - 컴포넌트가 랜더링 되면 컴포넌트 안 선언되어있는 함수들도 재선언이 된다.
  - 그러나 이럴필요가 있을까? 어차피 함수는 변하지 않을거고 한번 선언된 함수를 재사용하고 싶다.
  - 그럴땐 useCallback을 통해 함수를 넣어두면 컴포넌트가 재랜더링되도 함수는 재랜더링되지 않는다!
  ```javascript
  ...(중략)

  const getUserInfo = useCallback (async () => {
    const userInfo = await getCurrentUserInfo()
    setUserInfo(userInfo)
    isDataFetched.current = true
  }, [])

  ...(중략)
  ```

