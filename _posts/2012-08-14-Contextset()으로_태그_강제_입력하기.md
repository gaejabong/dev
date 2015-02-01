---
layout: post
title: Context::set()으로 태그 강제 입력하기
date: '2012-08-14 18:00:07 +0900'
categories:
- XpressEngine
tags:
- Context
- set
- 태그
- tags
summary: 글 작성시에 애드온을 호출하고 Context::set('tags', $value)를 실행해도 애초에 태그 필드가 비어있으면 값이 저장되지 않는다. Context.Class.php 파일을 분석하니 set() 함수가 다음과 같이 정의되어 있었다.
---
글 작성시에 애드온을 호출하고 <code>Context::set('tags', $value)</code>를 실행해도 애초에 태그 필드가 비어있으면 값이 저장되지 않는다. Context.Class.php 파일을 분석하니 set() 함수가 다음과 같이 정의되어 있었다.

{% highlight php %}
 **
 * @brief set a context value with a key
 **
function set($key, $val, $set_to_get_vars=0) {
	is_a($this,'Context')?$self=&amp;$this:$self=&amp;Context::getInstance();
	$self->context->{$key} = $val;
	if($set_to_get_vars === false) return;
	if($val === NULL || $val === '')
	{
		unset($self->get_vars->{$key});
		return;
	}
	if($set_to_get_vars || $self->get_vars->{$key}) $self->get_vars->{$key} = $val;
}
{% endhighlight %}

코드를 보면 $set_to_get_vars를 1로 설정해 주면 $self->get_vars를 통해 $val 값을 초기화하는 것을 알 수 있다. 따라서 다음과 같이 메서드를 호출하면 된다.
{% highlight php %}
Context::set('tags',$val, 1);
{% endhighlight %}
