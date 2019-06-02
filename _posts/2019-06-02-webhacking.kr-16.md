---
title: "webhacking.kr 16번 문제 풀이"
excerpt_separator: "<!--more-->"
categories:
  - webhacking
tags:
  - webhacking
  - wargame
classes: wide
---

[16번 바로가기](http://webhacking.kr/challenge/javascript/js3.html)

16번 문제에 들어가면 아래와 같은 화면을 볼 수 있다.

![16_1번](/img/16번_1.JPG)


화면만 보면 뭐하는 문제인지 모르겠지만, URL에 javascript라고 써있는걸보면 자바스크립트 문제 같다.
우선 소스를 확인해보자.

```html
<html>
<head>
<title>Challenge 16</title>
<body bgcolor=black onload=kk(1,1) onkeypress=mv(event.keyCode)>
<font color=silver id=c></font>
<font color=yellow size=100 style=position:relative id=star>*</font>
<script>
document.body.innerHTML+="<font color=yellow id=aa style=position:relative;left:0;top:0>*</font>";

function mv(cd)
{
kk(star.style.posLeft-50,star.style.posTop-50);
if(cd==100) star.style.posLeft=star.style.posLeft+50;
if(cd==97) star.style.posLeft=star.style.posLeft-50;
if(cd==119) star.style.posTop=star.style.posTop-50;
if(cd==115) star.style.posLeft=star.style.posLeft-50;;
if(cd==124) location.href=String.fromCharCode(cd);
}


function kk(x,y)
{
rndc=Math.floor(Math.random()*9000000);
document.body.innerHTML+="<font color=#"+rndc+" id=aa style=position:relative;left:"+x+";top:"+y+" onmouseover=this.innerHTML=''>*</font>";
}

</script>
</body>
</html>
```


키보드를 누르면 `mv()`함수가 실행되어 입력한 키코드에 맞는 동작을 수행하게 된다.

아래의 키를 제외한 키를 누르면, 화면에 별이 추가되고 표에 해당하는 키를 누르면 별의 위치가 변경된다.

![16_2번](/img/16번_2.JPG)

Key | Code
--------- | ---------
100 | 4 (Num Lock)
97 | 	1 (Num Lock)
119 | 	F8
115 | 	F4
124 | \|

여기서 124번 키('\|')를 누를경우 `location.href=String.fromCharCode(cd)`가 실행되어 특정 URL로 이동된다. 저기에 flag가 있는 것 같으니 이동해보자.


![16_3번](/img/16번_3.JPG)


바로 flag가 뜬다. 이걸 Auth 페이지에 입력하면 문제를 클리어할 수 있다.
