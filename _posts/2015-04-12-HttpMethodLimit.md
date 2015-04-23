---
layout: post
title:  "HTTP Method 제한"
date:   2015-04-10 15:28:00
categories: java
tags: shit
shortUrl: 
---

 웹 상에서 주로 쓰이는 프로토콜은 HTTP 이다. HTTP 프로토콜의 헤더는 통신에 필요한 여러 가지 정보들을 담고 있는다. 이 문서는 HTTP 헤더 정보 중 HTTP 메서드에 대해 설명하고, HTTP 메서드 제한의 필요성과 HTTP 메서드 제한 방법에 대해 설명하고자 한다.

 
HTTP method 란?
---------------- 

 HTTP 메서드는 서버가 HTTP 요청을 어떻게 처리해야 하는지 지시해 주는 필드이다.
서버는 HTTP method에 Requst-URI 에 대해 다른 동작을 수행할 수 있다.

Method 의 종류
---------------- 

__OPTIONS__ 

* Request-URI의 통신에 대한 정보들을 제공한다.
* 어떤 method 기능을 제공하는지, URI 에대한 기능적인 체크 동작을 수행한다. 
* Reqeust-URI 에는 body 필드를 정의할 필요가 없다.


__GET__

* Request-URI에 요청에 필요한 정보들을 모두 담는다.
* 새로고침과 같은 재요청이 잦은 요청에서 사용하면 유용하다.


__HEAD__

* 요청에 대한 응답 body를 반환하지 않고, 헤더 정보만 반환한다.
* 요청에 대한 유효성, 접근성 등을 확인할 때 많이 사용된다.


__POST__

* 요청에 필요한 정보들을 body 필드에 숨긴다.
* 길이 제한이 없고, 노출이 꺼려지는 정보들이 많을 때 더 많이 사용한다.
* Text 정보를 포함하여 바이너리로 된 데이터도 전송 가능하다.


__PUT__

* 요청 경로에 동봉된 정보들을 저장한다.
* 중복된 정보를 처리하는 방법에 대하여 정의해야한다.
* HTTP/1.1 에서는 정의하고있지 않다.


__DELETE__ 

* Request-URI 경로의 정보를 삭제한다.

__CONNECT__

* SSL tunneling 에서 사용되는 method 이다.

__TRACE__

* 클라이언트로부터 수신한 메시지를 응답에 포함시킨다.
* TRACE 메서드 요청에는 요청 정보를 포함하는 URI 를 사용하면 안될 것이다.


web.xml 설정으로 제한하기
---------------- 

web.xml 설정에 security-constraint 태그를 사용하여 제한할 수 있다.

		<security-constraint>
			<web-resource-collection>
			<web-resource-name></web-resource-name>
			<url-pattern>/*</url-pattern>
			<http-method>HEAD</http-method>
			<http-method>DELETE</http-method>
			<http-method>PUT</http-method>
			<http-method>OPTIONS</http-method>
			</web-resource-collection>
		</security-constraint>

security-constraint 는 web-resource-collection 내부에 정의된 방식의 요청을 제한하게 된다.

url-pattern 을 여러개 입력하여 pattern 별로 제한을 걸 수있다.

여기서 만약 제한할 http-method 를 하나도 설정하지 않으면 url-pattern에 해당하는 모든 요청이 제한되게 될 것이다.

이렇게 제한 되었을 때, 403번 error state를 반환하게 된다.

apache 설정으로 제한하기
---------------- 

apache 설정 파일 httpd.conf 에 설정을 추가해 주면 된다.

두가지 방법으로 제한을 걸 수 있다.

하나는 제한할 method들을 정의하는 방법이고, 다른 하나는 허용할 method들을 정의하는 방법이다.

__제한 할 method 정의__


		<Directory /home>
			<Limit PUT DELETE OPTIONS>
				Order allow,deny
				Allow from all
			 </Limit>
		</Directory>


__허용 할 method 정의__

		<Directory "/">
			<LimitExcept GET POST>
				Order deny,allow
				Deny from all
			</LimitExcept>
		</Directory>

allow from host 를 추가하여 득정 host에서 접근할 때는 모두 허용하는 방법이 있다.

Directory 라는 태그를 사용하여 특정 디렉토리로의 접근을 막고있다.

이 경우 디렉토리 별로 모두 제한을 걸어줘야하는 불편함이 있다.

그래서 URL pattern을 이용하여 제한하는 방법도 제공하고 있다.

Directory 대신 Location 을 사용하면 된다.


		<Location "/*">
			<LimitExcept GET POST>
				Order deny,allow
				Deny from all
			</LimitExcept>
		</Location>


최근 https 프로토콜을 많이 사용하고 있는 추세이다.

https 에서 method를 제한하기 위해서는 httpd.conf 파일에서만 설정을 해서는 안될 수 있다.

보통 httpd-ssl.conf 파일 과 같은 파일 명으로 https 프로토콜을 위한 설정 파일이 별도로 존재한다.

같은 방법으로 https 설정 파일에 method 제한을 걸어줘야 method가 제한되게 될 것이다.
		
		
참조
---------------- 

<http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html>
