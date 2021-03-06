---
layout: post
title:  "오라클 옵티마이저"
date:   2015-06-14 15:28:00
categories: java
tags: shit
shortUrl: 
---

Database 옵티마이저
---------------- 

옵티마이저는 데이터베이스의 성능을 좌우하는 가장 핵심 요소중 하나이다.

옵티마이저의 성능이 뛰어나 빠른시간안에 가장 최적화된 실행계획을 얼마나 잘 뽑아내느냐에 따라 DBMS의 성능이 결정된다고 보면된다.

이 포스팅에서는 오라클에서 사용되고 있는 옵티마이저에 대해 간략히 정리한다.

옵티마이저의 종류
---------------- 

* 규칙기반 옵티마이저
* 비용기반 옵티마이저

규칙기반 옵티마이저
---------------- 

* 실행계획 생성 시 몇가지 규칙을 가지고 실행계획을 생성
* 규칙의 주요 요소는 인덱스 유무와 종류, 포함된 연산자의 종류, 테이블의 종류이다.

Oracle 규칙기반 옵티마이저의 자주사용되는 규칙
---------------- 

오라클에서는 15가지 종류의 옵티마이저 규칙이 있는데 자주사용되는 규칙은 다음과 같다.

* 규칙 1 : Single row by rowid : rowid를 이용하여 하나의 행에 액세스하는 방식
* 규칙 4 : Single row by unique or primary key : Unique Index를 통해서 하나의 행에 액세스하는 방식
* 규칙 8 : Composit index : 복합 인덱스와 '=' 연산을 사용하여 검색하는 방식
* 규칙 9 : Single culumn index: 단일 컬럼 인덱스와 '=' 연산을 사용하여 검색하는 방식
* 규칙 10 : Bounded range search on indexed columns : 인덱스가 생성되어 있는 컬럼 양쪽 범위를 한정하여 검색하는 방식
* 규칙 11 : Unbounded range search on indexed columns : 인덱스가 생성되어 있는 컬럼에 한쪽 범위만 한정하는 형태로 검색하는 방식
* 규칙 15 : Full table scan : 전체 테이블을 액세스 하면서 조건절에 주어진 조건을 만족하는 행만 결과로 추출하는 방식

위 규칙에서말하는 rowid는 저장된 데이터로 바로 접근가능한 식별정보를 말한다.

규칙을 보면 Range를 검색하는 조건보다 '=' 조건이 우선순위가 높고, 인덱스가 있을수록 우선순위가 높다.

가장 우선순위가 낮은 경우는 Full table scan 즉, 전체 테이블 엑세스 이다.


비용기반 옵티마이저
---------------- 

규칙기반 옵티마이저에서 '=' 조건의 우선순위가 범위를 검색하는 조건보다 높다고 설명하였다.

또, 규칙기반 옵티마이저에서 전체 테이블 스켄보다 인덱스 스켄의 우선순위가 높다.

그러나 실제로 쿼리가 동작할 때 '=' 보다 Range가 더 적은 Row의 결과를 만드는 경우가 존재할 것이다. 즉, 범위 탐색의 비용이 더 적은 경우가 발생하게 된다.

그래서 비용기반 옵티마이저는 시스템 통계정보, 객체(테이블) 통계정보를 저장하고 있다가 실행계획을 생성하는 과정에서 사용한다.

좀 더 실제와 가깝게 비용을 계산하게 되는것이다.

그러나 통계정보가 부족할 때 정확한 비용 예측이 불가능해 비효율적인 실행계획을 생성할 수 있다는 단점이 존재한다.

오라클의 옵티마이저는 규칙기반 옵티마이저와 비용기반 옵티마이저를 적절히 섞어 빠르게 가장 효율적인 실행계획을 찾아낸다.(정확히 어떻게 찾아내는지는 공부한 책에 나와있지 않았다...)


비용기반 옵티마이저의 구성
---------------- 

* 질의 변환기
	* 사용자가 작성한 SQL문을 처리하기 용이한 형태로 변환
* 대안계획생성기
	* 동일한 결과를 생성하는 다양한 대안 계획을 생성하는 모듈
	* 연산의 적용 순서 변경, 연산 방법 변경, 조인 순서 변경
	* 이론적으로는 각 변경 방법마다 n!개의 대안 계획을 생성할 수 있다.
	* 실제로는 모든 대안계획을 생성하지 않고 적절히 제한을 두어 제한선까지의 대안계획만 생성한다.

* 비용예측기
	* 통계정보를 이용하여 실제 비용을 예측
* 딕셔너리
	* 비용예측기에 통계정보를 제공

실행계획
---------------- 

DBMS가 사용자가 질의한 SQL문을 어떻게 처리할지 예측하여 보여주는 것이다.


실행계획의 구성요소
---------------- 

실행계획은 다음과 같은 정보를 제공한다.

* 조인 순서
* 조인 기법 (NL join, Hash join, Sort merge join)
* 액세스 기법 (index)
* 최적화 정보
	* Cost : 상대적 비용 정보
	* Card : 예측된 결과 집합의 건수
	* Bytes : 사용되는 메모리의 용량

참조
---------------- 
책 : SQL 전문가 가이드 2013 Edition , KODB(한국데이터베이스진흥원)