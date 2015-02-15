---
layout: post
title:  "Refactoring Review"
date:   2015-02-14 20:00:00
categories: refactoring,java
tags: 
shortUrl:
---
#Refactoring Review
리팩토링과 개발을 진행하면서 JAVA LOC의 변화를 보면서 리뷰해 보았습니다.

###1월28일 리팩토링 시작 LOC 3016
처음 시작 단계에서의 총10개의 중복과 3016LOC를 볼수 있었습니다.


###1월30일 리팩토링 2일차 LOC 2785(-236)
Refactoring을 진행하면서 기존에 많았던 중복코드들을 제거하니 LOC가 많이 감소 하였습니다. 하지만 기존의 null체크와 예외들에 대한 코드를 추가 적으로 작성하니 
LOC가 다소 올라가게 되었습니다. 

###2월3일 리팩토링 5일차 LOC 2703(-72)
리팩토링 2일차보다 젠킨스에서 찾지 못했던 코드 중복들을 제거하고 Controller와 service역활을 나눠 코드의 효율성을 증가시켰습니다.
그 결과 LOC가 감소한것을 볼수 있었습니다. 

###2월8일 리팩토링 및 추가 구현 LOC 2918(+215)
storage Object, Redis, Interceptor, Spring Security 등을 추가하다 보니 LOC 증가하게 되었습니다. 새로운 기능에 대한 class, method추가 
때문에 LOC가 증가된것으로 판단됩니다.

###2월11일 리팩토링 및 추가 구현 LOC 2985(+67)
추가 기능에 따른 LOC증가가 아닌 대부분 이슈문제로 인한 수정 불가피로 인해 LOC가 증가 하였습니다. 중복코드나 구조 개선등은 하였지만, 오류와 기능 문제로 인해 LOC가 다소 향상 되었습니다.

###마무리..
리팩토링 주부터 현재까지 LOC를 분석 결과 중복코드를 제거하니 LOC가 감소하는 것을 볼수 있었습니다. 하지만 refactoring자체가 LOC를 줄이기 위한 목표가 아닌 설계나 code의 질을 향상 시키다보니 클래스, 인터페이스 추가등으로 오히려 LOC가 증가하는 것을 보게 되었습니다. 