---
layout: post
title: 애드온에서 document_srl 구하기
date: '2012-08-14 09:46:23 +0900'
categories:
- XpressEngine
tags:
- 애드온
- document_srl
summary: 애드온에서 document_srl을 구하기 위해서는 호출 시점이 글이 작성되고 난 직후인 'after_module_proc'이어야 한다. before_module_proc인 경우 action이 실행되지 않았으므로 document_srl 자체가 존재하지 않는다.
---
애드온에서 document_srl을 구하기 위해서는 호출 시점이 글이 작성되고 난 직후인 'after_module_proc'이어야 한다. before_module_proc인 경우 action이 실행되지 않았으므로 document_srl 자체가 존재하지 않는다.

###addons/addon_name/addon_name.addon.php

{% highlight php %}
<?php
if(!defined('__XE__')) exit();

// 새 글을 작성해서 저장버튼을 누르고 act가 실행된 직후
if(Context::get('act')=='procBoardInsertDocument' &amp;&amp; $called_position == 'after_module_proc' &amp;&amp; $this->toBool()) {
	$document_srl = $this->get('document_srl');
	//아래와 같은 방법으로도 document_srl을 구할 수 있다.
	//$var = $this->variables;
	//$document_srl = $var[document_srl];
	debugPrint("[새글 작성 직후] 문서 번호 : ".$document_srl);
}
?>
{% endhighlight %}
