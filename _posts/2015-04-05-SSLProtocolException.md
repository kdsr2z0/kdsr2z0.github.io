---
layout: post
title:  "SSLProtocolException: handshake alert: unrecognized_name"
date:   2015-04-05 15:28:00
categories: java
tags: shit
shortUrl: 
---

서버에서 다른 서버로 API를 호출하던 중 서버가 __'javax.net.ssl.SSLProtocolException: handshake alert: unrecognized_name'__  라는 Exception을 발생시키는 난감한 상황을 만났다.

대충 이름만 봐서 처음에 Connect를 맺을때, handshake 하는 도중에 unrecognized_name를 만나서 SSLProtocolException 가 발생했구나... 라고 이해가 되면 좋겠지만. 무슨 상황인지 알 수 없었다.

처음 보는 Exception 이기 때문에 당연히 google.com 을 찾아 검색을 하였다.

검색하자 마자 보이는 해결책 들은 enableSNIExtension=false SNI 확장 기능을 꺼라 라는 것이었다.

가장 쉬워 보이는 방법은 __System.setProperty("jsse.enableSNIExtension", "false");__ 을 런타임 중에 실행 시켜라.. 라는 방법이었지만. Spring Web 상에서 런타임중에 한번 호출한다고 저 기능이 제대로 꺼지지 않을 것 같아 시도해 보지 않고 다른 방법을 찾았다.

빌드 옵션에  -Djsse.enableSNIExtension=false 를 넣어보라고 해서 넣어보라고 해서, 빌드 옵션에 해당 부분을 넣어서 빌드해 보았지만, 문제가 해결되지 않았다...

Server Name Indication(SNI)
---------------- 

애써 무시하고 있었지만. 처음 보는 단어가 하나 더있었다. 

SNI ( Server Name Indication ) 이다.

wikipedia 에 따르면 SNI는 TLS (SSL과 유사한 Protocol) 의 확장된 프로토콜이라고 소개하고 있다.

좀 더 내용을 읽어 보니, name-based virtual hosting 이라는 기능을 안전하게 사용하기 위한 확장된 프로토콜이라고 하는것 같았다.

name-based virtual hosting는 하나의 IP address에 여러개의 DNS Hostname을 붙여서 동작하게 하도록 하는 기능이다.

이 기능의 이점은 하나의 인증 정보를 여러 도메인에 공유할 수 있다는 점이다.


<br>
그러나, man-in-the-middle attack에 취약한 면이 있고, SNI를 통해 man-in-the-middle attack 의 가능성을 줄일 수 있다고 하는데. 정확한 원리까지는 이해하지 못하였다.


SSLProtocolException 해결법
---------------- 

결정적인 해결법은  rfc4366 문서에서 찾았다.

문서에 보면 다음과 같은 구문이 있다.

		   If the server understood the client hello extension but does not
		   recognize the server name, it SHOULD send an "unrecognized_name"
		   alert (which MAY be fatal).

대충 해석해 보자면, 서버가 client가 보낸 접속 요청의 extension(확장기능) 을 이해할 수 있는데, server name 을 알지못하면  unrecognized_name alert를 보낸다는 것이다.

즉, client가 보낸 server_name(host name) 이 서버의 virtual host의 server_name 과 일치하는게 없다면, unrecognized_name 를 응답에 포함시키겠다는 말이다.

api server에 가서 apache 설정(conf/httpd.conf)의 virtual host 부분을 확인해 보니. server_name이 DNS에 등록한 domain name 과 다르게 되어있었다.

즉시, server_name 을 고치고 아파치 재시작 후 테스트를 해보니 정상 동작되는것을 확인할 수 있었다.

참조
---------------- 

<http://en.wikipedia.org/wiki/Server_Name_Indication>

<http://www.ietf.org/rfc/rfc4366.txt>

