---
layout: post
title:  "RedirectAttribute 와 FlashMap"
date:   2015-03-08 15:28:00
categories: java
tags: shit
shortUrl: 
---

Spring의 Redirect
---------------- 
Spring 에서 리다이렉트를 사용해야할 상황이 제법 많이 발생한다.

리다이렉트는 브라우저에서 표시되는 URL을 다른 URL로 바꿔주면서 그 페잊를 호출한다.
URL은 그대로 두고, 페이지의 내용만 바꿔주는 forward 와는 여러가지로 다르다.

포워딩은 Spring의 Controller 가 다른 view를 호출한다는 개념이면, 리다이렉트는 다른 URL을 호출하여 다른 Controller를 호출한다는 개념이다.

그 동작은 마치, html 의 <a> 태그의 href 속성과 같다고 보면된다.

스프링에서 리다이렉트는 다음과 같이 사용할 수 있다.

{code}

{code}

그런데 다른 페이지를 호출할 때, parameter를 전달해야할 상황이 발생할 수 있다.

어떻게 해야할까?

웹 환경에서 client 에서 server 로 parameter를 전달하는 방법은 두가지가 있다.

* Get
* Post

get과 post 의 특성은 따로 설명하지 않도록 하겠다.


앞서 redirect는 html 에서 a 태그의 동작과 비슷하다고 설명하였다.

즉, redirect도 a 태그와 같이 파라미터를 전달하기 위해서는 url 에 get 방식의 형태로 parameter를 전달할 수 있다.

Spring 프레임워크에서는 RedirectAttributes 라는 객체를 지원해 주고있다.

RedirectAttributes 객체는 다음과 같이 사용하여 URL에 String으로 attribute를 붗여줘야하는 수고를 덜어준다.

{code}
{code}

그러나 get 방식은 url에 parameter들이 노출되기 때문에 보안상 좋지 않다.

그렇다면 Post방식으로 Parameter를 전달할 수 있는 방법이 있을까?

RedirectAttributes 에는 FlashMap이라는 모델을 지원한다.

FlashMap은 Redirect전 session과 같은 장소에 저장한뒤 redirect 후 즉시 삭제한다. 마치 Post 방식 처럼 URL에 Parameter를 노출하지 않고 전달하게 되는 것이다.



정리
---------------- 
그런데 이 방식도 완전하진 않다. 

서버가 여러대일 경우를 생각해보자.

이 경우 세션을 사용하는 것이 문제가 될 수 있다. 여러대의 서버 세션을 공유하도록 장치해 두지 않으면 안될 것이다.

또, 다른 도메인으로 리다이렉트 할시 사용할 수 없는 방법이다. 다른 도메인의 서버와는 세션을 공유하도록 하는게 불가능하기 때문이다.





참조
---------------- 
http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/servlet/mvc/support/RedirectAttributes.html
http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/servlet/FlashMap.html

