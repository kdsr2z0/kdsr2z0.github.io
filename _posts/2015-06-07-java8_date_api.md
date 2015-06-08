---
layout: post
title:  "Java8 의 날짜 Api"
date:   2015-05-31 15:28:00
categories: java
tags: shit
shortUrl: 
---

Java8 의 날짜 Api
---------------- 

기존 자바의 Date 와 Calendar Api는 여러가지 문제점을 갖고 있었다. 참조 : <http://kdsr2z0.github.io/java_date_problem/>
그러한 문제점을 해결하기 위해 많은 오픈 소스들이 생겨났었는데, Java8에서부터는 깔끔하게 정리된 API가 제공된다.

Java8에서 새롭게 제공되는 API는 java.time 패키지 안에 정의되어있다.


시간 척도
---------------- 

* 하루 86400초 : 스무딩 기술 적용 : 윤초 사용하지 않음.
	* 윤초를 사용하지 않는다는 것은 매우 중요할 수 있다.
* 각 정오에 공식 시간과 정확히 맞춘다.
* 그 외에 정확하게 정의된 방식으로 공식 시간과 가깝게 맞춘다.

Instant
---------------- 

* 타임라인의 한 시점을 나타내는 Class
* Instant.Max 는 1,000,000,000년 12월 31일
* 현재 시간은 Instant.now()

Duration
---------------- 

* 두 Instant 사이의 시간.
* 초는  long, 나노초는 int에 저장.

__시간 연산__

* plus*, minus*, multipliedBy, dividedBy, negated, isZero, isNegative 
	* plus 로 시작하는 메서드는 각 단위의 시간을 더하는 메서드.
	* multipliedBy, dividedBy, negated : 차이 시간을 곱하기, 나누기, 부호 변환을 한다. Duration 객체에서만 사용가능
* 각 연산의 결과는 새로운 객체로 반환된다.


HumanTime
---------------- 

휴먼타임은 인간이 사용하는 시간을 말한다.

휴먼타임에는 지역 날짜/시간과 구역시간이 있다.

LocalDate
---------------- 

* 구역 시간이 시간계산에 방해가 되는 경우도 존재한다. (일광절약시간과 같은 이유로...)
* 따라서 LocalDate 클래스는 구역 시간이 배제된 년월일 정보만 갖는 객체를 제공한다.
* 메서드 : now, of, plus*, minus*, with*, get*, unitl, isBefore, isAfter, isLeapYear
	* of 는 지정한 시간의 객체를 생성
	* with은 객체에서 각 해당 단위의 시간만 변환된 객체 반환
	

예) 매년 255번째 날은 프로그래머의 날이다.

		LocalDate programmersDay = LocalDate.of(LocalDate.now().getYear(),1,1).plusDays(255);
		System.out.println("this year's programmersDay : " + programmersDay);

		==>
		this year's programmersDay : 2015-09-13


* 존재하지 않는 날짜로 리턴되지는 않는다.


		System.out.println("LocalDate.of(2016,  1, 31).plusMonths(1) : " + LocalDate.of(2016,  1, 31).plusMonths(1));
		System.out.println("LocalDate.of(2016,  3, 31).minusMonths(1) : " + LocalDate.of(2016,  3, 31).minusMonths(1));

		==>
		LocalDate.of(2016,  1, 31).plusMonths(1) : 2016-02-29
		LocalDate.of(2016,  3, 31).minusMonths(1) : 2016-02-29



날짜 조정기
---------------- 

Java8에서는 날짜 조정기라불리는 TemporalAdjusters 클래스를 제공한다.

일반적으로 날짜 조정 메서드의 결과를 LocalDate 객체의 with 메서드에 전달하여 사용한다.

* 다음, 혹은 이전 주 x요일 날짜 : next(dayOfWeek), previous(dayOfWeek), 오늘 포함 : nextOrSame(dayOfWeek), previousOrSame(dayOfWeek)
* 그 달의 n번째 지정요일 날짜 : dayOfWeekInMonth(n, dayOfWeek)
* 그 외 각 첫번째, 마지막 날짜 : first*, last*

Local Time
---------------- 

* 시분초 그리고 나노초 정보만 갖는 객체를 제공 : 24시간 이내의 시간 연산만 제공
* 메서드 : now, of, plus*, minus*, with*, get*, toSecondOfDay, toNanoOfDay, isBefore, isAfter


