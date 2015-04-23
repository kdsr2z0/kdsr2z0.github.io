---
layout: post
title:  "response.sendRedirect() 사용시 주의사항"
date:   2015-04-10 15:28:00
categories: java
tags: shit
shortUrl: 
---

response.sendRedirect() 서버가 사용자의 브라우저를 지정한 URL 로 리다이렉트 시킬 수 있는 유용한 도구이다.

Spring 에서는 Controller에서 반환 String에 "redirect:" 라는 prefix를 붙여주면 동일한 기능을 수행하게 되어서 Spring에서는 잘 사용하지 않고 있다.

그러나 interceptor와 같은 곳에서 redirect 시키도록 하기 위해서는 response.sendRedirect()를 반드시 사용해야한다.


response.sendRedirect() 의 문제점
---------------- 

response.sendRedirect() 를 사용하면서 주의해야할 점은 저 메서드를 호출한다고 break가 걸리는게 아니라는것이다.

다시 설명하자면 throw new Exception() 을 호출하게되면 break가 걸리게되어 catch를 만날때까지 스텍을 거슬러 올라가게 된다.

그러나 response.sendRedirect() 의 경우는 그냥 response에 redirect를 하라 라는 명령만 저장하는 기능을 갖고 있따고 보면된다.

response.sendRedirect() 호출 뒤에 있는 로직이 그대로 수행되게 된다.

만약 if(value == null) 을 검사한 뒤 response.sendRedirect() 만 호출해주게 되면 그 이후 로직이 그대로 수행되면서 NullPointException을 발생시키게 될것이다.

보안상 이유때문에 response.sendRedirect()를 호출하게 되는 상황은 좀 더 심각한 문제가 된다.

보안 로직이 수행되도록 redirect를 시키려고 한 것인데 response.sendRedirect() 구문 이후의 로직이 그대로 실행되기 때문에 보안상 제대로 막았다고 보기 힘들다.

일단 response.sendRedirect() 이후 로직이 수행되어 만들어진 페이지 + redirect 명령이 응답으로 되돌아가는 것이기 때문에 임의의 프로그램으로 http통신을 모니터링 하게 되면 인증을 거친 후에나 볼 수 있는 페이지를 인증을 거치지 않고 볼 수 있는 어이없는 상황이 만들어질 수도 있다.



그래서????
---------------- 

response.sendRedirect() 가 호출된 이후의 로직이 완벽하게 break가 될 필요가 있다.

response.sendRedirect()는 주로 Interceptor에서 많이 사용하고있다. Interceptor의 preHandle 의 경우 return type이 boolean이다.

간단히 response.sendRedirect()를 호출한 후 return false;를 호출해주게 되면 빈 페이지 + redirect 명령 이 응답되기 때문에 본래 의도했던데로 동작하게 될 것이다.


