---
title: "webhacking.kr 26번 문제 풀이"
excerpt_separator: "<!--more-->"
categories:
  - webhacking
tags:
  - webhacking
  - wargame
classes: wide
---

[26번 바로가기](http://webhacking.kr/challenge/web/web-11/)

26번 문제에 들어가면 아래와 같은 화면이 나온다.

![26번_1](/img/26번_1.JPG)

링크를 클릭해 "index.phps"로 가면 소스코드가 나온다.


```php
if(eregi("admin",$_GET[id])) { echo("<p>no!"); exit(); }

$_GET[id]=urldecode($_GET[id]);

if($_GET[id]=="admin")
{
@solve(26,100);
}
```

GET 파라메터 중 `id`를 가져와서 정규식으로 "admin"과 비교한 뒤 매칭되면 화면에 'no!'가 출력되고 종료된다.

![26번_2](/img/26번_2.JPG)


매칭이 안되면 `id`값을 URL decode 하고 다시 "admin"과 일치하는 확인하고 일치하면 문제가 풀린다.

`eregi()` 취약점인 `%00`을 삽입해 `%00admin`을 넣어도 문제가 풀리지 않는다. `NULL`문자에 대한 취약점이 없는 버전으로 추측된다.

생각해보면 뒤에서 URL decode를 하니까 "admin"을 URL encode를 두 번해고 넣으면 된다.

URL encode를 한 번 하면 `%61%64%6d%69%6e` 이고 이게 HTTP로 전송되면 'admin'이 되니까 한 번 더 하면 `%2561%2564%256d%2569%256e`으로 된다.
이렇게 하면 첫 번째 if문에서 매칭되지 않고 넘어간 뒤 뒤에 urldecode를 수행하면 다시 'admin'이 되어 문제가 풀린다.


![26번_3](/img/26번_3.JPG)
