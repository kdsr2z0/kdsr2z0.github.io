---
layout: post
title:  "ConcurrentSkipListMap 변수들"
date:   2015-02-15 15:28:00
categories: java
tags: shit
shortUrl: 
---

ConcurrentSkipListMap에는 다음과 같은 변수들이 선언되어 있다.

	private static final Object BASE_HEADER = new Object();
	private volatile transient HeadIndex head;
	final Comparator comparator;
	private transient KeySet keySet;
	private transient EntrySet entrySet;
	private transient Values values;
	private transient ConcurrentNavigableMap descendingMap;
	private static final Unsafe UNSAFE;
	private static final long headOffset;
	private static final long SECONDARY;
	
HeadIndex, KeySet, EntrySet, Values 등 ConcurrentSkipListMap 에 사용되는 Class들은 대부분 내부 클래스로 선언되어 있다.
내부 클래스에 대한 접근은 내부 변수와 동일하다.
내부 클래스의 특성은 다음과 같다.

* 캡슐화
* 내부에서 외부 변수 접근

그러나 ConcurrentSkipListMap의 내부 클래스들을 살펴본 결과, 두번째 특성인 내부에서 외부 클래스 변수에 직접 접근하지 않고있었다.

필요한 외부 클래스의 변수들을 생성자를 통해 전달받는것을 보아 첫번째 특성을 위해 내부 클래스를 사용하는 것 같다.

또, 내부 클래스를 사용하면, KeySet, EntrySet, Values 등 흔히 쓰이는 내부 클래스의 이름이 다른 클래스의 내부 클래스의 이름과 동일해도 상관없다는 장점도 존재하는 것 같다.

<br><br><br>

Node
---------------- 

SkipList 는 노드와 노드 사이를 링크로 연결하고 있다. 따라서 Node라는 내부 클래스가 존재하고 있다.

class Node의 변수는 

	final Object key;
	volatile Object value;
	volatile Node next;
	private static final Unsafe UNSAFE;
	private static final long valueOffset;
	private static final long nextOffset;

* key는 생성자에서 한번 set되면 변경되지 못하도록 되어있다.
* value와 next 는 volatile이 붙어있는 것을 보아, 비동기적으로 수정될 수 있는 변수임을 알 수 있다.
* 아래 세 변수는 메모리 관리와 관련된 변수들이다.

__value__ : value는 데이터를 가지거나, this 혹은 BASE_HEADER를 가리킬 수 있다.<br>
__key__  : key는 말 그대로 노드를 구별하고, 정렬할 때 사용하는 노드의 key를 말한다. value 가 this나 BASE HEADER를 가리킬 때, key는 null 일 수 있다.<br>
__next__  : 다음 노드를 가리킨다.

Node 클래스의 생성자

	Node(Object obj, Object obj1, Node node) {
		key = obj;
		value = obj1;
		next = node;
	}

	Node(Node node) {
		key = null;
		value = this;
		next = node;
	}

Node 클래스에서 제공하는 method 는 다음과 같다.

* casValue(Object obj, Object obj1) : 기존 value 값이 obj와 같다면 obj1로 변환 시킴.
* casNext(Node node, Node node1) : 기존 next 값이 node와 같다면 node1로 변환 시킴.
* helpDelete(Node node, Node node1) : 현재 노드가 node, node1 사이라면 현재 노드를 삭제시킴.

<br><br><br>

Index
---------------- 

Index 클래스는 SkipList의 계층 구조를 만들어주는 클래스 이다.

다음은 Index 클래스의 변수들이다.

	final Node node;
	final Index down;
	volatile Index right;
	private static final Unsafe UNSAFE;
	private static final long rightOffset;

* right 변수만 비동기적으로 수정될 수 있는 변수임을 알 수 있다.
* 아래 두 변수는 메모리 관리와 관련된 변수들이다.

__node__ : Index가 가리키고 있는 node를 저장한다.<br>
__down__ : 하위 Level으로 이동한다.<br>
__right__ : 다음 Index로 이동한다.

Index 클래스의 생성자

	Index(Node node1, Index index, Index index1) {
		node = node1;
		down = index;
		right = index1;
	}
	
Index 클래스에서 제공하는 method 는 다음과 같다.

* link(Index index, Index index1) : 기존 right 가 index 와 같다면 index1로 변환 시킴.
* unlink(Index index) : index 가 right라면 right = index.right
* indexesDeletedNode() : node.value = null 로 처리한다. (이렇게 처리한 후, doPut 과정에서 unlink 시키는 듯 하다.)

<br><br><br>

HeadIndex
---------------- 

SkipList의 시작점으로, HeadIndex의 node는 __Object BASE_HEADER = new Object()__ 를 value로 갖는 Node를 가리키고 있다.

head는 새로운 level이 생기게 되면 추가될 수 있다.

<br><br><br>

cas
---------------- 

위에서 node와 index의 method에 붙어있는 cas 는 compareAndSwap의 줄인말이다.

sun.misc.Unsafe 에서 제공하는 함수를 호출하기 때문에 method 이름 마다 cas를 붙인 듯 하다.

compareAndSwap 는 (Object o, long offset, Object expected, Object x) parameter를 전달받는데, o의 메모리  offset 위치의 값이 expected 와 일치한다면 x 로 변환 시킨다.

compareAndSwap의 경우 Atomically 하게 데이터를 업데이트 하기 때문에, 병렬 프로그래밍에서 데이터의 동기화 문제를 해결하기 위해 사용하는 듯 하다.


<br><br><br>

참조
---------------- 

* <http://www.docjar.com/docs/api/sun/misc/Unsafe.html>