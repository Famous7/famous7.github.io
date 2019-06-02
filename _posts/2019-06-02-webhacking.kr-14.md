---
title: "webhacking.kr 14번 문제 풀이"
excerpt_separator: "<!--more-->"
categories:
  - webhacking
tags:
  - webhacking
  - wargame
classes: wide
---

[14번 바로가기](http://webhacking.kr/challenge/javascript/js1.html)

14번 문제에 들어가면 아래와 같은 화면이 나온다. 우선 페이지 소스를 보자.

![14번_1](/img/14번_1.JPG)


```html
<html>
<head>
<title>Challenge 14</title>
<style type="text/css">
body { background:black; color:white; font-size:10pt; }
</style>
</head>
<body>
<br><br>
<form name=pw><input type=text name=input_pwd><input type=button value="check" onclick=ck()></form>
<script>
function ck()
{
var ul=document.URL;
ul=ul.indexOf(".kr");
ul=ul*30;
if(ul==pw.input_pwd.value) { alert("Password is "+ul*pw.input_pwd.value); }
else { alert("Wrong"); }
}

</script>


</body>
</html>
```

별다른 내용은 없고, 입력폼에 값을 넣고 "check" 버튼을 누르면 그 값과 특정 값을 비교해서 일치하면 flag가 출력되는 형태이다.

`ck()`함수를 보면 페이지 URL에서 '.kr'의 인덱스를 구한뒤 30을 곱한 값과 입력폼에 입력한 값을 비교한다. 일치하는 경우 `ul * pw.input_pwd.value` 가 flag로 출력된다. 즉 `ul ** 2`이 flag가 된다.

'http://webhacking.kr/challenge/javascript/js1.html' 에서 '.kr'의 인덱스는 17이다. 따라서 ul=`17*30`으로 510이며 flag=`510 ** 2`임으로 260100이다.

크롬 개발자 도구의 console창에 코드를 복사해 넣으면 쉽게 답을 구할 수 있다.

구한답인 '260100'을 Auth 페이지에 들어가 입력하면 문제를 클리어할 수 있다.


![14번_2](/img/14번_2.JPG)
