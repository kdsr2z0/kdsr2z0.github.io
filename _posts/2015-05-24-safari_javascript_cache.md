---
layout: post
title:  "IOS Safari history.back() 문제 : pageshow Event"
date:   2015-05-24 15:28:00
categories: java
tags: shit
shortUrl: 
---


문제 상황
---------------- 

모바일에서 호출되는 페이지 제작 중, 다음과 같은 상황을 만났다.
* query 가 무거워 페이지가 로딩되는데 시간이 걸린다.
* 모바일의 특성상 페이지가 로딩중이라는 표시가 눈에 잘 안뛴다.
* 로딩중 버튼을 다시 클릭할 시, 처음부터 다시 로딩된다.
* 성격 급한 사람은 계속 누른다.
* 서버는 무거운 query를 계속해서 수행해야한다.

문제 해결
---------------- 

* javascript flag 변수 하나를 flag=false 상태로 생성한다.
* submit event는 변수가 false일때만 수행 한다.
* submit event 직전에 falg=true 로 변경 한다.


이렇게 해결할 시, 버튼을 눌렀을때 한번만 submit 이벤트를 수행하게 되어, 서버가 받게되는 불필요한 부하를 줄일 수 있다.

IOS safari 문제 발생
---------------- 

위 방법으로 해결하였을때 IOS Safari 에서 문제가 발생했다.

Safari는 페이지가 submit 될 때 이 전 페이지를 그대로 보존한다.

즉, history.back() 과같은 javascript 이벤트에 의해 원래의 페이지로 돌아오게 되면 페이지 reload 없이 이전 상태 그대로의 페이지를 보여준다.

이 방법은 아주 빠른 웹 페이지 이동을 가능하게 해준다.

그러나 위 문제 상황을 해결한 방법의 경우, 페이지가 reload 될 때마다 flag=false 가 새롭게 생성되어야 한다.

어떤 이유로 인해 history.back() 해서 돌아왔는데 flag=ture 상태이면 submit 이벤트가 다시는 동작하지 않게 된다.

pc의 web이나 android의 웹의 경우 history.back()으로 페이지에 되돌아올 때 페이지를 다시 로드한다. 따라서 flag=false 가 새롭게 생성된다.

그러나, IOS 7.x.x 버전이나 그 이상의 버전에서의 safari에서는 위와 같은 동작을 정상적으로 수행할 수 없다.

IOS safari 문제 해결 - 실패 케이스
---------------- 

html event에는 onload 이벤트와 unload 이벤트가 있다.

두 이벤트에 flag=false 로 변경해주는 로직을 추가했다.

그러나 safari에서는 당연하게도 다음 페이지로 넘어가거나, 다시 페이지로 되돌아 올 때 두 이벤트가 발생하지 않았다.


IOS safari 문제 해결 
---------------- 

열심히 구글링을 한 결과 pageshow 라는 이벤트를 찾게되었다.

이 이벤트의 설명은 다음과 같다.

	The pageshow event is triggered on the "to" page, after the transition animation completes.

즉, 현재 페이지로 전환 에니메이션이 완료되었을때 발생하는 이벤트 이다.

이 이벤트의 경우 history로 인한 이동에서도 반드시 수행되게 되어있다.

이 이벤트 외에도 pagebeforeshow, pagebeforehide, pagehide 이벤트가 있다.

이 이벤트 사용 방법은 다음과 같다.

	$("document").on("pageshow",function(event){...})
	
	
	$("document").on("pageshow","page",function(event,data){...})
	

두 번째 방법의 page 사용법은 다음 코드에서 볼 수 있다.

	<div data-role="page" id="pageone">
	  <div data-role="header">
		<h1>Header Text</h1>
	  </div>

	  <div data-role="main" class="ui-content"> 
		<p>Page One</p>
		<a href="#pagetwo">Go to Page Two</a>
	  </div>

	  <div data-role="footer">
		<h1>Footer Text</h1>
	  </div>
	</div> 

	<div data-role="page" id="pagetwo">
	  <div data-role="header">
		<h1>Header Text</h1>
	  </div>

	  <div data-role="main" class="ui-content">
		<p>Page Two</p>
		<a href="#pageone">Go to Page One</a>
	  </div>

	  <div data-role="footer">
		<h1>Footer Text</h1>
	  </div>
	</div> 
	

주의사항 - 위 코드들은 jquery에 의해 수행된다.


참조
---------------- 
<http://www.w3schools.com/jquerymobile/event_pageshow.asp>