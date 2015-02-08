---
layout: post
title:  "toast rookie"
date:   2015-02-08 20:00:00
categories: jekyll,themes
tags: omg,shit
shortUrl: http://goo.gl/JhfZT9
---
# NHNEnt rookie..!

##1주차: cloud note 기획
미션 cloud note 만들어라. 
1. 노트의 기능은 무엇인가?
    -   노트적기 -> 다양한 형식(MD 또는 HTML), 단순한 텍스트
    -	노트공유 -> URL 공유, 파일 공유, 권한(읽기, 쓰기, 삭제) 공유
2. 유사 모델은 무엇이 있는가?
    -   에버노트 -> 다양한 형식(HTML), URL 공유, 파일 공유, 권한 공유
    -   구글메모 -> 메모 형식의 단순한 텍스트, 간편함, 기본적 공유기능
    -   트렐로 -> 다양한 형식, 노트별로 멤버 공유형식, 협업에 적합함
    -   구글문서 -> 실시간 동시 편집, 다양한 형식, 다양한 노트파일의 종류 등 Office의 개념
    -   네이버메모 -> 구글메모와 유사
    -   네이버오피스 -> 구글문서와 유사
    -   ... 등등등
3. 우리는 어디에 집중할 것인가?
    -   시간이 좀더 주어진다면 다양한 형식(MD 라던가 HTML)방식의 노트 공유를 해보고 싶었음 -> 유사모델은 에버노트
    -   그러나 시간적인 한계가 있으므로 네이버 메모의 기능을 구현하기로 결정
    -   특색을 주기위한 해시태그(#태그)를 추가해보기

위와 같은 기획을 하면서 다음주에는 열심히 개발하자는 마음을 가짐.

##2주차: 프로젝트 개발
1. Github 사용기
	-	SVN을 보다 좋은 점이 뭐지???, 너무 merge가 힘드네, .gitignore 
	-	2주차때는 git에 투자하는 시간이 개발 시간 보다 길었다. 하지만 git사용법을 잘 사용 할수 있게 되었다.
2. 개발
	-	spring mvc, mybatis framwork -> setting이 반이다.
	-	실제 개발은 3일(휴일 포함)

##3주차: 프로젝트 개발(추가 기능)
1. UI변경
	-	도저히 지금 UI로는 추가 기능 힘들다 -> UI변경 -> 기본 기능 파멸..
2. 추가 기능
	-	기본 기능의 파멸로 인해 추가기능 시간 부족 -> 휴일 야근 근무... -> 겨우 완성
	
##4주차: Jenkins
1.	TestCase
	-	JUnit4, Mockito 등을 이용해서 모든 TC개발
2.	Klocwork
	-	Critical, Severe, Error, Unexpected 모든 것을 다 지우기-> 봐도 이유를 모르겠고, 반복되는 이유( 대부분 null, 변수 선언 문제)-> 개발시 유념하자.

##5주차: Refactoring
1.	Redis
	-	Two Server -> Session problem -> Redis를 사용하기
2.	Storage Object
	-	Toast cloud를 이용해서 별도의 파일 서버 -> Json, Https통신
3. Refactoring
	-	code Review -> 설계 구조 리팩토링 및 controller -> service로 이동
	
1주차부터 5주차를 다시 되새겨 보았습니다. 어떻게 보면 짧은 시간이기도 하고 긴 시간이기도 하지만 다양한 경험을 하였고 기술적으로 많이 배웠다.
아직 2주정도 남았지만 끝까지 열심히 해야겠다.<br/>
![1]({{ site.url }}/assets/1.jpg)
![2]({{ site.url }}/assets/2.jpg)
