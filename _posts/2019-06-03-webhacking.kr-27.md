---
title: "webhacking.kr 27번 문제 풀이"
excerpt_separator: "<!--more-->"
categories:
  - webhacking
tags:
  - webhacking
  - wargame
classes: wide
---

[27번 바로가기](http://webhacking.kr/challenge/web/web-12/)

27번 문제에 들어가면 아래와 같은 화면이 나온다. SQLi 문제임을 알 수 있다.

![27번_1](/img/27번_1.JPG)

페이지 소스를 보면 "index.phps"에 소스코드가 있다고 나온다. 들어가보면 php 소스코드를 볼 수 있다.

```php
if($_GET[no])
{

if(eregi("#|union|from|challenge|select|\(|\t|/|limit|=|0x",$_GET[no])) exit("no hack");

$q=@mysql_fetch_array(mysql_query("select id from challenge27_table where id='guest' and no=($_GET[no])")) or die("query error");

if($q[id]=="guest") echo("guest");
if($q[id]=="admin") @solve();

}
```

`union`, `select`, `limit` 등 SQLi에 사용되는 문자를 일부 필터링하고 'challenge27_table'에서 `id=guest`이고 `no=(사용자입력)`인 row의 id를 가져온다.
이후 가져온 id가 'admin'이면 문제가 풀린다.


이런류의 문제는 뻔하다. table에 guest와 admin이 있는데 앞에서 `id=guest`인 조건을 무시하게끔 한 뒤 `id=admin`인 row만 가져오면 된다.
우선 guest의 no를 알아내기 위해 값을 입력해보자.


`no=1`로 하면 아래와 같이 "guest"라고 출력된다. 즉 guest의 no는 1이다.

![27번_2](/img/27번_2.JPG)

`no=2`로 입력하면 "query error"가 출력된다. 이때는 쿼리문에 해당되는 row가 하나도 없기 때문에
에러가 발생한다.

![27번_3](/img/27번_3.JPG)

그냥 단순히 `id='guest' and no=(2) or id='admin'` 뭐 이런식으로 앞에 있는 조건문을 무시하고 `id=admin`인 row를 가져오면 된다.

근데 문제는 `no=(사용자입력값)` 이런식으로 되어 있어서 괄호의 짝을 맞춰야 하는데, 여는 괄호인 `(`와 주석인 `#`가 필터링된다는 것과 `=`가 필터링된다는 것이다.


주석은 `#`를 많이 사용하지만 `-- `으로도 사용할 수 있다. 하이픈 뒤에 반드시 한칸의 공백이 있어야 한다.
`=`는 `like`, `in()`, `strcmp()` 등으로 우회할 수 있는데 괄호를 사용할 수 없으니까 `like`를 사용하면 된다.


이를 통해 아래와 같은 쿼리를 만들 수 있다.
`2) or id like 'admin' -- ` 이 쿼리가 들어가면 php 코드는 아래와 같아진다.

```php
$q=@mysql_fetch_array(mysql_query("select id from challenge27_table where id='guest' and no=(2) or id like 'admin' -- )")) or die("query error");
```

주석 뒤쪽이 무시되고 `id=admin`인 row만 선택되서 문제가 풀려야 되는데, 아래와 같이 "query error"이 뜬다.
대부분의 워게임 문제가 그렇듯 저기 나와 있는 코드가 실제 서버의 소스코드는 아니다. 쿼리에 문제는 없지만 이걸로 문제를 풀 수 는 없어 보인다.


![27번_3](/img/27번_3.JPG)

다른 방법은 admin의 no를 추측하는 건데 `no=1`은 guest니까 `no=2`부터 해보자. 아래와 같은 쿼리를 사용하면 된다.

`2) or no like 2 -- `

![27번_4](/img/27번_4.JPG)

운좋게 문제가 풀린다. admin의 no는 2였던 것이다.
