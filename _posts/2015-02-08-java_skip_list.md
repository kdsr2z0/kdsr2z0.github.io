---
layout: post
title:  "Java의 ConcurrentSkipListMap"
date:   2015-02-15 15:28:00
categories: java
tags: shit
shortUrl: 
---

java.util.concurrent package
----------------
* * *

java.util.concurrent package는 병렬 프로그램에서 유용한 공통적인 유틸 클래스들을 제공해 주고 있다. 

* __Executors__ interface : 사용자 정의 시스템을 지원하기 위한 Executors interface를 제공한다.
* __Queue__ : 멀티 쓰레드에서 병렬적으로 Task를 처리하기 위해서는 Queue를 많이 사용할 수 밖에 없다. concurrent package에서는 안전한 동작을 non-blocking으로 지원해주는 ConcurrentLinkedQueue를 비롯하여, BlockingQueue interface 구현한 여러가지 Queue들을 지원한다.
* __Timing__ : 타임아웃에 관련된 클래스들을 제공한다.
* __Synchronizers__ : 동기화를 위한 클래스들 ; Semaphore, CountDownLatch, CyclicBarrier, Pharser, Exchanger 등을 제공한다.
* __Concurrent Collections__ : 멀티쓰레드 프로그램에서는 Queue를 제외 하고도 여러가지 자료 구조들이 필요할 것이다. concurrent package에서는 ConcurrentHashMap, ConcurrentSkipListMap, ConcurrentSkipListSet, CopyOnWriteArrayList, CopyOnWriteArraySet의 자료구조를 제공하고 있다.
* 또한, java.util.concurrent.locks 패키지와 java.util.concurrent.atomic 패키지를 포함하고 있다.




java의 concurrent skip list
----------------
* * *

Java에서는 jdk 1.6 버전에서 부터 ConcurrentSkipListMap, ConcurrentSkipListSet 클래스를 통해 Skip List를 제공해 주고 있다.


concurrentskiplistmap의 extends, implements
----------------
* * *

__abstract class AbstractMap__

* Map의 기본인 key,와 value 를 관리하기위한 변수 및 메소드 들을 제공해 주고 있다.

__key와 value는 변수__

* volatile transient Set<K> keySet; 
* volatile transient Collection<V> values; 
* transient : 직렬화 과정에서 제외 시킨다는 표시.
* volatile : 비동기적으로 변경될 수 있는 변수라는 표시.

__public interface Serializable__

* Serializable interface 를 implements 하게 되면 별도의 method를 구현하지 않아도 직렬화 기능을 사용할 수 있도록 해 준다.

__public interface Cloneable__

* Object.clone() 을 호출 하였을 때, CloneNotSupportedException을 발생하지 않도록 하기 위하여 표시해 주는 interface

__public interface ConcurrentNavigableMap__

* extends interface ConcurrentMap : 병렬 프로그래밍에서 Map의 메모리 무결성 효과를 주기 위한 interface 이다.
* extends interface SortedMap : implements한 Map 클래스가 정렬된 상태임을 표시해 준다.


마치며....
----------------
* * *

다음 포스팅 부터 본격적으로 소스를 분석할 예정이다.

소스를 한번 훑어본 결과 java에도 goto문이 있다는 사실을 처음 알게 되었다...

생성자 부터, search/ insert/ remove 시 동작하는 과정들을 잘 정해 볼 예정이다.




참조
----------------
http://docs.oracle.com/javase/7/docs/api/java/util/concurrent/package-summary.html


