---
title: "webhacking.kr 20번 문제 풀이"
excerpt_separator: "<!--more-->"
categories:
  - webhacking
tags:
  - webhacking
  - wargame
classes: wide
---

[20번 바로가기](http://webhacking.kr/challenge/codeing/code4.html)

20번 문제에 들어가면 아래와 같은 화면이 나온다. 자바스크립트 문제니까 프로그래밍하지 말라고 쓰여있다.


![20번](/img/20번_1.JPG)


입력폼이 많은데 우선 소스코드를 확인해보자.

```html
<html>
<head>
<title>Challenge 20</title>
<style type="text/css">
body { background:black; color:white; font-size:10pt; }
input { background:silver; color:black; font-size:9pt; }
</style>
</head>
<body>
<center><font size=2>time limit : 2</font></center>
<form name=lv5frm method=post>
<table border=0>
<tr><td>nickname</td><td><input type=text name=id size=10 maxlength=10></td></tr>
<tr><td>comment</td><td><input type=text name=cmt size=50 maxlength=50></td></tr>
<tr><td>code</td><td><input type=text name=hack><input type=button name=attackme value="skjnmvacma"
 style=border:0;background=lightgreen onmouseover=this.style.font=size=30 onmouseout=this.style.font=size=15></td></tr>
<tr><td><input type=button value="Submit" onclick=ck()></td><td><input type=reset></td></tr>
</table>
<script>
function ck()
{

if(lv5frm.id.value=="") { lv5frm.id.focus(); return; }
if(lv5frm.cmt.value=="") { lv5frm.cmt.focus(); return; }
if(lv5frm.hack.value=="") { lv5frm.hack.focus(); return; }
if(lv5frm.hack.value!=lv5frm.attackme.value) { lv5frm.hack.focus(); return; }

lv5frm.submit();

}
</script>

<br>

do not programming!<br>

this is javascript challenge

</body>
</html>
```

'id', 'cmt', 'hack' 입력폼에 각각 값을 입력한 뒤 submit 버튼을 누르면 'hack' 입력폼 옆에 있는 'attackme'라는 버튼에 있는 값과 'hack'의 값을 비교해 일치하는 경우 submit이 된다.


'id', 'cmt'에 아무값이나 채우고, 'hack'과 'attackme'값을 일치하게 입력하고 submit을 눌러보자.


![20번_2](/img/20번_2.JPG)

'Wrong'이라고 출력되고 문제가 클리어되지 않는다. 위에 'time limit 2'가 쓰여있는데 아마 이걸 2초안에 해야 문제가 풀리는 것 같다. 'attackme'에 있는 값은 매번 랜덤하게 10자리로 설정되는 것 같다.


 크롬 개발자 도구를 띄워둔 뒤 페이지를 새로 고침하고 2초안에 아래의 코드를입력하면 문제를 클리어할 수 있다.


단순히 'id', 'cmt'에 임의의 값을 채우고 `lv5frm .hack.value=lv5frm.attackme.value`로 'hack'에 'attackme'의 값을 입력하고 submit하는 코드이다.

```javascript
lv5frm.id.value = 'HiHi'
lv5frm.cmt.value = 'HiHi'
lv5frm .hack.value = lv5frm.attackme.value
lv5frm.submit()
```


![20번_3](/img/20번_3.JPG)


프로그래밍하지 말라고 했는데... 심심해서 해봤다. 아래의 파이썬 코드를 실행해도 문제가 풀린다.
같은 동작을 python requests 모듈로 구현한 것이다.

 한 가지 특이한건 쿠키에 'st'라는 값이 있는데 request 보낼때 이 값을 일치시켜야 문제를 클리어할 수 있다.


 ```python
 import requests
 import re


 URL = 'http://webhacking.kr/challenge/codeing/code4.html'
 cookies = {'PHPSESSID': '세션쿠키값', 'st':'1'}
 data = {'id':'aa', 'cmt':'aaa'}
 response = requests.get(URL, cookies=cookies)

 text = response.text
 idx = text.find('value=')
 value = text[idx+7:idx+17]

 cookies['st'] = response.headers['Set-Cookie'].split('=')[1]
 data['hack'] = value

 response = requests.post(URL, data=data, cookies=cookies)

 print(response.status_code)
 print(response.text)

 ```
