---
layout: post
title:  "Spring AOP "
date:   2015-02-13 19:00:00
categories: jekyll,themes
tags: omg,shit
shortUrl: http://goo.gl/JhfZT9
---
손쉬운 Spring AOP 적용

Spring 프로젝트를 진행하면서 모든 DAO 객체에서 일정한 공통로직이 발생하게 되었다. 이러한 공통사항에 대해서 중복 코드를 줄이기 위해 상속개념을 이용해서 해결하려고 했지만
구조상 많은 부분의 변경이 불가피해 Spring의 AOP을 적용시켜 문제를 해결하게 되었다.

Ex) 실제로 AOP가 적용된 코드
{% highlight java %}
@Before(value = "execution(public * com.toast.happiness.controller.BoardController.*(..))")
	public void beforeAdvice(JoinPoint joinPoint){
		//TODO 
	}
{% endhighlight %}

위의 코드를 보면 여러가지 AOP개념들을 설명해 보겠다.

(1) pointcut
어느 위치나 상황(메소드)에서 적용될지를 지정 하는 것을 의미하는 것이다.PointCut인 excution내부를 보자. 실제 어느 객체의  메소드가 실행 될 때 메소드 내부를 실행할지를 선택 할 수 있다.
com.toast.happiness.controller.BoardController객체 내부의 메소드 중에 public 접근 권한인 것들이 실행 될때 적용 된다는 것이다.
pointcut 지정자는 아래와 같은 형식이다.
{% highlight xml %}
excution(<접근 수정자 패턴><반환 형식 패턴><선언 형식 패턴><메소드 이름 패턴>(<매서드 매개변수 패턴>)<예외 발생 패턴>)
{% endhighlight %}

Ex 다양한 적용
{% highlight java %}
expression="execution(Integer test01.ver2.BoardService.write())"
: 리턴 타입이 Integer인 BoardService의 write 메서드

expression="execution(* get*(*))"
: getter이면서 파라미터가 1개인 메서드들
{% endhighlight %}

(2) Advice 
Advice는 공통적으로 적용될 모듈이나 로직을 의미한다. 위의 예시를 보면 @Before라는 어노테이션이 적용된 것을 볼 수 있다. before Advice는 대상 메소드가 적용되기전에 실행 되는 것을 의미한다.
반대로 after Advice는 대상 메소드가 적용 된 후에 호출 되는것이다. 이 두가지를 혼합으로 Around Advice가 적용된 예시 이다. 
   
{% highlight java %}
@Around(value = "execution(public * com.toast.happiness.controller.BoardController.*(..))")
	public void aroundAdvice(ProceedingJoinPoint pjp){
		
		//TODO Before

		pjp.proceed();
		
		//TODO After
	}
{% endhighlight %}
ProceedingJoinPoint의 proceed()를 실행하면 대상 메소드가 호출 된다. 호출 앞뒤로 원하는 모듈을 실행 시켜 줄 수 있다.

(3)JoinPoint

Advice가 적용될 메소드를 의미하고 메소드정보들을 가지고 있다. 이 객체를 파라미터로 전달 받기 위해서는 첫번재 파라미터로 지정해야한다.
JoinPoint는 대상메소드의 이름, 객체, 인자정보등을 알수 있는 다음과 같은 메소드들과 내부 객체들이 있다.

JoinPoint 인터페이스는 호출되는 대상 객체, 메서드 그리고 전달되는 파라미터 목록에 접근할 수 있는 메서드를 제공
  -Signature getSignature( ) - 호출되는 메서드에 대한 정보를 구함<br/>
  -Object getTarget( ) - 대상 객체를 구함<br/>
  -Object[ ] getArgs( ) - 파라미터 목록을 구함 <br/>
  
org.aspectj.lang.Signature 인터페이스는 호출되는 메서드와 관련된 정보를 제공하기 위해 다음과 같은 메서드를 정의
  - String getName( ) - 메서드의 이름을 구함<br/>
  - String toLongName( ) - 메서드를 완전하게 표현한 문장을 구함(메서드의 리턴 타입, 파라미터 타입 모두 표시)<br/>
  - String toShortName( ) - 메서드를 축약해서 표현한 문장을 구함(메서드의 이름만 구함)<br/>

실제로 적용된 예를 보겠다.

{% highlight java %}
@After (value = "execution(public * com.toast.happiness.model.dao.*DaoImpl.*(..))")
	public void selectAfter(JoinPoint joinPoint){
		
		Signature signature = joinPoint.getSignature();
		Object target = joinPoint.getTarget();
		Object[] args = joinPoint.getArgs();
		
		//connection 위치 변경
		CustomerContextHolder.setCustomerType(args[0]);
	}
{% endhighlight %}
 파라미터들을 이용하여 공통된 부분들을 처리한 예제를 볼 수 있었다. 대상 메소드 전에 미리 파라미터를 가지고 SqlSession에 connection서버 위치를 변경해주는 역할을 적용하여 코드의 중복을 제거 하였다. 이렇듯 AOP를 이용하여 공통 관심사(모듈)들을 추출하여 적용 방안들을 더욱더 생각해 본다면 프로젝트에 효율을 높일수 있을 것이다.
