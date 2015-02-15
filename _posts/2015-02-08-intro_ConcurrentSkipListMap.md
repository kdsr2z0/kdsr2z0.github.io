---
layout: post
title:  "Skip List에 대하여"
date:   2015-02-09 09:28:00
categories: java
tags: shit
shortUrl: 
---

Skip List 란?
----------------

Wikipedia 문서에 의하면, Skip List는  빠르게 Elements 를 검색 할 수 있도록 하는 정렬 된 자료 구조라고 설명하고 있다.
skip list는 추가의 메모리를 더 사용하여 검색/삽입/삭제 를 O(log n) 시간에 가능하도록 하는 자료 구조이다.

![skip list](http://upload.wikimedia.org/wikipedia/commons/thumb/2/2c/Skip_list_add_element-en.gif/640px-Skip_list_add_element-en.gif)

이미지 참조 : http://en.wikipedia.org/wiki/Skip_list

Skip List는 계층 구조를 갖는다. 탐색 시, 상위 계층에 의해 몇번의 skip이 이루어지고, 이러한 skip에 의해 탐색 시간이 절약되게 된다.

Skip List는 확률적인 자료 구조이다. (자세한 내용은 이후, 삽입 알고리즘 설명시 자세한 설명이 포함 될 예정.) 계층의 개수는 확률 매개변수 p에 의해 ![계층 개수](http://upload.wikimedia.org/math/c/2/1/c215210fdc1c5b368ddfde3fd60a1c9a.png) 정도가 적당 하다.

매개변수 p는 보통, 1/2 또는 메모리 효율을 위해 1/4를 사용한다(속도는 1/2가 가장 빠르다).


Skip List의 특징
----------------

자료 구조에서 트리는 메모리를 더 사용하여 O(log n) 시간에 데이터를 검색/삽입/삭제를 수행 할 수 있는 특징을 부여하는 자료 구조이다.

많은 종류의 트리(AVR tree, Red-Black tree, Splay tree, B tree 등등...)는 대부분 트리가 편향되지 않게 유지하기 위한 삽입/삭제 알고리즘을 제공하고 있다.

트리의 편향을 막기위해 많은 효율적인 방법의 리벨런싱 방법을 사용하고 있다. 그런데 트리의 리벨런싱 도중에 자료구조에 대한 접근이 'Lock'되어야 한다는 특징이 있다. 

이러한 특징은 멀티쓰레드 프로그래밍에서 사용하기에 약점이 될 수 있다.

skip List는 tree 자료구조 보다 성능적으로도 밀리고, 더 많은 메모리 공간을 사용 하여야만 O(log n)의 성능을 제공 할 수 있다.

그러나 skip 리스트의 경우 리벨런싱 작업이 필요없다. 따라서, 멀티쓰레드 시스템 구조에서 트리보다 유용한 자료 구조일 수 있는 것이다.



skip list 사용 예제
----------------

* 대표적인 사용 처는 __Redis__ 가 있다. 
* java 에서는 __ConcurrentSkipListSet__과 __ConcurrentSkipListMap__를 통해 멀티쓰레드 자료 구조를 제공하고있다.
* 그 외에, Lock이 적게 사용되어야만 하는  병렬 처리 시스템들에서 사용되고 있다.


마치며...
----------------
앞으로 <http://docs.oracle.com/javase/8/docs/api/> 문서를 참조하면서 Jadclipse 라는 이클립스 도구를 사용하여 ConcurrentSkipListMap의 삽입/삭제 소스를 직접 찾아보면서, 어떤 알고리즘을 사용하는지, 어떤 방식으로 멀티 쓰레드를 지원하고 있는지를 살펴 볼 것이다.




참조
----------------
http://en.wikipedia.org/wiki/Skip_list

