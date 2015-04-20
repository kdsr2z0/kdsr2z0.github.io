---
layout: post
title:  "HTTP Method 제한"
date:   2015-04-10 15:28:00
categories: java
tags: shit
shortUrl: 
---

HTTP 프로토콜은 Request 시 몇가지 방식의 method를 제공하고 있다.

보통 GET, POST 를 사용하고 있지만, 여러가지 Method 들이 추가로 '정의' 되어 있다.

여기서 굳이 정의 되어 있다고 말한 이유는 다른 Method 들은 잘 사용하지 않고 있기 때문이다.

보안상의 문제로 GET, POST 를 제외한 다른 method들은 제한해 두는게 일반적이다.


Method 의 종류
---------------- 

__OPTIONS__ 

* 요청 URI 의 통신에 대한 정보를 제공한다.
* cache 되지 않는다.
* body를 정의할 필요가 없다.

__GET__

* Request-URI 에 Form entity 정보를 정의한다.
* 요청 정보가 URI 에 모두 있기 때문에 새로고침과 같은 동작을 자주 해야하는 요청에서 유용하다.

__HEAD__

* 이 메소드의 요청에는 message-body 정보를 반환하지 않는다.
* 응답의 meta 정보는 GET과 동일하다.
* 요청에 대한 유효성, 접근성 등을 확인할때 종종 사용된다.

__POST__

* Request-URI 에서 entity 들을 숨길때 사용한다.
* GET 방식 보다 사용하기는 조금 불편해도 entity의 길이 제한에서 자유로운 편이기때문에 더 많이 사용된다.

__PUT__

* 동봉된 entity를 Request-URI 경로에 저장한다.
* HTTP / 1.1 에서는 정의하고있지 않고 있다.

__DELETE__ 

* Request-URI 로 식별되는 resource를 삭제한다.
* 클라이언트로부터 수신한 메시지를 response에 포함시킨다.
* TRACE 요구에서는 Entity를 포함해서는 안된다.

__CONNECT__

* SSL tunneling 에서 사용되는 method 이다.



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

		
참조
---------------- 

<http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html>
