---
layout: post
title:  "ConcurrentSkipListMap 생성자"
date:   2015-02-15 15:28:00
categories: java
tags: shit
shortUrl: 
---

ConcurrentSkipListMap 에 사용되는 Class들은 대부분 내부 클래스로 선언되어 있다.
내부 클래스에 대한 접근은 내부 변수와 동일하다.
내부 클래스의 특성은 다음과 같다.

* 캡슐화
* 내부에서 외부 변수 접근

그러나 내부 클래스들을 살펴본 결과 두번째 특성인 내부에서 외부 클래스 변수에 직접 접근하지 않고, 생성자를 통해 전달받는것을 보아 첫번째 특성을 위해 내부 클래스를 사용하는 것 같다.
또, 내부 클래스를 사용하면, 내부 클래스의 이름이 다른 클래스의 내부 클래스의 이름과 동일해도 상관없다는 장점도 존재하는 것 같다.

<br><br><br>

ConcurrentSkipListMap는 네 종류의 생성자
---------------- 

* ConcurrentSkipListMap() : 기본 생성자
* ConcurrentSkipListMap(Comparator comparator1) : 키의 정렬 순서를 임의로 정할 수 있다. 참조 : [Comparator Class](http://docs.oracle.com/javase/7/docs/api/java/util/Comparator.html)
* ConcurrentSkipListMap(Map map) : map의 데이터를 ConcurrentSkipListMap에 모두 put 한다.
* ConcurrentSkipListMap(SortedMap sortedmap) : 정렬된 맵을 ConcurrentSkipListMap에 모두 put 한다.

<br><br><br>

initialize()
---------------- 
각 생성자는 initialize() 함수를 호출 한다.

	private void initialize() {
		keySet = null;
		entrySet = null;
		values = null;
		descendingMap = null;
		head = new HeadIndex(new Node(null, BASE_HEADER, null), null, null, 1);
	}

생성자에서 HeadIndex 하나를 생성해 주고 있다.

<br><br><br>

buildFromSorted(SortedMap sortedmap)
---------------- 
ConcurrentSkipListMap(SortedMap sortedmap) 생성자의 경우, buildFromSorted(SortedMap sortedmap)메소드를 호출하고 있다.
이 메소드 첫 부분에 보면, 흥미로운 소스가 보였다.

	ArrayList arraylist = new ArrayList();
	for (int i = 0; i <= headindex.level; i++)
		arraylist.add(null);

	Object obj = headindex;
	for (int j = headindex.level; j > 0; j--) {
		arraylist.set(j, obj);
		obj = ((Index) (obj)).down;
	}

ArrayList 의 경우, add(int paramInt, E paramE) 라는 메소드를 제공해 주고 있음에도 불구하고,
루프 두번을 수행하여 데이터를 set해주고 있는것을 볼 수 있다.

마치, C 에서 malloc으로 공간을 확보 하고, data를 set 하는것과 같아 보인다.

이 경우, 데이터가 얼마만큼 들어올지 정확히 알 고 있는 상태이기 때문에, 처음 for문으로 한번의 메모리 할당을 수행하고, 두번째 for문으로 data를 set함으로써 메모리 할당을 한번만 하도록 한 것으로 보인다.

실제로 메모리 할당이 한번만 수행되는지는 확신하진 못하지만, 저런식으로 두번의 for문을 호출한 이유가 그것 뿐이라고 짐작한다.

만약 arraylist.add(int paramInt, E paramE) 를 통헤 data를 추가하였다면, ArrayList가 내부적으로 효율적인 메모리 할당을 한다고 하더라도, 여러번의 메모리 할당과 data 복사가 발생했을 것이다.

또, 아직 코드를 전체 다 보지는 못하였지만, 비슷한 알고리즘의 코드가 여러군데 보이는 것 같았다.(예를 들어 ConcurrentSkipListMap 에 data를 put하는 코드)
이렇게 코드가짜여진 것도 함수를 호출하는데에 대한 낭비를 줄이기 위함으로 보인다.
 
<br><br><br>

마치며...
---------------- 
약간 편법처럼 보이지만, Java 에서 성능을 위해서 할 수 있는 여러가지 코딩 스타일이 존재한다는 것을 다시한번 알게 되었다. 다음 포스트에서는, Node class와 Index class, HeadIndex class 를 살펴 보고 어떤식으로 skip list 구조를 만드는지 살펴볼 예정이다.


