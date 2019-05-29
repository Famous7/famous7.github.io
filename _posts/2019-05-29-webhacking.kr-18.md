---
title: "webhacking.kr 18번 문제 풀이"
excerpt_separator: "<!--more-->"
categories:
  - webhacking
tags:
  - webhacking
  - wargame
classes: wide
---

[18번 바로가기](http://webhacking.kr/challenge/web/web-32/index.php)

문제 페이지에 접속하면 아래와 같은 화면이 나온다. SQLi 관련 문제라는것을 알 수 있고, 친절하게도 indexs.php에 소스코드가 있다.
indexs.php의 php 소스코드만 보면 아래와 같다.

![18번](/img/18번_1.JPG)

```php
if($_GET[no])
{

  if(eregi(" |/|\(|\)|\t|\||&|union|select|from|0x",$_GET[no])) exit("no hack");

  $q=@mysql_fetch_array(mysql_query("select id from challenge18_table where id='guest' and no=$_GET[no]"));

  if($q[0]=="guest") echo ("hi guest"); 
  if($q[0]=="admin")
  {
    @solve();
    echo ("hi admin!");
  }

}
```

DB에서 GET 파라메터 `no`에 넣은 값에 해당하는 id를 가져오고 id가 "admin"이면 문제가 풀린다. 쿼리를 보면 where절에 `id=guest`로 id값은 고정되어 있다.
`no=1`로 하면 id가 "guest"로 나온다. table에서 guest의 no는 1임을 알 수 있다. `no=2`로 하면 당연히 맞는 레코드가 없기 때문에 아무것도 출력되지 않는다.

![18번](/img/18번_2.JPG)

쿼리 실행전 정규식으로 SQLi에 사용되는 문자를 필터링하는 과정에서 `or` 등의 문자는 필터링하지 않기 이를 이용해 where절을 조작하면 쉽게 풀 수 있다.
연산자 우선순위가 `and`가 `or`보다 높기 때문에 `where id='guest' and no=2 or 1=1`로 입력하면 `(where id='guest' and no=2) or 1=1`와 같다.
이렇게 입력하면 table의 모든 레코드가 선택된다. `limit`를 사용해서 하나씩 가져오면 된다. 공백문자와 탭문자가 필터링되기 때문에 라인피드(%0a)를 이용하면 된다.

`2%0aor%0a1=1%0alimit%0a1,1`이렇게 입력해서 table의 첫 번째 id 값을 가져올 수 있다. 운좋게 바로 문제가 풀렸다. table의 첫 번째 id가 "admin"이다.
`2%0aor%0ano=2` 이런식으로 `no` 값을 직접 지정해도 된다. 이렇게 입력해도 문제가 풀리는데 admin의 no가 2이기 때문이다.

![18번](/img/18번_3.JPG)
