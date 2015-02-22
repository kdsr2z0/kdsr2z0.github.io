---
layout: post
title:  "Java의 ConcurrentSkipListMap"
date:   2015-02-8 15:28:00
categories: java
tags: shit
shortUrl: 
---

java.util.concurrent package
---------------- 
java.util.concurrent package는 병렬 프로그램에서 유용한 공통적인 유틸 클래스들을 제공해 주고 있다.<br>
ConcurrentSkipListMap 역시 이 패키지에 속해있는 클래스 이다.

* __Executors__ interface : 사용자 정의 시스템을 지원하기 위한 Executors interface를 제공한다.
* __Queue__ : 멀티 쓰레드에서 병렬적으로 Task를 처리하기 위해서는 Queue를 많이 사용할 수 밖에 없다. concurrent package에서는 안전한 동작을 non-blocking으로 지원해주는 ConcurrentLinkedQueue를 비롯하여, BlockingQueue interface 구현한 여러가지 Queue들을 지원한다.
* __Timing__ : 타임아웃에 관련된 클래스들을 제공한다.
* __Synchronizers__ : 동기화를 위한 클래스들 ; Semaphore, CountDownLatch, CyclicBarrier, Pharser, Exchanger 등을 제공한다.
* __Concurrent Collections__ : 멀티쓰레드 프로그램에서는 Queue를 제외 하고도 여러가지 자료 구조들이 필요할 것이다. concurrent package에서는 ConcurrentHashMap, ConcurrentSkipListMap, ConcurrentSkipListSet, CopyOnWriteArrayList, CopyOnWriteArraySet의 자료구조를 제공하고 있다.
* 또한, java.util.concurrent.locks 패키지와 java.util.concurrent.atomic 패키지를 포함하고 있다.


<br><br><br>

java의 concurrent skip list
----------------
Java에서는 jdk 1.6 버전에서 부터 ConcurrentSkipListMap, ConcurrentSkipListSet 클래스를 통해 Skip List를 제공해 주고 있다.

<br><br><br>

ConcurrentSkipListMap의 extends, implements
---------------- 
__abstract class AbstractMap__

* Map의 기본인 key,와 value 를 관리하기위한 변수 및 메소드 들을 제공해 주고 있다.
* volatile transient Set<K> keySet; 
* volatile transient Collection<V> values; 
* transient : 직렬화 과정에서 제외 시킨다는 표시.
* volatile : 비동기적으로 변경될 수 있는 변수라는 표시.

<br><br>
__public interface Serializable__

* Serializable interface 를 implements 하게 되면 별도의 method를 구현하지 않아도 직렬화 기능을 사용할 수 있도록 해 준다.

<br><br>
__public interface Cloneable__

* Object.clone() 을 호출 하였을 때, CloneNotSupportedException을 발생하지 않도록 하기 위하여 표시해 주는 interface

<br><br>
__public interface ConcurrentNavigableMap__		

* extends interface ConcurrentMap : 병렬 프로그래밍에서 Map의 메모리 무결성 효과를 주기 위한 interface 이다.
* extends interface SortedMap : implements한 Map 클래스가 정렬된 상태임을 표시해 준다.

<br><br><br>

정리...
---------------- 
ConcurrentSkipListMap의 extends와 implements 만 보아도 ConcurrentSkipListMap가 가진 특성을 잘 알 수 있다.<br>

* AbstractMap 를 extends 하고 있으므로 Map의 기본 형태를 따른다는것을 알 수 있다.
* 또한 implements된 interface를 보면 직렬화(Serializable) 가능하고, 클론(Cloneable)을 만들 수 있으며, 병렬 작업을 지원하는 정렬된 Map 클래스(ConcurrentNavigableMap)임을 알 수 있다.

<br><br><br>

마치며...
---------------- 
자바에서 제공되는 클래스를 처음으로 유심히 보게된 것 같다.<br>
어떤 패키지에 속해 있는지, 어떤 클래스를 상속 받는지, 어떤 인터페이스를 구현하는지 만을 보아도 해당 클래스가 어떤 특성을 갖는지 알 수 있도록 되어있는 것을 볼 수 있다. <br>
앞으로 상속이나 인터페이스의 사용에 고민을 좀 더 하여야 할 듯 하다.


참조
----------------
http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/package-summary.html


