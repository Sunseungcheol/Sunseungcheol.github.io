---
layout: post
categories: Web
tags: [Web, localstorage, cookie, javascript]
---
cookie 의 경우 예전에 재직중일때 자주 사용했지만 Web Storage 는 사용해 본적이 없었다.

먼저 cookie 는 persistent cookies와 session cookies 두가지가 있는데 쿠키를 구울 때 만료일을 지정하면 persistent cookies 이고 지정하지 않는다면 session cookies 이다. session cookies 의 경우 만료일이 따로 없이 브라우저, 탭이 열려있는 동안만 저장되고 닫을 때 자동으로 영구 삭제가 된다. 나는 보통 페이지에 오랜만에 접속하거나(쿠키에 만료일을 두어서) 첫 접속을 파악해 팝업등의 새로운 페이지를 여는데 사용했었다.

WWeb Storage 또한 두가지 종류가 있는데 sessionStorage 와 localStorage 이다. 두가지의 차이점은 이렇다.

sessionStorage는 각각의 출처에 대해 독립적인 저장 공간을 페이지 세션이 유지되는 동안(브라우저가 열려있는 동안) 제공합니다.

○ 세션에 한정해, 즉 브라우저 또는 탭이 닫힐 때까지만 데이터를 저장합니다.
○ 데이터를 절대 서버로 전송하지 않습니다.
○ 저장 공간이 쿠키보다 큽니다. (최대 5MB)

localStorage도 위와 같지만, 브라우저를 닫았다 열어도 데이터가 남아있습니다.
○ 유효기간 없이 데이터를 저장하고, JavaScript를 사용하거나 브라우저 캐시 또는 로컬 저장 데이터를 지워야만 사라집니다.
○ 저장 공간이 셋 중 제일 큽니다.


localStorage 의 경우 쿠키와 같이 도메인만 같으면 인식이 가능하지만 sessionStorage 는 다른 탭이나 브라우저의 값을 인식하지 못한다.

localStorage 는 MDN 문서에는 

`localStorage 읽기 전용 속성을 사용하면 Document 출처의 Storage 객체에 접근할 수 있습니다. 저장한 데이터는 브라우저 세션 간에 공유됩니다. localStorage는 sessionStorage와 비슷하지만, localStorage의 데이터는 만료되지 않고 sessionStorage의 데이터는 페이지 세션이 끝날 때, 즉 페이지를 닫을 때 사라지는 점이 다릅니다. ("사생활 보호 모드" 중 생성한 localStorage 데이터는 마지막 "사생활 보호" 탭이 닫힐 때 지워집니다.)`

`localStorage에 저장한 자료는 페이지 프로토콜별로 구분합니다. 특히 HTTP(http://example.com)로 방문한 페이지에서 저장한 데이터는 같은 페이지의 HTTPS(https://example.com)와는 다른 localStorage에 저장됩니다.`

`키와 값은 항상 각 문자에 2바이트를 할당하는 UTF-16 DOMString의 형태로 저장합니다. 객체와 마찬가지로 정수 키는 자동으로 문자열로 변환합니다.`

라고 설명되어 있다. 


## Web Storage 와 Cookie 의 차이점

쿠키를 굽게되면 웹 요청시 매번 쿠키정보를 포함하여 서버로 전송시킨다. 하지만 Web Storage 의 경우 저장된 데이터가 클라이언트에 있을 뿐 서버로 전송되지 않아 네트워크 트래픽 비용에서 우위점이 있다고 한다. 또한 쿠키의 경우 4KB 까지의 문자열만 저장이 가능하다. 하지만 Web Storage 는 문자열 뿐만이 아닌 javascript 객체 까지 저장이 가능하며 쿠키와 다르게 저장기한이 따로 없다.

스토리지의 경우 저장기한이 없기 때문에 유지보수시 하나하나 삭제해줘야 하는 단점이 있다. 아직까지는 두가지 모두 장단점이 있어 필요에 의해 쿠키를 사용할지 스토리지를 사용할지 결정을 해야되는 것 같다. 
