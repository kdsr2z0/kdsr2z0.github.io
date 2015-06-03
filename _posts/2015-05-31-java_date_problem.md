---
layout: post
title:  "java date api의 문제점"
date:   2015-05-31 15:28:00
categories: java
tags: shit
shortUrl: 
---


복잡한 날짜 계산법
---------------- 

__1582년 10월 4일 다음 10일은 생략되어있다.__


		TimeZone utc = TimeZone.getTimeZone("UTC");
		Calendar calendar = Calendar.getInstance(utc);
		calendar.set(1582, Calendar.OCTOBER , 4);
		String pattern = "yyyy.MM.dd";
		SimpleDateFormat format = new SimpleDateFormat(pattern);
		format.setTimeZone(utc);
		String theDay = format.format(calendar.getTime());
		System.out.println("theDay : " + theDay);

		calendar.add(Calendar.DATE, 1);
		String nextDay = format.format(calendar.getTime());
		System.out.println("nextDay : " + nextDay);

		==>
		theDay : 1582.10.04
		nextDay : 1582.10.15


그레고리력을 적용하면서 율리우스력에 의해 누적된 오차를 교정하기 위한 건너뛴 기간.


__서울 1988년 5월 7일 23시의 1시간 후는 5월 8일 새벽 1시 : 일광절약시간제 : 재현 안됨__


		TimeZone seoul = TimeZone.getTimeZone("Asia/Seoul");
		Calendar calendar = Calendar.getInstance(seoul);
		calendar.set(1988, Calendar.MAY , 7, 23, 0);
		String pattern = "yyyy.MM.dd HH:mm";
		SimpleDateFormat format = new SimpleDateFormat(pattern);
		format.setTimeZone(seoul);
		String theDay = format.format(calendar.getTime());
		System.out.println("theDay : " + theDay);

		calendar.add(Calendar.HOUR_OF_DAY, 1);
		String nextDay = format.format(calendar.getTime());
		System.out.println("nextDay : " + nextDay);

		==>
		theDay : 1988.05.07 23:00
		nextDay : 1988.05.08 00:00


많은 나라들이 21세기 이후로 폐지하여 2013년 기준으로 유럽과 북미, 중동 일부 국가와 남반구 일부 지역에서만 실시되고 있다.


__서울 1961년 8월 9일 23시 59분의 1분 후는 0시 30분__


		TimeZone seoul = TimeZone.getTimeZone("Asia/Seoul");
		Calendar calendar = Calendar.getInstance(seoul);
		calendar.set(1961, Calendar.AUGUST, 9, 23, 59);
		String pattern = "yyyy.MM.dd HH:mm";
		SimpleDateFormat format = new SimpleDateFormat(pattern);
		format.setTimeZone(seoul);
		String theDay = format.format(calendar.getTime());
		System.out.println("theDay : " + theDay);

		calendar.add(Calendar.MINUTE, 1);
		String nextDay = format.format(calendar.getTime());
		System.out.println("nextDay : " + nextDay);

		==>
		theDay : 1961.08.09 23:59
		nextDay : 1961.08.10 00:30


과거 타임존을 \+8:30 분으로 적용했던적이 몇번 있는데 그 부분을 보정해준다.



__협정세계시 2012년 6월 30일 23시 59분 59초의 2초 후는 7월 1일 0시0분1초 : 윤초가 적용되지 않았다.__

Reddit, Foursquare, Yelp, LinkedIn 등 많은 기업이 장애를 겪었다. Linux + Java 환경의 시스템이 많았고, Cassandra, Hadoop, Elasticsearch 등 데이터 저장, 검색 플랫폼에서  CPU를 100% 사용하는 문제가 발생했다고 한다. <http://www.zdnet.co.kr/news/news_view.asp?artice_id=20120702094444>

시간대 관리
---------------- 

* 시간대 데이터베이스라는 곳에 저장되어 관리하고 있다.
* java는 운영체제의 시간대 패치와 독립적이다.
* TZUpdater라는 도구로 JRE 전체를 업그레이드하지 않고 시간대 데이터만 최신으로 갱신하는 방식도 지원한다.


java 날짜와 시간 API 문제점
---------------- 

__1) 불변 객체가 아니다.__

* VO는 한번 생성되었을때 변경될 수 있으면 안된다.
* 변경 가능하다면, 여러객체에서 공유될때, 한 객체에서의 set이 다른 객체에 어떤 영향을 미칠지 알 수 없다.


__2) int 상수 필드 남용.__

* 예 : calendar.add(Calendar.SECOND, 2);

__3) 헷갈리는 월 지정.__

