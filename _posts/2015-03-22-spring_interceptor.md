---
layout: post
title:  "Spring과 HandlerInterceptor"
date:   2015-03-29 15:28:00
categories: java
tags: shit
shortUrl: 
---

HandlerInterceptor 소개
---------------- 

 HandlerInterceptor 는 Spring 에서 custom interceptor 를 등록할 때 사용하는 interface 이다.
 
 Spring 에서 interceptor 는 웹 서비스에서 필요한 여러가지 기능들을 쉽게 구현할 수 있도록 한다.
 
 interceptor는 주로 검증, 보안, 로깅 등... 시스템 내에서 전체적으로 적용되어야할 기능들을 수행 한다.
 
 스프링에서는 기본적으로 여러가지 Interceptor를 이미 제공하고있는데, Annotation과 같이 사용되면서 강력한 기능들을 제공해 주고 있다.
 대표적으로 여러가지 타입의 파라미터를 자동으로 mapping 시켜주는 기능 들을 예로 들 수 있다.
 
 mapping URL pattern을 지정하여, 특정 URL 로 접근하였을 때만 interceptor가 실행되도록 할 수 있다.

 
 

Spring에서 Interceptor 동작
---------------- 

* preHandle : 요청 시 Controller 로 진입하기 직전에 실행, boolean 타입의 반환값을 갖는데, false 를 return 하게되면 요청을 거부하게 된다.
* postHandler : 요청 시 Controller 에서 return 되는 과정에서 실행된다. postHandle에 도달하기 전에 Exception 이 발생한다면 Exception 이후의 postHandle 은 생략되게 된다.
* afterCompletion : postHandle 과 달리 Exception 이 발생하더라도 반드시 실행된다. try catch 구문에서 finally 와 비슷한 개념이다.

interceptor 동작 과정을 그림으로 보면 다음과 같다.

![](/img/interceptor.png)




interceptor class 선언
---------------- 

HandlerInterceptor interface를 이용하여 다음과 같은 method 들을 override 시키면 된다.

		package com.test.test.interceptor;

		import javax.servlet.http.HttpServletRequest;
		import javax.servlet.http.HttpServletResponse;

		import org.springframework.stereotype.Component;
		import org.springframework.web.servlet.HandlerInterceptor;
		import org.springframework.web.servlet.ModelAndView;

		@Component
		public class TestInterceptor implements HandlerInterceptor {
			
			@Override
			public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
				System.out.println("Call Interceptor : " + request.getRequestURI());
				System.out.println("preHandle");
				return true;
			}
			
			@Override
			public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
				System.out.println("postHandle");
			};
			
			@Override
			public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
				System.out.println("afterCompletion");
			}
		}


그런데 만약 세 method 중 preHandle만 필요할 경우 나머지 method를 모두 구현하기 귀찮을 수 있다.
그럴 경우, HandlerInterceptorAdapter를 상속 받아 필요한 method만 사용하면 좀 더 편할 수 있다.
		http://www.springframework.org/schema/mvc/spring-mvc.xsd

interceptor bean 생성
---------------- 

<interceptors> 태그 안에서 bean 으로 생성해 주면, interceptor로 등록되게 된다.

		
		<?xml version="1.0" encoding="UTF-8"?>
		<beans:beans xmlns="http://www.springframework.org/schema/mvc"
			xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
			xmlns:beans="http://www.springframework.org/schema/beans"
			xmlns:context="http://www.springframework.org/schema/context"
			xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
				http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
				http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

			<!-- DispatcherServlet Context: defines this servlet's request-processing infrastructure -->
			
			<!-- Enables the Spring MVC @Controller programming model -->
			<annotation-driven />

			<!-- Handles HTTP GET requests for /resources/** by efficiently serving up static resources in the ${webappRoot}/resources directory -->
			<resources mapping="/resources/**" location="/resources/" />

			<!-- Resolves views selected for rendering by @Controllers to .jsp resources in the /WEB-INF/views directory -->
			<beans:bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
				<beans:property name="prefix" value="/WEB-INF/views/" />
				<beans:property name="suffix" value=".jsp" />
			</beans:bean>
			
			<context:component-scan base-package="com.test.*" />
			
			<interceptors>
				<interceptor>
					<mapping path="/*"/>
					<beans:bean class="com.test.test.interceptor.TestInterceptor"/>
				</interceptor>
			</interceptors>
		</beans:beans>


여러개의 interceptor 
----------------

당연한 이야기이겠지만 interceptor는 여러개 등록할 수 있다.


		<interceptors>
			<interceptor>
				<mapping path="/*"/>
				<beans:bean class="com.test.test.interceptor.TestInterceptor"/>
			</interceptor>
			<interceptor>
				<mapping path="/*"/>
				<beans:bean class="com.test.test.interceptor.TestInterceptor2"/>
			</interceptor>
			<interceptor>
				<mapping path="/*"/>
				<beans:bean class="com.test.test.interceptor.TestInterceptor3"/>
			</interceptor>
		</interceptors>
		

interceptor 는 등록된 순서대로 순차적으로 실행 된다.
실행 순서는 다음과 같다.


		preHandle -> preHandle2 -> preHandle3 -----------------------------> controller
								<- postHandle <- postHandle2 <- postHandle3-------
		afterCompletion <- afterCompletion2 <- afterCompletion
		

실행되는 순서는 반드시 지켜지게 되지만, 각 interceptor는 독립적인 기능을 수행하도록 하는것이 바람직하다.
		

참조
---------------- 

http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/servlet/HandlerInterceptor.html

