---
layout: post
title:  "Javascript 비동기 프로그래밍"
date:   2015-02-15 15:28:00
categories: java
tags: shit
shortUrl: 
---

Javascript 의 특성
---------------- 

* javascript는 script 언어이므로 인터프리터(interpreter) 에 의해 실행된다.
* 객체 기반의 언어이다.
* 단일 쓰레드에 의해 실행된다.


자바스크립트는 빠르고 쉽게 배워서 사용할 수 있다.

쉬운 언어이기 때문에 그만큼 많이 사용되지만, 조심해야할 부분을 쉽게 놓칠 수 있다.

또, 생각보다 다양한 방식으로 javascript를 프로그래밍 할 수 있다.



Javascript의 비동기 프로그래밍
----------------

동기 프로그래밍이란, 어떤 작업을 요청한 후 그 작업이 완료되기까지 기다렸다가, 응답을 받아 처리하는 것을 말한다.

반면 비동기 프로그래밍이란, 어떤 작업을 요청한 후 다른 작업을 수행하다가 interrupt 등을 이용하여 작업 완료를 알리면 그에대한 응답을 받아 처리하는 것을 말한다.

Javascript는 단일 쓰레드 기반이기때문에	비동기 프로그래밍을 하기가 어렵다. 더군다나 javascript는 별도의 사용자 정의 interrupt function을 지원하지 않고 있다.

그러나 Javascript에서도 비동기 프로그래밍을 할 수 있다.  정확하게는 비동기 프로그래밍 처럼 동작하게 만들 수 있는 것이다.

그 방법은 timer를 활용하는 방법이다.


javascript의 timer
----------------

* __setTimeout :__  단 한번 실행 된다.


		setTimeout(function(){ alert("Hello"); }, 3000);


* __setInterval :__  지정된 시간마다 실행된다.


		setInterval(function(){ alert("Hello"); }, 3000);


* __clearTimeout :__ 타이머를 제거한다.


		function myFunction() {
			myVar = setTimeout(function(){ alert("Hello"); }, 3000);
		}

		function myStopFunction() {
			clearTimeout(myVar);
		}

		
timer를 이용한 비동기 프로그래밍
----------------

단일 쓰레드에의해 실행되는 비동기 프로그래밍은 단순하다.

setInterval 을 이용하여 주기적으로 callback 함수를 호출하여 timeCount 를 증가 시키면서 특정 timeCount에 도달했을때, event를 발생(function 호출) 시키는 것이다.



		<!DOCTYPE html>
		<html>
		<body>

		<button onclick="myFunction()">Try it</button>

		<script>
		var timeCount = 1;
		function myFunction() {
			setInterval(function(){ 
				timeCount++;
				if(timeCount%10==0) {
					alert("Hello"); 
				}
			}, 1000);
		}
		</script>

		</body>
		</html>


위 예제 코드에 보면, 버튼을 클릭할 때 마다 timer가 하나씩 생겨난다.

하지만 그러한 방법은 위험한 방법이다. 여러개의 타이머가 동작할 시, 각 타이머의 callback에 의해 다른 timer의 지연이 발생할 수 있기 때문이다.

여러개의 비동기 작업이 있을때는 timer를 여러개 쓰는것 보다, 하나의 timer에 여러개의 timeCount를 두는것이 더 효율 적일 것이다.


		var timeCount = 1;
		var timeCount2 = 1;
		var timeCount3 = 1;
		function myFunction() {
			setInterval(function(){ 
				timeCount++;
				timeCount++;
				timeCount++;
				if(timeCount%10==0) {
					alert("Hello"); 
				}
				if(timeCount2%20==0) {
					alert("Hello2"); 
				}
				if(timeCount3%30==0) {
					alert("Hello3"); 
				}
			}, 1000);
		}


Ajax
----------------

그 외 시간이 많이 걸릴 수 있는 입출력(파일, url 호출 )에 대해 비동기 작업을 지원하고 있다.

Jquery에서 지원하는 기능으로, 파일 입출력이 성공했을때, 혹은 실패했을때에 대한 event 처리를 비동기적으로 할 수 있도록 지원하고 있다.

 

setTimeout의 적!
---------------- 

* __alert()__
* __confirm()__
* __prompt()__
	


위의 세가지	function은 javascript로 브라우져 상의 메시지 창을 띄우는 방법이다.

문제는 위 메시지들은 blocking 된다는 것에 있다.

만약 페이지의 이벤트 중 메시지 창을 뛰우는 것이 있다면, setTimeout의 timeCount 증가가 멈추게 될 것이다.


또, timer의 callback 함수도 동기적으로 동작하는 function 이다.

callback 함수에서 너무 무거운 function을 호출하게 되면, timer가 지연될 수 있다는 것을 염두에 두어야 한다.

	



참조
---------------- 

http://www.w3schools.com/jsref/met_win_settimeout.asp


