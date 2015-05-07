---
layout: post
title:  "Ajax 동기 동작"
date:   2015-05-03 15:28:00
categories: java
tags: shit
shortUrl: 
---

Ajax는 정말 유용한 도구이다.

웹 서비스에서 페이지 이동이나 리로드 없이 서버와 통신을 하도록해주는 유용한 도구이다.

웹 페이지가 화면전환 없이 서버와 통신을 하게되면 좀 더 자연스러워 보이게 된다.

그런데 Ajax를 기본 설정대로 사용하다보면 불편해질 때가 있다. 그 이유는 평소 쓰는것만 쓰고, 알고 있는것만 쓰고있기 때문인 것 같다.

어떤 값을 서버와 통신하여 validation 체크 후 그 다음 동작을 하도록 해야하는 상황이 종종 발생한다.

이럴때는 비동기 통신일 경우 callback함수를 통해 제어를 해야하는 불편함이 생긴다.

서버와의 통신이 지연되는 경우 그 사이에 다른 이벤트가 발생하게되면 특정 이벤트가 중복해서 발생하거나, 누락이 발생할 수 있다.

이러한 불편함을 해소할 수 있는 옵션들이 Ajax에 다 마련이 되어있다.

결론 - Ajax 동기 통신
---------------- 

결론은 간단하다. Ajax를 동기통신으로 옵션을 변경하여 사용하게 되면, Ajax 통신이 끝날때까지 웹페이지가 block 된다.

		$.ajax({
			async:false
		});

안좋게 보일수도 있지만, 불필요한 이벤트를 방지시킬 수 있는 장점이 있다.

다음은 간단한 테스트 코드이다.


		<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
		<%@ page session="false" %>
		<html>
		<head>
			<title>Home</title>
		</head>
		<body>
		<h1>
			Hello world!  
		</h1>

		<P>  The time on the server is ${serverTime}. </P>
		<button id="btn">button</button>
		<div id="resultLog">

		</div>
		<script type="text/javascript" src="/test/resources/js/jquery-1.8.3.min.js"></script>
		<script>
		$(document).ready(function(){
			$("#btn").click(function btnClickHandler(){
				$.ajax({
					url:"/test/ajaxTest",
					//async:false,
					success: function(data, dataType){
						$("#resultLog").html($("#resultLog").html() + "<br>" + data + " :: ");
					}
				});
				$("#resultLog").html($("#resultLog").html() + "<br>" +  "after ajax call");
			});
		});
		</script>

		</body>
		</html>

		

async 옵션이 주석 처리가 된 상태에서는 

		after ajax call
		SUCCESS ::

순서로 출력되게된다. 비동기 통신이기 때문에 ajax 통신 도중에 당음 로직이 실행되는것이다.

반면, async:false 옵션을 주게되면

		SUCCESS ::
		after ajax call
		
		
ajax 이벤트가 완료된 다음 다음 로직이 실행되는것을 볼 수 있다.

Ajax의 옵션들
---------------- 

흔히 사용하지 않는 옵션들은 잘 모르기 마련이다. 하지만 알고있게 되면 언젠가는 유용하게 쓰이지 않을까 싶다.

참조로 둔 ajax api 페이지 를 보는게 더 좋을지도 모르지만, 영어로 된 페이지 이기 때문에 한글로 간단하게 정리하였다.(공부할 겸...)

* __url__
	* 연결하고자하는 주소를 적으면 된다. 어렴풋이 기억나서 확실하지는 않지만 파일경로를 입력해서 비동기로 파일을 읽을수도 있다.

* __async__
	* default:true, 동기 통신을 하고자하면 false를 주면된다.

* __beforeSend__
	* request 전에 호출되는 이벤트, return false를 주게되면 ajax 이벤트가 취소된다.

* __cache__
	* default : false 통신 결과를 cache하지 않는다.

* __complete__
	* success, error 가 발생한 이후 발생되는 Event

* __contentType__
	* 서버로 데이터를 보낼 때 사용.

* __context__
	* 특정 element를 context로 설정하여, callback 함수의 주체로 만들 수 있다.

* __crossDomain__
	* default : false, ture 로 설정하면 domain이 달라도 요청이 가능하다.

* __data__
	* key, value pairs를 통해 서버로 parameter를 전달할 수 있다.

* __dataFilter__
	* 서버로 부터 전달받은 데이터를 필터링 할 수 있다.

* __dataType__ 
	* 서버에서 반환되는 데이터의 형식을 지정한다.
	* xml, html, script, json, jsonp, text 가 있다.

* __error__
	* 통신 중 실패했을 경우 호출되는 Event

* __global__
	* global Event를 동작시킬 것인지 설정한다. default 값은 true, 설정시 등록해놓은 Ajax global event가 bind 되어 발생한다.

* __headers__
	* key/value pairs 를 등록하여 헤더에 값을 추가한다.

* __ifModified__
	* Last-Modified header의 값을 보고 변경된 경우에만 true를 반환한다.

* __jsonp__
	* callback 함수를 json 형태의 parameter로 전달한다. crossDomain 문제를 해결할 수 있다고 하는데.. 한번 연구해봐야할 것 같다.

* __method__
	* http 전송 방식을 설정한다.

* __mimeType__
	* mimeType 을 설정한다.

* __password__
	* password 가 필요한 http 통신의 비밀번호를 설정한다.

* __processData__
	* 서버에서 받은 데이터를 자동으로 쿼리 문자열로 변환할지 여부를 설정할 수 있다.

* __scriptCharset__
	* 서버와 script의 character set 이 다르다면 설정해야한다.

* __success__
	* 통신이 성공하면 호출되는 함수를

* __statusCode__
	* 특정 state Code 에서 bind 될 event를 설정한다.

* __timeout__
	* 제한시간을 설정한다.

* __type__
	* method 옵션의 또다른 이름.

* __username__
	* 인증이 필요한 http 통신에서 사용자를 설정한다.

* __xhr__
	* XMLHttpRequest 나 ActiveXObject 가 만들어질 때 실행되는 callback 함수이다.




참조
---------------- 
<http://api.jquery.com/jquery.ajax/>
<https://api.jquery.com/category/ajax/global-ajax-event-handlers/>