* java 1.0에서 1월을 0으로 사용하기 시작했기 때문에, 많은 실수를 유발한다.


		Calendar calendar = Calendar.getInstance();
		calendar.set(2015, 1 , 1);
		String pattern = "yyyy.MM.dd";
		SimpleDateFormat format = new SimpleDateFormat(pattern);
		String theDay = format.format(calendar.getTime());
		System.out.println("theDay : " + theDay);

		==>
		theDay : 2015.02.01


__4) 일관성 없는 요일 상수__

* Calendar.get(Calendar.DAY_OF_WEEK) : 일요일은 1
* Date.getDay() : 일요일은 0


		Date date = new Date();
		Calendar calendar = Calendar.getInstance();
		System.out.println("Date.getDay() : " + date.getDay() );
		System.out.println("Calendar.get(Calendar.DAY_OF_WEEK) : " + calendar.get(Calendar.DAY_OF_WEEK)  );

		==>
		Date.getDay() : 3
		Calendar.get(Calendar.DAY_OF_WEEK) : 4


__5) 불편한 역할 분담 및 날짜계산.__

* 날짜 생성은 Date 객체, 날짜 연산은 Canlender 객체, 최종 결과는 다시 Date객체

__6) 오류에 둔감한 시간대 ID__


		TimeZone seoulAsia = TimeZone.getTimeZone("Seoul/Asia"); //틀림
		TimeZone asiaSeoul = TimeZone.getTimeZone("Asia/Seoul"); //옳은 ID

		System.out.println("seoulAsia : " + Calendar.getInstance(seoulAsia)); //id가 GMT인 것으로 출력된다.
		System.out.println("asiaSeoul : " + Calendar.getInstance(asiaSeoul));

		==>
		seoulAsia : java.util.GregorianCalendar[time=1433289738018,areFieldsSet=true,areAllFieldsSet=true,lenient=true,zone=sun.util.calendar.ZoneInfo[id="GMT",offset=0,dstSavings=0,useDaylight=false,transitions=0,lastRule=null],firstDayOfWeek=1,minimalDaysInFirstWeek=1,ERA=1,YEAR=2015,MONTH=5,WEEK_OF_YEAR=23,WEEK_OF_MONTH=1,DAY_OF_MONTH=3,DAY_OF_YEAR=154,DAY_OF_WEEK=4,DAY_OF_WEEK_IN_MONTH=1,AM_PM=0,HOUR=0,HOUR_OF_DAY=0,MINUTE=2,SECOND=18,MILLISECOND=18,ZONE_OFFSET=0,DST_OFFSET=0]
		asiaSeoul : java.util.GregorianCalendar[time=1433289738040,areFieldsSet=true,areAllFieldsSet=true,lenient=true,zone=sun.util.calendar.ZoneInfo[id="Asia/Seoul",offset=32400000,dstSavings=0,useDaylight=false,transitions=22,lastRule=null],firstDayOfWeek=1,minimalDaysInFirstWeek=1,ERA=1,YEAR=2015,MONTH=5,WEEK_OF_YEAR=23,WEEK_OF_MONTH=1,DAY_OF_MONTH=3,DAY_OF_YEAR=154,DAY_OF_WEEK=4,DAY_OF_WEEK_IN_MONTH=1,AM_PM=0,HOUR=9,HOUR_OF_DAY=9,MINUTE=2,SECOND=18,MILLISECOND=40,ZONE_OFFSET=32400000,DST_OFFSET=0]

__7) Date 하위 클래스 문제__

* java.sql.Date : 클래스의 이름이 같아 헷갈림.
* java.sql.TimeStamp : 나노초 필드를 추가한 클래스, equals() 의 대칭성을 어겼다. Date 와 TimeStamp 클래스 간의 equals() 동작시, a.equals(b) 가 true더라도 b.equals(a)가 false 인 경우가 있다.


		long currentDateTime =System.currentTimeMillis();
		Date a = new Date(currentDateTime);
		Timestamp b = new Timestamp(currentDateTime);
		Date c = b;

		System.out.println("a.eauls(c)=" + a.equals(c));
		System.out.println("c.eauls(a)=" + c.equals(a));

		==>
		a.eauls(c)=true
		c.eauls(a)=false


__8) 불필요한 checked Exception__


		try {
			Date date = new SimpleDateFormat("yyyyMMdd").parse("9234120120508");
			System.out.println(date);
		} catch (ParseException e) {
			System.out.println("ParseException"); //파싱 에러가 발생하지 않음.
		}

		==>
		Sun Nov 08 00:00:00 KST 9564



날짜 관련 오픈 소스
---------------- 

Joda-Time: <http://www.joda.org/joda-time>

Time and Money Code Library: <http://timeandmoney.sourceforge.net>

CalendarDate: <http://calendardate.sourceforge.net>

date4j: <http://www.date4j.net>



참조
---------------- 
<http://helloworld.naver.com/helloworld/textyle/645609>