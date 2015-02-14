---
layout: post
title:  "Spring Security 1장"
date:   2015-02-08 19:00:00
categories: Spring,Security
tags: omg,shit
shortUrl: http://goo.gl/JhfZT9
---
Spring Security란?

스프링 시큐리티는 매우 지증적이며 대부분 간단한 선언만으로 작동하므로 대량의 보안 관련 코드를 손쉽게 절약해줍니다. 스프링 시큐리티를 이용하는 것만으로도 최첨단 보안장치를 설치하는 것과 동일한 효과를 얻을 수 있습니다.
적용하는 방식은 강력하면서도 쉽습니다. 단 몇십줄의 코드만으로도 대형 웹서비스사와 비슷한 수준의 보안을 유지할 수 있다는 장점이 있습니다. 
이러한 장점에도 불구하고 한국에서는 스프링 시큐리티에 대한 활용이 미비한 상태입니다. 제대로된 포럼글이나 최신버전에 맞는 설명이 많지 않다는 것입니다.  스프링 시큐리티가 ACEGI란 이름으로 시작해, 세상에 나온지 벌써 10년 가까이 됬음에도 아직까지 큰 관심이 없다는 것은 한국이 아직도 웹서비스의 발전이 미미하거나 보안에 대해 크게 간과하면서 많은 관심이 두고있지 않다는 사실일지도 모릅니다.

1-1장 보안이란?
스프링 시큐리티는 두가지 관점이 있습니다. Authentication(인증), Authorization(권한 부여)

인증: 보통 3가지로 분류
(1)크리텐셜(redential:자격) 기반 인증: 
웹에서 주로 사용하는 인증방식으로써, 즉 권한을 부여 받는데 1차례 인증과정 필요 대게 사용자명 비밀번호 입력 받아 확인한다. 스프링 시큐리티에서는 아이디를 프린시플(principle), 비밀번호를 크리덴셜(credential)이라고 부르기도 합니다. 스프링 시큐리티는 크리텐셜 기반 인증이다.

(2)이중 인증(Two-factor authentication):
한번에 2가지 방식으로 인증, 일반적으로 로그인과 보안 인증서 같은 2가지 방법으로 인증한다. 인증이 하나 더 추가 됨으로써 프로그래밍 적으로 변화해야 할 부분은 상당히 광범위해진다.

(3)물리적 인증:
웹의 영역이 아니지만 효과적인 보안 수단 중에 하나이다. 예를 들어 컴퓨터 지문 인식


권한 부여

부여된 권한(Granted Autority) : 적절한 절차로 사용자가 인증되었다면 권한을 부여(Granted Authority)해야 할 것입니다. 회원가입 등을 통해 반영구적인 권한이 부여됬다면 우리는 이 회원에게 부여된 권한을 어딘가에 저장해야 된다.

리소스의 권한(Intercept) : 사용자의 권한만 있다고 보안이 제대로 동작할리는 없습니다. 보안이란 본래 권한이 없는 자들이 원천적으로 리소스에 접근할 수 없도록 막아내는 것이기 때문입니다. 그런 의미에서 적절한 권한을 가진자만 해당 자원에 접근할 수 있도록 자원의 외부요청을 원천적으로 가로채는 것(Intercept)이 웹보안, 그 중 권한부여(Authorization)의 핵심 원칙이라 할 수 있겠습니다.

우리가 보안에서 "어떤 방식으로 권한을 부여할지" , "해당 리소스에 어떻게 권한을 부여할지" 등을 해결 해야된다. 이러한 부분들을 스프링 시큐리티에서 편리하게 보안을 해줄수 있도록 기능을 제공한다.


1-2장 리소스 권한(Intercept)

스프링 시큐리티는 리소스에 접근권한을 설정을 intercept 방식으로 작동하고 있다. 아무리 서버 성능이 좋더라도 일일이 리소스에 권한을 설정할 수 없는 노릇이다. 대신 MVC에서 DispatcherServlet처럼 멋지게 클라이언트의 요청을 가로챌 수만 있다면 간단히 문제를 해결할 수 있을 것이다.
스프링 시큐리티는 DispatcherServlet이나, AOP를 이용해 프록시를 생하지 않고 DelegatingFilterProxy클래스를 사용합니다. DelegatingFilterProxy를 이용하면서도 AOP 포인트컷의 활용 또한 가능합니다.

<br/><img src="http://cfile7.uf.tistory.com/image/1918D43A4FBCE44401E08A"/>

web.xml에 DelegatingFilterProxy를 등록해 봅시다.
{% highlight xml %}
<filter>
    <filter-name>springSecurityFilterChain</filter-name>
    <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
</filter>
<filter-mapping>
    <filter-name>springSecurityFilterChain</filter-name>
    <url-pattern>/-</url-pattern>
</filter-mapping>
{% endhighlight %}

주의할점은 filter-name 의 값이 반드시 springSecurityFilterChain이어야 한다는 점입니다. DelegatingFilterProxy 클래스는 setTargetBeanName(String)이라는 메서드를 갖고 있는데 이 메서드는 실제 요청을 처리할 필터를 주입받습니다. 만약 이 메서드를 통해 구현할 필터빈을 주입받지 못한다면 DelegatingFilterProxy 클래스는 기본값으로 filter-name의 값과 동일한 빈이 스프링 컨텍스트에 존재하는지를 검색하게 됩니다. DelegatingFilterProxy 클래스가 springSecurityFilterChain이란 빈이 필요하다고 직접 빈을 만들어줄 필요는 없습니다. 왜냐하면 springSecurityFilterChain은 스프링 시큐리티의 inner bean이기 때문에 자동으로 생성되기 때문이죠.
DispatcherServlet와 DelegatingFilterChain이 가로채면 우선순위는 필터가 우선순위가 됩니다. xml에 필터로 등록되었기 때문입니다.

필터를 이용해 모든 URL이 DelegatingFilterProxy를 통과하도록 컨테스트를 설정합니다. 보안에 대해서 따로 관리해줄 컨테스트 파일(security-context.xml)을 만듭니다.
web.xml에 등록을 하셔야 합니다.
<br/><img src="http://cfile8.uf.tistory.com/image/157646464FBCE445338403"/>
<br> 2장을 기대하세요

출처 [linuxism]

[linuxism]:   http://linuxism.tistory.com/671