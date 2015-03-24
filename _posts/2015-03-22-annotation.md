---
layout: post
title:  "Annotation"
date:   2015-03-08 15:28:00
categories: java
tags: shit
shortUrl: 
---

Annotation 소개
---------------- 

 어노테이션(Annotation)의 사전적 뜻은 주석이다. 그러나 자바에서 Annotation은 주석 이상의 역할을 한다.

Annotation의 등장은 jdk 1.5 버전 부터였다. 초창기의 어노테이션은 주석의 역할에서 크게 벗어나지 않았다.

조금 더 설명하면, 컴파일 시 error를 발생시키거나, 아니면 특정 error나 warning 을 무시하기 위해 사용되는 등 어떤 의미를 가지고 있는 주석의 역할을 Annotation이 담당하고 있었다.

이후, 컴파일 시  xml 파일등을 읽기 위한 메타데이터를 저장하는 용도로 사용되다가 지금에 와서는 런타임시에도 사용되고 있다.

그 활용성은 Spring 프레임 워크에서 사용되고 있는 것을 보면 아주 놀라울 정도로 복잡한 작업을 심플하게 처리하도록 해 주고 있는것을 보여준다.

어떤 PreHandler나 PostHandler와 적절히 같이 사용되면 마치 메크로를 쓰는듯한 느낌을 받을 수 있는 아주 고마운 도구이다.





Annotation 용도
---------------- 

Annotation은 앞서 소개에서 간단히 설명한 것 처럼 크게 세가지 정도의 용도로 사용되고 있다.

* __Information for the compiler__ : 컴파일 시에 어떤 에러를 발견하거나, 어떤 warning 들을 숨기기 위해서 사용된다. 이러한 Annotation은 컴파일 시에 버려진다.
	* @Override, @SuppressWarnings, @Deprecated 등 과 같은 Annotation이 이러한 역할을 제공한다.
	
* __Compile-time and deployment-time processing__ : 컴파일 동안만 사용되는 정보를 제공하는 용도로 사용한다. generate code, XML 파일등의 경로와 같은 정보를 저장하고있다가 컴파일 시 정보를 제공한다.
	* @Retention, @Documented, @Target 등이 있다. 이러한 Annotation은 컴파일 시 특별한 정보를 제공한다. 사실, 예를 든 Annotation은 Custom Annotation을 정의할때 사용하는 것들이다.

* __Runtile processing__ : 런타임시에도 남아 어떤 역할을 하게되는 Annotation들이 있다. 
	* @NotNull과 같은 Spring에서는 Validation 등의 역할을 제공하는 여러 Annotation들이있다. 프레임 워크가 제공하는 Annotation들 이외에도 다양한 Custom Annotation들을 만들어 런타임시 사용할 수 있다.

	
Custom Annotation 선언
---------------- 

단순히 주석으로 활용하는 Annotation은 다음과 같이 정의 될 수 있다.

		@interface ClassPreamble {
		   String author();
		   String date();
		   int currentRevision() default 1;
		   String lastModified() default "N/A";
		   String lastModifiedBy() default "N/A";
		   // Note use of array
		   String[] reviewers();
		}

위 코드에서 볼 수 있듯이 Annotation의 Element 들을 넣을 수 있으며, 배열도 가능하다. 이러한 Annotation은 다음과 같이 사용 한다.

		@ClassPreamble (
		   author = "John Doe",
		   date = "3/17/2002",
		   currentRevision = 6,
		   lastModified = "4/12/2004",
		   lastModifiedBy = "Jane Doe",
		   // Note array notation
		   reviewers = {"Alice", "Bob", "Cindy"}
		)
		public class Generation3List extends Generation2List {

		// class code goes here

		}

만약 이러한 Annotation에 @Documented를 붙이게 되면 공개되는 API 에서 Annotation의 Element들이 함께 제공되게 될 것이다.


		@Documented
		@interface ClassPreamble {
		   String author();
		   String date();
		   int currentRevision() default 1;
		   String lastModified() default "N/A";
		   String lastModifiedBy() default "N/A";
		   // Note use of array
		   String[] reviewers();
		}
		
다음 포스팅에서는 좀 더 다양한 Custom Annotation 활용법을 소개하도록 하겠 습니다.

참조
---------------- 
https://docs.oracle.com/javase/tutorial/java/annotations/
http://en.wikipedia.org/wiki/Java_annotation
http://docs.oracle.com/javase/7/docs/api/java/lang/annotation/Documented.html

