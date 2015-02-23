---
layout: post
title:  "ConcurrentSkipListMap 생성자"
date:   2015-02-15 15:28:00
categories: java
tags: shit
shortUrl: 
---



ConcurrentSkipListMap 네 종류의 생성자
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

initialize 메소드를 보게 되면 HeadIndex를 제외하고 모두 null로 초기화 되어있다.

각 변수들은 keySet() 와 같은 메소드들을 찾아보면 메소드가 호출될 때, 생성되는것을 볼 수 있다.

	public NavigableSet keySet() {
		KeySet keyset = keySetView;
		return keyset == null ? (keySetView = new KeySet(this)) : keyset;
	}

이것은, 사용자가 keySet() 과 같은 set들을 사용하지 않을 확률이 더 높기 때문에 ConcurrentSkipListMap를 생성할 때 불필요한 오버헤드를 줄이고자 함 인 것 같다.
	