ZonedDateTime
---------------- 

* 구역에 해당하는 년, 월, 일, 시, 분, 초, 나노초 제공한다.
* zoneId는 ZoneId.getAvailableZoneIds를 호출하면 나온다. : 실험 환경에서는 589개의 ZoneId가 출력되었다.


		System.out.println(ZoneId.getAvailableZoneIds().size());
		==>
		589


* 메서드 : now, of, ofInstant, plus*, minus*, with*, get*, getOffset(UTC로부터으 offset), to*, isBefore, isAfter
* 일광절약시간을 고려한 날짜 계산을 위해서는 Period 클래스를 사용해야한다.

예) 일광절약 시간이 적용된 케이스와, Duration 클래스를 사용하여 시간연산을 한 케이스와 Period 클래스를 사용한 케이스의 결과를 볼 수 있다.

		ZonedDateTime origin = ZonedDateTime.of(LocalDate.of(2013, 10, 27), LocalTime.of(2,30), ZoneId.of("Europe/Berlin"));
		System.out.println("origin : " + origin);
		ZonedDateTime plusOneHours = origin.plusHours(1);
		System.out.println("plusOneHours : " + plusOneHours);
		ZonedDateTime plusTwoHours = origin.plusHours(2);
		System.out.println("plusTwoHours : " + plusTwoHours);
		ZonedDateTime plusDurationSevenDay = origin.plus(Duration.ofDays(7));
		System.out.println("plusDurationSevenDay : " + plusDurationSevenDay);
		ZonedDateTime plusPeriodSevenDay = origin.plus(Period.ofDays(7));
		System.out.println("plusPeriodSevenDay : " + plusPeriodSevenDay);

		==>
		origin : 2013-10-27T02:30+02:00[Europe/Berlin]
		plusOneHours : 2013-10-27T02:30+01:00[Europe/Berlin]
		plusTwoHours : 2013-10-27T03:30+01:00[Europe/Berlin]
		plusDurationSevenDay : 2013-11-03T01:30+01:00[Europe/Berlin]
		plusPeriodSevenDay : 2013-11-03T02:30+01:00[Europe/Berlin]



DateTimeFormatter
----------------

기존에 있던 SimpleDateFormat과 특별히 차이점을 발견하지는 못하였지만, java.time 패키지의 클래스들을 지원한다.

* 미리 정의된 표준 포맷터가 있다.
* 로케일 종속 포맷터가 있다.
* 커스텀 패턴을 이용할 수 있다.
* 하휘 호환성 때문에 java.util.DateFormat 인스턴스가 필요할 경우 toFormat() 메서드를 호출하면 된다.


		System.out.println("표준 포맷터 : " + DateTimeFormatter.ISO_DATE_TIME.format(ZonedDateTime.now()));
		System.out.println("로케일 종속 포맷터 형식: " + DateTimeFormatter.ofLocalizedDate(FormatStyle.FULL).format(ZonedDateTime.now()));
		System.out.println("로케일 종속 포맷터 변경: " + DateTimeFormatter.ofLocalizedDate(FormatStyle.FULL).withLocale(Locale.FRENCH).format(ZonedDateTime.now()));
		System.out.println("커스텀 포맷터 : " + DateTimeFormatter.ofPattern("E yyy-MM-dd HH:mm").format(ZonedDateTime.now()));

		==>
		표준 포맷터 : 2015-06-03T09:05:32.937+09:00[Asia/Seoul]
		로케일 종속 포맷터 형식: 2015년 6월 3일 수요일
		로케일 종속 포맷터 변경: mercredi 3 juin 2015
		커스텀 포맷터 : 수 2015-06-03 09:05


* pasre는 각 날짜 클래스의 parse method를 사용한다.


		LocalDate churchsBirthday = LocalDate.parse("1903-06-14");
		System.out.println("basic formatter parse : " + churchsBirthday);
		ZonedDateTime apollo11launch = ZonedDateTime.parse("1969-07-16 03:32:00-0400", DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ssxx"));
		System.out.println("custom formatter : " + apollo11launch);

		==>
		basic formatter parse : 1903-06-14
		custom formatter : 1969-07-16T03:32-04:00



참조
---------------- 
가장 빨리 만나는 자바8, 카이 호스트만 지음, 길벗