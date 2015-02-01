---
layout: post
title: NSURLSession 클래스를 활용해 XE 로그인하기
date: '2014-04-02 12:00:05 +0900'
categories:
- XpressEngine
tags:
- xpressengine
- ios
- objective-c
- 아이폰
summary: 인디스쿨 앱의 핵심적인 기능 중 한 가지는 바로 게시물의 내용을 가져오는 것이다. XE에서는 API를 통해 JSON 형태로 데이터를 가지고 올 수 있다고 하니 일단 iOS쪽에서 URL Request를 보내는 것에 대한 공부가 필요했다. 이와 관련해서 참고할 수 있는 애플 개발자 문서가 있었다.
---
인디스쿨 앱의 핵심적인 기능 중 한 가지는 바로 게시물의 내용을 가져오는 것이다. XE에서는 API를 통해 JSON 형태로 데이터를 가지고 올 수 있다고 하니 일단 iOS쪽에서 URL Request를 보내는 것에 대한 공부가 필요했다. 이와 관련해서 참고할 수 있는 애플 개발자 문서가 있었다. 바로 [URL Loading System Programming Guide](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/URLLoadingSystem/URLLoadingSystem.html#//apple_ref/doc/uid/10000165-BCICJDHA)이다.
이 문서에 따르면 iOS 7.0 이상과 OS X v10.9 이상에서는 NSURLSession이라는 클래스를 사용하면 가능하다고 한다. 그 이전 버전에서 작동하게 하려면 NSURLConnection 클래스를 사용해야 한다고 하는데 일단 이건 패스하기로 했다.
같은 문서의 [Life Cycle of a URL Session](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/URLLoadingSystem/NSURLSessionConcepts/NSURLSessionConcepts.html#//apple_ref/doc/uid/10000165i-CH2-SW1)이라는 섹션을 보면 URL Session이 어떤 순서에 의해 기본적으로 동작하는지를 이해할 수 있다.

1. Create a session configuration. For background sessions, this configuration must contain a unique identifier. Store that identifier, and use it to reassociate with the session if your app crashes or is terminated or suspended.
2. Create a session, specifying a configuration object and a&nbsp;<code>nil</code> delegate.
3. Create task objects within a session that each represent a resource request.Each task starts out in a suspended state. After your app calls <code>resume</code> on the task, it begins downloading the specified resource.
The task objects are subclasses of <code>NSURLSessionTask</code>--<code>NSURLSessionDataTask</code>, <code>NSURLSessionUploadTask</code>, or <code>NSURLSessionDownloadTask</code>, depending on the behavior you are trying to achieve. These objects are analogous to <code>NSURLConnection</code> objects, but give you more control and a unified delegate model.
Although your app can (and typically should) add more than one task to a session, for simplicity, the remaining steps describe the life cycle in terms of a single task.
4. For a download task, during the transfer from the server, if the user tells your app to pause the download, cancel the task by calling<code>cancelByProducingResumeData:</code> method. Later, pass the returned resume data to either the <code>downloadTaskWithResumeData:</code> or<code>downloadTaskWithResumeData:completionHandler:</code> method to create a new download task that continues the download.
5. When a task completes, the <code>NSURLSession</code> object calls the task's completion handler.
6. When your app no longer needs a session, invalidate it by calling either <code>invalidateAndCancel</code> (to cancel outstanding tasks) or <code>finishTasksAndInvalidate</code> (to allow outstanding tasks to finish before invalidating the object).

인터넷에서 예제 자료를 구하려고 검색을 많이 했었는데 iOS의 메모리 관리 체계가 바뀌기 전의 자료들이 있어서 호환이 되지 않는 경우도 많았다. 여러번의 검색을 통해 대략적으로 파악해서 다음과 같이 소스코드를 제작했다.

{% highlight objective-c %}

// 세션의 종류 설정
NSURLSessionConfiguration *defaultConfigObject = [NSURLSessionConfiguration defaultSessionConfiguration];

// 세션 생성
NSURLSession *defaultSession = [NSURLSession sessionWithConfiguration: defaultConfigObject delegate: self delegateQueue: [NSOperationQueue mainQueue]];
NSURL * loginUrl = [NSURL URLWithString:@"http://site/?act=procMemberLogin"];
NSMutableURLRequest * loginRequest = [NSMutableURLRequest requestWithURL:loginUrl];
NSString * params = @"user_id=아이디&amp;password=비밀번호";
[loginRequest setHTTPMethod:@"POST"];<br />
[loginRequest setHTTPBody:[params dataUsingEncoding:NSUTF8StringEncoding]];
NSURLSessionDataTask * dataTask = [defaultSession dataTaskWithRequest:loginRequest completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {

// response body출력을 위한 nsdata -> nsstring 변환
NSString *strData = [[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding];

// 헤더출력<br />
NSLog(@"%@", response);<br />

// response<br />
NSLog(@"%@", strData);
}
}];
[dataTask resume];
{% endhighlight %}

로그인에 성공하든 성공하지 않든 클라이언트에서 해당 URL에 접속하는 데 문제만 없다면 HTTP 헤더는 status code 200을 보내게 된다. 다만 한 가지 발견한 것이 있다면 로그인에 성공했을 경우에는 다음과 같이 로그인에 성공했을 경우 이동하는 URL이 출력된다. (아래의 경우에는 http://site/home)

{% highlight objective-c %}

<NSHTTPURLResponse: 0x10929b630> { URL: http://site/home }

{% endhighlight %}

반면 로그인에 실패했을 경우에는 로그인을 처리하는 URL이&nbsp;그대로 출력되는 것을 확인할 수 있었다.

{% highlight objective-c %}
<NSHTTPURLResponse: 0x10929b630> { URL: http://site/?act=procMemberLogin }
{% endhighlight %}

현재는 로그인을 위해 xe에 직접 POST로 아이디와 비밀번호는 전송하고 xe/modules/member.controller.php가 처리하는 형태라 보안상으로도 좋지 않기 때문에 추후에는 xe에서 자체 모듈을 개발할 필요가 있어 보인다.
