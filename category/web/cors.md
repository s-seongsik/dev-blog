---
layout: post
title: CORS란? 무엇이고 어떻게 사용하는것인가?
tags: [Api, Rest Api, Http]
comments: true
---

FastAPI로 REST API를 개발하던 도중 CORS라는 것을 접하게 되었다.   
  
회사에서 '프론트엔드'를 많이 다루다 보니 백엔드 관련 경험이 부족했다.  
  
이번 FastAPI를 개발하면서 여러가지 이슈와 개념들을 접하게 되었다.  
  
그중 하나가 CORS라는 개념이다. 

-------------

## CORS란 무엇인가?
> 교차 출처 리소스 공유(Cross-origin resource sharing, CORS), 교차 출처 자원 공유는 웹 페이지 상의 제한된 리소스를 최초 자원이 서비스된 도메인 밖의 다른 도메인으로부터 요청할 수 있게 허용하는 구조이다.  
  
사전정의 그대로 가져온 것이다. 교차 출처 리소스 공유?, 그리고 최초 자원이 서비스된 도메인 밖의 다른 도메인으로부터 요청할 수 있게 허용하는 구조?  
역시 정의대로 해석하려고하면 이해가 어렵다...

잘 정리된 블로그를 보고 이해를 도왔다.  
(https://beomy.github.io/tech/browser/cors/) 

-------------

## URL 구조
CORS를 이해기 위해서는 **출처(Origin)** 라는 개념을 알아야한다.  
먼저 URL 구조를 살펴보자면, 

- https://localhost:8080/user?page=1#Origin이란?  


| Protocol | Host | Port(생략가능) | Path | Query String | Fragment |   
| :------ |:--- | :--- | :------ |:--- | :--- |  
| https | localhost | 8080 | user | pase=1 | Origin이란? |  


* 출처란 URL 구조에서 Protocol, Host, Port를 합친 것을 말한다.

-------------

## 동일 출처 정책(Same-Origin Policy)의 개념과 장점과 단점
보통 API를 테스트할 때 Postman, swagger를 사용합니다. 해당 툴들을 이용할 땐 잘되다가, 브라우저를 통해 api를 호출하게 되면 CORS policy라는 오류가 발생할 때가 있다. 이거는 브라우저가 동일 출처 정책을 지키기 때문에 다른 출처의 리소스 접근을 금지하기 때문에 발생하는 것이다. 

**동일 출처 정책의 장점은** XSS나 XSRF등의 보안 취약점을 노린 공격을 방어할 수 있다.  
**동일 출처 정책의 단점은** 외부 리소스를 사용할 수 없다.

동일 출처 정책의 단점을 보완하기 위한 SOP의 예외 조항이 CORS이다.

-------------

## CORS의 동작원리
**단순 요청 방법(Simple request)**  
단순 요청은 API를 요청하고, 서버는 Access-Control-Allow-Origin 헤더를 포함한 응답을 브라우저에게 보낸다. 브라우저는 Access-Control-Allow-Origin 헤더를 확인해서 CORS 동작을 작동할 지 판단하는 원리
    
**단순 요청 방법(Simple request) 조건**
* 요청 메서드는 GET, HEAD, POST 중 하나여야만 한다.  
* Accept, Accept-Language, Content-Language, Content-Type, DPR, Downlink,  Save-Data, Viewport-Width, Width를 제외한 header를 사용하면 안 됩니다.  
* Content-Type 헤더는 application/x-www-form-urlencoded, multipart/form-data, text/plain 중 하나를 사용해야 합니다.
> 위의 2,3번 조건은 까다로운 편입니다. 2번 조건은 사용자 인증에 사용되는 Authorization 헤더를 사용하지 못하기 때문이고, 3번 조건은 대다수의 REST API들이 Content-Type으로 application/json를 사용하기 때문에 지키기 어렵다.  


**예비 요청을 먼저 보내는 방법(Preflight request)**  
GET, POST, PUT, DELETE 등의 메서드로 API를 요청하기 전에 OPTIONS라는 메서드를 통해 실제 요청을 전송할지 판단한다.

OPTIONS 메서드로 서버에 예비 요청을 보낸뒤, 서버는 예비 요청에 대한 응답을 Access-Control-Allow-Origin 헤더를 포함하여 브라우저에게 보낸다. 브라우저는 단순 요청과 동일하게 Access-Control-Allow-Origin 헤더를 확인 후 CORS를 동작할 지 판단하는 원리이다.

-------------

## FastAPI로 개발된 REST API로 CORS 테스트!
1. 현재 내가 위치한 브라우저는 'https://beomy.github.io' 이다. 즉 'https://beomy.github.io' 에서 API서버에 리소스를 요청한다고 생각해보자.  
![image](https://user-images.githubusercontent.com/52439201/138238315-0d29ecf0-f487-4015-8362-8ea4a8d92c53.png)

<<<<<<< HEAD
2. REST API  
=======

2. REST API
>>>>>>> 137f2bd80ce7a62f2f9859f7d812efdb6fff1457
(GET) localhost:8000/user 호출 시 아래의 리소스를 리턴
```
[
  {
    "username": "Rick"
  },
  {
    "username": "Morty"
  }
]
```

3. 현재 내가 위치한 브라우저의 출처에서 API의 리소스를 요청하면 아래와 같이 Access-Control-Allow-Origin 헤더가 요청리소스에 포함되어 있지 않다라는 오류가 뜬다.  
![image](https://user-images.githubusercontent.com/52439201/138239426-54668cc0-ca94-48ff-b5ad-266779257c38.png)

<<<<<<< HEAD
4. FastAPI 서버에서 응답헤더에 Access-Control-Allow-Origin와 출처를 추가해준다.   
* allow_methods=["*"]은 Access-Control-Allow-Methods POST, GET, PUT, DELETE 모두 허용이고 
* allow_headers=["*"]은 Access-Control-Allow-Headers 응답 헤더를 모두 허용하겠다는 의미이다.

![image](https://user-images.githubusercontent.com/52439201/138239252-f05afda6-16d2-4b93-834a-ee9badbfdd3b.png)

=======

4. FastAPI 서버에서 응답헤더에 Access-Control-Allow-Origin와 출처를 추가해준다.  
![image](https://user-images.githubusercontent.com/52439201/138239252-f05afda6-16d2-4b93-834a-ee9badbfdd3b.png)


>>>>>>> 137f2bd80ce7a62f2f9859f7d812efdb6fff1457
5. 다시 요청을 보내면 서버에서 리소스가 리턴되는 것을 확인할 수 있다.
![image](https://user-images.githubusercontent.com/52439201/138239564-b992115f-6595-4215-b933-8669731c288c.png)

6. 개발자 도구 > Network에서 응답 헤더를 확인해보면 서버에서 Access-Control-Allow-Origin 헤더를 브라우저에 보낸것을 확인할 수 있다!.  
![image](https://user-images.githubusercontent.com/52439201/138240176-adbc5c1d-7de3-41d2-a004-7f9753eecfaa.png)
<<<<<<< HEAD


## 끝으로
프론트엔드 개발자 입장에서 서버로 리소스를 요청할 때 CORS 에러가 발생한다면, 서버에 Access-Control-Allow-Origin 등 CORS를 해결하기 위한 몇 가지 응답 헤더를 포함해 달라고 요청한다는 것을 배웠다. 

Node.js, FastAPI 등의 대부분의 프레임워크에서 CORS 응답 헤더를 추가해 주기는 기능이 있어 간편하게 사용할 수 있지만, CORS가 무엇이고 해당 프레임워크의 지원이 없더라도 CORS 에러 문제가 발생할 때 발생원인과 어떻게 해결해야 되는지 알아보는 시간이었다.
=======
>>>>>>> 137f2bd80ce7a62f2f9859f7d812efdb6fff1457
