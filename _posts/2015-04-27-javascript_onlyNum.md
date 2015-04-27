---
layout: post
title:  "javascript 숫자만 입력 - ie11 한글 입력 되는 이슈"
date:   2015-04-10 15:28:00
categories: java
tags: shit
shortUrl: 
---

웹 서비스를 만들다 보면, input태그에 숫자만 입력하도록 해야할 때가 있다.

그런데 이상하게 태그 기본 옵션 중 onlynumber 이라는 옵션 따위를 지원하고 있지 않다.

jquery 를 주로 사용하고 있기 때문에 jquery 함수가 존재하는지 찾아봤지만 jquery 라이브러리가 재공하는 함수는 없었다.


javascript로 숫자만 입력
---------------- 

숫자 입력을 하도록 하는 방법은 몇가지 존재한다.

그 중 잘 사용하고 있는 방법이 두가지 있는데, replace 하는 방법과, 이벤트의 기본 동작을 막아버리는 방법이다.

	$("#onlyNumber").keyup(function(event){
		var inputVal = $(this).val();
		$(this).val(inputVal.replace(/[^0-9]/gi,''));
	}

가장 쉽게 생각할 수 있는 방법이 replace 하는 방법인데, 브라우져마다 차이가 있겠지만, 화면에 입력한 문자가 노출되었다가 사라지는 현상이 나타난다.

약간 부자연스러워보이게되는 처리 방법이다.


	function numberKeyPress(e) {
	    var key;
	
	    if(window.event)
	         key = window.event.keyCode; //IE
	    else
	         key = e.which; //firefox
		
	    // backspace or delete or tab
	    var event; 
	    if (key == 0 || key == 8 || key == 46 || key == 9){
	    	event = e || window.event;
		    if (typeof event.stopPropagation != "undefined") {
		        event.stopPropagation();
		    } else {
		        event.cancelBubble = true;
		    }	
			return ;
		}
	   
	    if (key < 48 || (key > 57 && key < 96) || key > 105 || e.shiftKey) {
	    	e.preventDefault ? e.preventDefault() : e.returnValue = false;
	    }
	}

이벤트 객체의 속성을 읽어 e.preventDefault 즉, 이벤트의 기본 동작을 막아버리는 방법이 있다. 이 함수는 keydown 이벤트에서 호출해야 브라우저에 상관없이 숫자만 입력되도록 할 것이다.

약간 놓치기 쉬운 부분은 e.shiftKey 인데, e.shiftKey 체크를 하지않으면 숫자 특수 키들은 막지 못하게 된다.

이 방법을 사용하면 좀더 자연스럽게 처리할 수있다.

IE11 한글 입력 이슈
---------------- 
keydown 이벤트에 e.preventDefault 로 숫자만 입력되도록 막는경우 IE11 에서 호환성 보기를 끈 상태에서 한글이 입력되는 문제가 확인되었다.

정확한 원인은 아직 찾지 못하였는데, 아마 [CompositionEvent|https://developer.mozilla.org/ko/docs/Web/API/CompositionEvent] 가 keydown 이벤트 이후에 동작하기 때문이라고 예상하였다.

CompositionEvent 를 막아서 이것때문인지 원인을 파악해보려 여러가지 실험을 하였지만, 계속 한글이 입력이 되었다.

결국 찾은 방법은 input 태그에서 한글을 아예 입력하지 못하도록 막는 방법이었다.

style="ime-mode:disabled" 를 input 태그의 옵션으로 주게되면, 한글입력이 막히게 된다.

IE5부터 추가된 이 속성은 W3C 표준은 아니지만 현재 파이어폭스 3 이상 그리고 웹킷 계열 브라우저(사파리, 구글 크롬)에서도 구현될 예정인 것으로 알려져 있다.

ime-mode의 속성은 다음과 같다.


		auto -> 기본값. 아무것도 지정 안하면 이거다.
		active -> 포커스가 들어가자마자 한글
		inactive -> 포커스가 들어가자마자 영어
		disabled -> 한글 입력 불가

	
참조
---------------- 
<https://developer.mozilla.org/ko/docs/Web/API/CompositionEvent>