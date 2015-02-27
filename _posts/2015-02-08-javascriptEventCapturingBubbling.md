---
layout: post
title:  "Javascript Event Capturing 과 Bubbling"
date:   2015-02-08 15:28:00
categories: javascript
tags: javascript
shortUrl: 
---

예제 코드
---------------- 

	<!DOCTYPE html>
	<html>
	<head>
	<meta charset=utf-8 />
	<title> JS 디버깅 연습1 </title>

	</head>
	<body>
			<div class="testDiv" id="1">
				<div class="testDiv" id="2">
					<div class="testDiv" id="3">
						<button id="btn1">button1</button>
						<a href="http://www.naver.com">go naver</a>
					</div>
					<div class="testDiv" id="4">
						<button id="btn2">button2</button>
						<
					</div>
				</div>
			</div>
			
			<div class="testDiv" id="6">
				<div class="testDiv" id="7">
					<div class="testDiv" id="8">
						<button id="btn3">button3</button>
					</div>
				</div>
			</div>			
			
	</body>
	<script type="text/javascript" src="jquery-1.11.2.js"></script>
	<script type="text/javascript">
		$(document).ready(function() {

			$("a").bind("click",function(e) {
				//e.preventDefault(); 설정 하게 되면 a tag의 default event 가 막히게 된다.
				//e.stopPropagation();
				//return false;
			});
			
			$("#2").bind("click",function(e) {
				//e.stopPropagation();
			});

			$(".testDiv").bind("click",function(e) {
				console.log("clicked div id : " + $(this).attr("id"));
			});
			
			document.getElementById("6").addEventListener("click",function(e) {
				console.log("clicked div id : " + $(this).attr("id"));
			},true);
			document.getElementById("7").addEventListener("click",function(e) {
				console.log("clicked div id : " + $(this).attr("id"));
			},true);
			document.getElementById("8").addEventListener("click",function(e) {
				console.log("clicked div id : " + $(this).attr("id"));
			},true);
		});
	</script>
	</html>

위 코드를 보면 아래 자바 스크립트에서 class 가 testDiv인 Element에 event listener를 등록 하고 있다. <br>
jquery에 의해  내부적으로 $(".testDiv")로 select한  리스트의 각 Element에 event listener가 등록된다.<br>
위 코드에 의해 다음과 같은 노드 구조가 생성된다.

![](/img/capturingbubbling1.JPG)


Capturing 
---------------- 

Capturing은 Event가 발생한 Element까지 접근하는 과정을 말한다.

![](/img/capturingbubbling2.JPG)

이벤트가 발생한 지점까지 Capturing 하게 된다.


Bubbling
---------------- 

Bubbling은 Capturing의 끝 지점에서 부터 되돌아 오는 과정을 말한다.<br>

![](/img/capturingbubbling3.JPG)

* button1을 클릭 하였을 때의 로그
	
	
		clicked div id : 3
		clicked div id : 2
		clicked div id : 1
		
	
* button2를 클릭 하였을 때의 로그



		clicked div id : 4
		clicked div id : 2
		clicked div id : 1	

이때, 각 Element의 Event와 EventListener를 호출하게 된다.<br>
그래서 위 로그를 보면 가장 자식 노드 부터 호출하고 있다.


Capturing Event
---------------- 

Capturing Event 라는 것도 있다. Capturing을 하면서 실행하는 이벤트를 말한다.<br>
Capturing Event는 jquery 에서는 제공하지 않기 때문에 addEventListener 를 통해서 지정할 수 있다.

* button3을 클릭 하였을 때의 로그


		clicked div id : 6
		clicked div id : 7
		clicked div id : 8	

		
위 로그를 보게 되면 캡쳐링 한 순서대로 이벤트가 실행 된 것을 볼 수 있다.


Capturing과 Bubbling 막기
---------------- 

__e.preventDefault();__

* html tag에 지정된 기본 event의 동작을 막는다.
* 예를 들어 위 예제의 a tag click event에 e.preventDefault(); 를 걸어주게 되면 go naver 링크를 클릭하게 되더라도 페이지가 이동하지 않는다.

__e.stopPropagation();__

* 사용자 정의 이벤트의 Capturing과 Bubbling 막는다.
* div 2 에 e.stopPropagation(); 를 걸어주게 되면 그 이후에 걸려 있는 이벤트 들이 동작하지 않게 된다.
* a tag를 이용하여 link로 이동을 할 때, __부모 노드들의 이벤트가 bubbling되어 모두 실행된 후__ 페이지가 이동하게 되어있다. 만약 e.stopPropagation(); 를 걸어주게 되면 버블링 되지 않고 페이지 이동만 할 수 있게 된다.

	
	clicked div id : 3
	clicked div id : 2
	clicked div id : 1	
	Navigated to http://www.naver.com/



__return false;__

* e.preventDefault(); e.stopPropagation(); 기능을 모두 수행해 준다.


<br><br><br>

참조
---------------- 

* <http://www.kirupa.com/html5/event_capturing_bubbling_javascript.htm>

