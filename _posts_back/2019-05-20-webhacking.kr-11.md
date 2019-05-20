layout: post
titile: webhacking.kr 11번 문제 풀이
comments: true
categories: [Webhacking]
tags: [Webhacking]

webhacking.kr 11번 문제풀이
====

[11번 바로가기](http://webhacking.kr/challenge/codeing/code2.html)

문제 페이지에 접속하면 아래와 같은 PHP 코드와 그 결과화면이 보인다.
정규식에는 자신의 IP주소가 포함되어 있다.

```PHP
$pat="/[1-3][a-f]{5}_.*[자기IP주소].*\tp\ta\ts\ts/";

if(preg_match($pat,$_GET[val])) { echo("Password is ????"); }
```

`pat`의 정규식에 매칭되는 `val`을 url에 HTML GET 파라메터로 넣어주면된다.

정규식의 동작은 아래의 사이트에서 아래의 사이트에서 확인할 수 있다.

[REGEXR](https://regexr.com/)


1. 1~3까지 문자 a-f까지 문자 5개_
2. 자기 아이피 주소 0번 이상 반복
3. (tap문자)p(tap문자)a(tap문자)s(tap문자)s 0번 이상 반복

이렇게 문자열을 구성한 뒤 url?val=문자열, 이렇게 url을 구성하면된다.
tap문자는 %09 로 인코딩하여 넣으면 된다.

`?val=1aaaaa_[0.0.0.0]%09p%09a%09s%09s`
