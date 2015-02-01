---
layout: post
title: NSTimer, NSDate를 이용해서 간단한 타이머(StopWatch)만들기
date: '2012-07-14 23:24:44 +0900'
categories:
- Objective-C
tags:
- 타이머
- nsdate
- nstimer
- nsdateformatter
summary: 스택 기반 타이머를 만들기 위해서는 먼저 기본적인 타이머를 구현해 보는 것이 필요했다. 아직 iOS에서 Hello World도 겨우 구현하는 내가 타이머를 쉽게 만들 수는 없는 일이었다. 구글링을 하다보니 iOS5의 환경에 맞지는 않지만 그래도 충분히 참고할 수 있는 글을 하나 발견했다.
---
[스택 기반 타이머](http://xrath.com/2012/05/time-management-based-on-stack)를 만들기 위해서는 먼저 기본적인 타이머를 구현해 보는 것이 필요했다. 아직 iOS에서 Hello World도 겨우 구현하는 내가 타이머를 쉽게 만들 수는 없는 일이었다. 구글링을 하다보니 iOS5의 환경에 맞지는 않지만 그래도 충분히 참고할 수 있는 글을 하나 발견했다. 바로 [Sample iPhone Application: StopWatch Tutorial](http://www.apptite.be/blog/ios/sample-ios-application-stopwatch)이다.

Outlet이나 Action의 개념은 이제 이해했기 때문에 UI를 꾸미고 그에 대한 메소드를 입력하는 것에는 큰 어려움이 없었다. 하지만 소스의 일부분은 iOS5에서 더이상 사용하지 않는 방식이라 조금 생소하기도 했다.

{% highlight objective-c %}
- (void)updateTimer
{
    NSDate *currentDate = [NSDate date];
    NSTimeInterval timeInterval = [currentDate timeIntervalSinceDate:startDate];
    NSDate *timerDate = [NSDate dateWithTimeIntervalSince1970:timeInterval];
    NSDateFormatter *dateFormatter = [[NSDateFormatter alloc] init];
    [dateFormatter setDateFormat:@"HH:mm:ss.SSS"];
    [dateFormatter setTimeZone:[NSTimeZone timeZoneForSecondsFromGMT:0.0]];
    NSString *timeString=[dateFormatter stringFromDate:timerDate];
    stopWatchLabel.text = timeString;
    [dateFormatter release];
}
{% endhighlight %}

에서 <code>[dateFormatter release];</code>나 <code>startDate = [[NSDate date]retain];</code>
같은 부분들이다.

새로워진 부분들은 잘 이해가 되지 않는 것도 있지만 [iTunes U - iPad and iPhone Application Development](http://itunes.apple.com/kr/itunes-u/ipad-iphone-application-development/id473757255)를 통해서 알아가고 있으니 차차 좋아질 것이라고 생각한다. 여하튼 소스코드를 참고해서 실행을 시켜보니 Timer가 제대로 가지 않고 00:01:00 만 계속 출력되는 것이었다. 처음에는 무엇이 문제인지 알 수가 없어서 공부하는 셈치고 <code>NSTimer</code>, <code>NSDate</code>의 Reference를 훑어보았다. 하지만 아무리 살펴보아도 문법에 어긋나거나 잘못된 부분을 찾을 수가 없었다.

그래서 nslog를 통해 콘솔로 데이터 값을 확인해 보면서 디버깅하려 했지만 콘솔을 통해 보는 것은 한계가 있었다. 그래서 Breakpoint를 설정하고 변수 값들을 상세히 살펴보았다.

![](/images/2012-07-14/stacktimer.png)

이 화면에서 볼 수 있듯이 처음 Start 버튼을 눌렀을 때의 날짜와 시간을 저장해두는 startDate, 매1초마다 현재의 시간을 저장하는 currentDate, startDate과 currentDate의 시간차이를 계산해서 저장하는 timeInterval, 그리고 1970년 1월 1일을 기준으로(00:00:00) timeInterval 만큼 시간을 더해나가는 timerDate까지 모두 이상 없이 저장이 잘 되고 있다는 사실을 확인할 수 있었다. 다만 일정 형식에 맞게 NSDate 클래스를 NSString으로 변환해주는 dateFormatter를 거치고 나면 항상 값이 00:01:00 으로 변한다는 사실을 알게 되었다.

{% highlight objective-c %}
- (void)updateTimer
{
    NSDate *currentDate = [NSDate date];
    NSTimeInterval timeInterval = [currentDate timeIntervalSinceDate:self.startDate];
    // NSLog([NSString stringWithFormat:@"%f", timeInterval]);
    NSDate *timerDate = [NSDate dateWithTimeIntervalSince1970:timeInterval];
    NSDateFormatter *dateFormatter = [[NSDateFormatter alloc] init];
    [dateFormatter setDateFormat:@"HH:mm:ss"];
    [dateFormatter setTimeZone:[NSTimeZone timeZoneForSecondsFromGMT:0.0]];
    NSString *timeString = [dateFormatter stringFromDate:timerDate];
    //NSLog(timeString);
    [self.timeDisplay setText:timeString];
}
{% endhighlight %}

이 중에서 의심가는 부분은 딱 하나 바로 <code>[dateFormatter setDateFormat:@"HH:mm:ss"];</code>였다. 그래서 <code>NSDateFormatter</code>를 살펴보았고 Reference에서 [Data Formatting Guide](https://developer.apple.com/library/ios/#documentation/Cocoa/Conceptual/DataFormatting/Articles/dfDateFormatting10_4.html#//apple_ref/doc/uid/TP40002369-SW1)를 안내하길래 들어가 보았다. 그리고 가이드에 실려 있는 예제어서 다음과 같은 부분을 찾을 수 있었다.

{% highlight objective-c %}
    [rfc3339DateFormatter setDateFormat:@"yyyy'-'MM'-'dd'T'HH':'mm':'ss'Z'"];
{% endhighlight %}

어라, 연도와 날짜를 나타내는 구분자 외의 문자를 표현하기 위해 작은 따옴표를 사용하는 것이 아닌가. 그래서 updateTimer 메소드를 수정해봤다.

{% highlight objective-c %}
- (void)updateTimer
{
    NSDate *currentDate = [NSDate date];
    NSTimeInterval timeInterval = [currentDate timeIntervalSinceDate:self.startDate];
    // NSLog([NSString stringWithFormat:@"%f", timeInterval]);
    NSDate *timerDate = [NSDate dateWithTimeIntervalSince1970:timeInterval];
    NSDateFormatter *dateFormatter = [[NSDateFormatter alloc] init];
    [dateFormatter setDateFormat:@"HH':'mm':'ss"];
    [dateFormatter setTimeZone:[NSTimeZone timeZoneForSecondsFromGMT:0.0]];
    NSString *timeString = [dateFormatter stringFromDate:timerDate];
    //NSLog(timeString);
    [self.timeDisplay setText:timeString];
}
{% endhighlight %}

그리고 실행을 다시 해보니...

![](/images/2012-07-14/stacktimer_success.png)

빙고!

지나고 나서 돌이켜보면 별것 아닌 것 같지만 이렇게 하나씩 내 힘으로 문제를 해결해 나가는 것이 재미있다. 다음 시간에는 Pause와 Reset을 구현해보고 소스를 조금 수정해서 StackTimer Class를 정의하고 Stack에 StackTimer Object를 저장하는 것까지 완성해봐야겠다. 오늘의 공부 끝!
